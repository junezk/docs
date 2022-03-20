# Go 设置各种选项的最佳实践

## 背景

在 Go 里面写一个 struct 时，经常会遇到要给 struct 里面的各个字段提供设置功能。这个问题看起来很简单很容易，实际上困扰了不少人，连 Go 的三巨头之一 Rob Pike 都曾经为之苦恼了一段时间，后来找到了最佳实践后还为此开心地写了一篇 Blog。

我最早是在 GRPC 的代码里发现这个套路的，后来在今年 7 月 Go 官方 Blog 里又看到了对这个套路的推荐，以及 Rob Pike 的 Blog 链接。我自己在代码里尝试之后感觉很好，又推荐给同事尝试，大家都很喜欢。

## 示范案例

我用这样一个需求案例来对比一下各种套路的优劣。

我们要写一个 struct，它的核心功能是创建一个网络连接 net.Conn 的实例，也就是实现下面这个方法：

```go
type MyDialer struct {
	dialer *net.Dialer
}

func (d *MyDialer) DialContext(ctx context.Context, addr net.Addr) (net.Conn, error) {
	return d.dialer.DialContext(ctx, addr.Network(), addr.String())
}
```

针对这个 Dialer ，我们增加两个选项，一个是连接超时，一个是重试次数。代码就变成了这样：

```go
type MyDialer struct {
	dialer *net.Dialer
	timeout time.Duration
	retry int
}

func (d *MyDialer) DialContext(ctx context.Context, addr net.Addr) (conn net.Conn, err error) {
	for i := 0; i < d.retry+1; i++ {
		d.dialer.Timeout = d.timeout
		conn, err = d.dialer.DialContext(ctx, addr.Network(), addr.String())
		if err == nil {
			return conn, err
		}
	}
	return nil, err
}
```

现在问题来了，我们需要完成一个构造 MyDialer 的方法，在构造时可以指定超时和重试的配置。

这个问题很简单，对不对？实际上并非如此，我们来看一下怎么设计。

## 常规套路

在说最佳套路之前，先梳理一下常见的常规套路。分析这些套路的优劣，有助于理解最佳套路为何是最佳的。

常规套路大致可以分三种：

- 字段导出为公共
- 在生成方法上增加配置字面量
- 提供 Set 系列方法

### 常规套路 1：导出字段

首先我们可以考虑一种最简单的方式，把 MyDialer 里面需要对外设置的字段都导出。

```go
type MyDialer struct {
	Dialer *net.Dialer
	Timeout time.Duration
	Retry int
}
```

Go 标准库中大部分结构体都是这样处理的，例如 http.Client 等。这种做法简单得令人发指，不过却有一些问题。

1. 因为没有初始化方法，部分字段在使用的时候是需要先判断一下调用者是否初始化的。例如这个例子里面，如果 *net.Dialer 没有初始化，那么运行时会直接 panic。
2. 为了解决 #1 的问题，我们还需要在使用这些字段的时候判断一下是否初始化过，如果没有初始化，就使用默认值。
3. 使用方法 #2 又引入一个更麻烦的问题，默认值如果不是一个类型的零值，那就无法判断字段的值是未被初始化，还是调用者有意设置的。

考虑一下这样的代码：

```go
func (d *Dialer) DialContext(ctx context.Context, addr net.Addr) (conn net.Conn, err error) {
	if d.Dialer == nil {
		d.Dialer = &net.Dialer{}
	}
	if int64(d.Timeout) == 0 {
		d.Timeout = time.Second // 使用默认的超时
	}
	if d.Retry == 0 {
		// 完了……到底是调用者不想重试，还是他忘了设置？
		// d.Retry = 2
	}
}
```

### 常规套路 2：使用 Config 结构体

第二种常规套路是设置一个 New 方法，使用一个 Config 结构体。

我们先说不使用 Config 结构体的方法：

```go
func NewMyDialer(dialer *net.Dialer, timeout time.Duration, retry int) *MyDialer {
	return &MyDialer{
		dialer: dialer,
		timeout: timeout,
		retry: retry,
	}
}
```

在很多语言里面，这是最典型的写法。但是这种写法对于 Go 来说很不合适，原因在于 Go 不支持多态函数，如果以后增加了新的字段，在很多语言里面（例如 Java 或 C++），只要再声明一个参数不同的新的 New 方法就可以了，编译器会自动根据调用处的参数格式选择对应的方法，但是 Go 就不行了。

为了避免这种问题，很多库会使用 Config 结构体：

```go
type Config struct {
	Dialer *net.Dialer
	Timeout time.Duration
	Retry int
}

// 这样调用：
// dialer := MyDialer(&Config{Timeout: 3*time.Second})

func NewMyDialer(config *Config) *MyDialer {
	d := MyDialer{
		dialer: config.Dialer,
		timeout: config.Timeout,
		retry: config.Retry,
	}
	// 再检查一下设置是否正确
	if d.dialer == nil {
		d.dialer = &net.Dialer{}
	}
	if int64(d.timeout) == 0 {
		d.timeout = time.Second
	}
	if d.retry == 0 {
		// 问题又来了，调用者是不是故意设置retry为0的呢？
	}
}
```

使用 Config 模式最麻烦的问题就在于对配置零值的处理。以至于有段时间看到很多人走这样的邪路：

```go
type Config struct {
	// ... other fields
	Retry *int
}
```

通过配置项指针是否为`nil`来判断是否为调用者故意设置。不过使用上很麻烦：

```go
// 直接用字面量会无法编译：
config := Config{
	Retry: &3,
}
// 必须创造一个临时变量：
r := 3
config := Config{
	Retry: &r,
}
```

### 常用套路 3：提供 Set 方法

提供 Set 方法是另一种常见套路，配合上 New 方法使用，几乎能满足绝大多数情况。

```go
type MyDialer struct{...}

func NewMyDialer() *MyDialer {
	return &MyDialer{
		dialer: &net.Dialer{},
		timeout: time.Second,
		retry: 2,
	}
}

func (d *MyDialer) SetRetry(r int) {
	d.retry = r
}
```

在许多场景下，Set 模式已经非常不错了，但是在下面两种情况下仍然有些麻烦：

1. 有一些对象的字段希望只在生成的时候配置一次，之后就不能再修改了。这个时候用 Set 就不能很好地保证这一点。
2. 有时候我们希望我们提供出去的库的功能是以 interface 来表示的，这样可以更容易地将实现替换掉。在这种情况下使用 Set 模式会大大增加 interface 的方法数量，从而增加替换实现的成本。

举例来说：

```go
// 接下来 MyDialer 以接口方式提供
type MyDialer interface {
	DialContext(ctx context.Context, addr net.Addr) (net.Conn, error)
}

// 而 myDialer 作为 MyDialer 接口的实现，是不导出的
type myDialer struct {...}

func NewMyDialer() MyDialer {
	return &myDialer{}
}
```

在这种设计下，如果使用 Set 模式，就需要为 MyDialer 这个接口增加 `SetRetry`, `SetTimeout`, `SetDialer` 这一系列方法，使用方如果在写单测等时候需要替换掉 MyDialer 的话，也需要在自己的测试替身（Test Double）实现上增加这三个方法。

## Option Types 套路

Rob Pike 把这个套路称为 Option Types ，我就沿用这个方法。这种看上去似乎是 23 种经典设计模式中的命令模式的一种形态。

Options Types 套路的核心思路是创建一个新的 Option 类型，这个类型负责修改配置，被调用方接收这个类型来修改自己的选型，调用方创建这个类型传给被调用方。

我们继续刚才的例子，现在假设我们分别设计了 MyDialer 的接口和实现，让调用者使用 MyDialer 接口，但是我们提供 New 方法创建 MyDialer 的实现 myDialer

```go
// MyDialer 是导出的接口类型
type MyDialer interface {
	DialContext(context.Context, net.Addr) (net.Conn, error)
}

// myDialer 是未导出的接口实现
type myDialer struct {...}
```

### 实现步骤

1. 首先，我们需要创建一个 Option 类型。

```go
type Option interface {
	apply(*myDialer)
}
```

复制代码

1. 接下来我们让 myDialer 可以处理这个类型。

```go
// 我们可以在构造方法中使用
func NewMyDialer(opts ...Option) MyDialer {
	// 首先我们将默认值填上
	d := &myDialer{
		timeout: time.Second,
		retry: 2,
	}
	// 接下来用传入的 Option 修改默认值，如果不需要修改默认值，
	// 就不需要传入对应的 Option
	for _, opt := range opts {
		opt.apply(d)
	}
	// 最后再检查一下，如果 Option 没有传入自定义的必要字段，我
	// 们在这里补一下。
	if d.dialer == nil {
		d.dialer = &net.Dialer{}
	}
	return d
}

// 我们也可以提供单独的方法，并随接口导出，提供类似 Set 模式的功能。
func (d *myDialer) ApplyOptions(opts ...Option) {
	for _, opt := range opts {
		opt.apply(d)
	}
}
```

1. 现在我们来实现 Option 类型。

先用常规方式写一种啰嗦的写法：

```
type retryOpt struct {
	retry int
}

func RetryOption(r int) Option {
	return &retryOpt{retry:r}
}

func (o *retryOpt) apply(d *myDialer) {
	d.retry = o.retry
}

type timeoutOpt struct {
	timeout time.Duration
}

func TimeoutOption(d time.Duration) Option {
	return &timeoutOpt{timeout: d}
}

func (o *retryOpt) apply(d *myDialer) {
	d.timeout = o.timeout
}
// ... dialer 的 Opt 类似

```

常规方式里面需要一个实现 Option 接口的类型，和一个该类型的构造方法。所以我们设置 3 个字段，就需要写 9 段代码。

下面我们用**函数转单方法接口**的套路，来简化实现 Option 的代码。

```go
type optFunc func(*myDialer)

func (f optFunc) apply(d *myDialer) {
	f(d)
}

func RetryOption(r int) Option {
	return optFunc(func(d *myDialer) {
		d.retry = r
	})
}

func TimeoutOption(timeout time.Duration) Option {
	return optFunc(func(d *myDialer) {
		d.timeout = timeout
	})
}

func DialerOption(dialer *net.Dialer) Option {
	return optFunc(func(d *myDialer) {
		d.dialer = dialer
	})
}

```

### 使用示例

接下来我们使用这个 MyDialer，看看有多方便：

```go
// 无自定义 Option，全部使用默认的
d := NewMyDialer()
// 只修改 Retry，并且 Retry 是0次
d := NewMyDialer(RetryOption(0))
// 修改多个 Option
d := NewMyDialer(RetryOption(5), TimeoutOption(time.Minute), DialerOption(&net.Dialer{
	KeepAlive: 3*time.Second,
}))

```

### 补充

Rob Pike 是在 2014 年写 Blog 总结这个套路的，当时他的 Option 不是一个 interface，而是一个 function。使用上略有差异。目前普遍认为函数转单方法接口这种做法更灵活，建议大家使用这个方式。

## 总结

最后我说一个我总结这个套路的心得。

首先，最初我在寻找一个创建对象的最佳套路时，主要的方向还是看那五个创建型模式（工厂、抽象工厂、生成器、单例、原型），看来看去也没有找到合适的，没想到截止目前找到的最佳套路是命令模式。再次说明套路重要，对套路的创新更加重要。

其次，我想感叹一下，作为 [r@google.com](mailto:r@google.com) 这个顶级邮箱的拥有者，Rob Pike 老爷子仍然坚持亲自写代码，并在代码细节上如此尽善尽美，令人敬仰。而我们国内技术圈却经常花大量时间讨论架构师应不应该写代码，甚至架构师是否需要会写代码，这可能也是许多技术文章字里行间散发着一股伤痕文学气息的原因之一吧。

## 评论

> 思想的光芒
>
> ​	我的妈呀 这要是有十来个参数的配置 那得写多少东西啊
>
> eddix
>
> ​	https://pkg.go.dev/google.golang.org/grpc#DialOption grpc 这里有几十个参数要配置，写了几百行代码。
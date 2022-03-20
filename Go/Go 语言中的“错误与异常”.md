# Go 语言中的“错误与异常”

## 学到什么 

1. 什么是错误？
2. 如何创建错误？
3. 如何处理错误？
4. `errors` 包的使用？
5. 什么是异常？
6. 如何处理异常？
7. `defer` 关键字的作用？
8. `recover` 函数的使用？

## 什么是错误

在写代码时，不是所有情况都能处理，对于不能处理的逻辑，就需要使用错误机制告诉上层调用者。

在 Go 语言中，错误是被作为一个接口类型对待，它不像其它语言一样使用 `try/catch` 去捕捉，只需在函数或方法之间使用一个错误类型变量去传递。

## 创建错误

这里所说的创建错误，实际上就是去实现错误接口，接口如下：

```
type error interface {
    Error() string
}
```

该接口是 Go 标准包内置的，所有创建的错误类型都需要实现此接口，怎么去实现接口，不懂的看看上篇文章 [《接口》](https://mp.weixin.qq.com/s?__biz=MzIzNzQwNTQwNg==&mid=2247484511&idx=1&sn=2778802b7c57181fa6426fbe4b076e10&scene=21#wechat_redirect)。

### 1. errors.New

Go 语言中内置了一个处理错误的标准包，你不需要自己去实现 `error` 接口，它有函数帮你处理，如下：

```
import "errors"

var ErrNotFound = errors.New("not found")
```

导入 `errors` 包，调用 `New` 函数创建了一个错误并保存到 `ErrNotFound` 变量，该错误信息为 `not found` 。

### 2. fmt.Errorf

`fmt` 标准包内也有一个创建错误的函数 `Errorf` ，该函数可以使用占位符设置错误信息，比 `errors.New` 函数更灵活。

```
import "fmt"

var ErrHuman = fmt.Errorf("%s不符合我们人类要求", "老苗")
```

### 3. 自定义错误类型

如果上述两种方式你觉得还不够灵活，那可以自定义错误类型。

```
type ErrorPathNotExist struct {
   Filename string
}

func (*ErrorPathNotExist) Error() string {
   return "文件路径不存在"
}

var ErrNotExist error = &ErrorPathNotExist{
   Filename: "./main.go",
}
```

- 自定义了一个`ErrorPathNotExist` 结构体，该结构体实现了 `error` 接口。
- 创建了一个 `ErrNotExist` 错误类型变量。

这种如果不明白具体怎么应用，不着急，往下看。

> 补充知识点：如果方法的接收者没有被使用可以直接省略掉，例：`func (*ErrorPathNotExist)` ，不省略的话就是这样：`func (e *ErrorPathNotExist)` ，当然也可以使用下划线"_"代替 "e" 只是没有必要性。

## 打印错误

在项目开发中，错误常常通过函数或方法的返回值携带，返回的位置也通常被放置在最后一位。

```
// error/file.go
package main

import "io/ioutil"

// 读取文件内容
func LoadConfig() (string, error) {
    filename := "./config.json"
    b, err := ioutil.ReadFile(filename)
    if err != nil {
        return "", err
    }

    content := string(b)
    if len(content) == 0 {
        return "", errors.New("内容为空")
    }
    return content, nil
}
```

- `ReadFile` 函数读取 "config.json" 文件内容。
- `(string, error)` 返回两个值，第一个为文件内容，第二个为错误。
- `err != nil` 用于判断是否有错误，如果有 `return` 直接返回。
- `string(b)` 变量 b 的类型为 `[]byte` ，该操作是将 `[]byte` 类型转为 `string` 。
- 增加了一个“内容为空”的错误判断，该错误也可以直接保存到变量中返回。

```
var ErrEmpty = errors.New("内容为空")

func LoadConfig() (string, error) {
// ...
    return "", ErrEmpty
// ...
}
```

现在假设 "config.json" 文件不存在，调用 `LoadConfig` 函数看看结果。

```
package main

import (
    "fmt"
    "log"
)

func main() {
    content, err := LoadConfig()
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("内容：", content)
}

// 输出
2021/09/23 16:57:25 open ./config.json: The system cannot find the file specified.
```

- 当 `err` 不等于 `nil` 时，打印错误，并退出程序。
- `log` 标准包包含打印日志的函数集。
- `log.Fatal` 函数打印日志，并终止程序向下执行。
- 输出的错误消息显示没有找到指定的文件。
- 打印错误时，也可以使用 `fmt` 包，例如：`fmt.Println(err)` ，只是输出信息没 `log` 包多。

### os.Exit

该函数通知程序退出，并且该函数之后的逻辑将不会被执行。在调用时需要指定退出码，为 0 时，表示正常退出程序。

```
os.Exit(0)
```

不主动调用该函数，即程序从 main 函数自然结束时，默认的退出码为 0。在使用编写工具时或许能看到成功的退出码信息，例如我使用的是 Goland，执行代码后输出的结果末尾会显示如下信息。

```
Process finished with exit code 0
```

如果不正常退出，退出码则为非 0，通常使用 1 表示未知错误。

```
os.Exit(1) 
```

在使用 `log.Fatal` 函数时，内部就调用了 `os.Exit(1)` 。

## 错误加工

### 1. 错误拼接

在返回错误时，如果想携带附加的错误消息时，可以使用 `fmt.Errorf` ，现在修改 `LoadConfig` 函数。

```
func LoadConfig() (string, error) {
    filename := "./config.json"
    b, err := ioutil.ReadFile(filename)
    if err != nil {
        return "", fmt.Errorf("读取文件出错:%v", err)
    }

    // ...
}
```

`%v` 占位符表示获取数据的值，在这块表示错误消息，后续会详细讲解占位符的使用。

现在重新执行上面的 `main` 函数，还是假设 "config.json" 文件不存在。

```
// ...
content, err := LoadConfig()
if err != nil {
    log.Fatal(err)
}
//...

// 输出
2021/09/24 11:37:33 读取文件出错:open ./config.json: The system cannot find the file specified.
```

`LoadConfig` 函数返回的 `err` 变量中携带了附加的错误消息，但这样有个问题，附加的错误和原始错误消息杂糅在一块导致不能分离。

### 2. 错误嵌套和 errors.Unwrap

上面所说的”错误消息杂糅在一块导致不能分离“问题，如果还没有明白的话，可以再看看这块，你应该就豁然开朗了。

![图片](Go 语言中的“错误与异常”.assets/640.webp)错误嵌套

错误嵌套就类似上图，`err1` 嵌套类 `err2` ，`err2` 也可以继续嵌套。如果想从 `err1` 中获取 `err2` 就剥一层，类似洋葱一样，一层一层往里找。

那怎么实现这种嵌套关系呢，还是使用 `fmt.Errorf` 函数，只是使用另外一个占位符`%w` ，`w` 的英文全名就是 `wrap` 。

继续修改 `LoadConfig` 函数，如下：

```
func LoadConfig() (string, error) {
    filename := "./config.json"
    b, err := ioutil.ReadFile(filename)
    if err != nil {
        return "", fmt.Errorf("读取文件出错:%w", err)
    }

    // ...
}
```

现在再执行 `main` 函数，即调用 `LoadConfig` 该函数，并打印错误。

```
2021/09/24 18:07:14 读取文件出错:open ./config.json: The system cannot find the file specified.
```

是不是发现错误结果没有变化，那修改下 `main` 函数。

```
package main

import (
    "errors"
    "fmt"
    "log"
)

func main() {
   content, err := LoadConfig()
   if err != nil {
      log.Fatal(errors.Unwrap(err))
   }

   fmt.Println("内容：", content)
}

// 输出
2021/09/24 18:11:09 open ./config.json: The system cannot find the file specified.
```

在打印错误时，增加了一个 `errors.Unwrap` 函数，该函数就是用来取出嵌套的错误，再看看输出的结果，附加的错误信息”读取文件出错:“已经没有了。

### 3. 自定义错误类型

在上面讲过了如何自定义错误类型，现在讲讲如何应用你自定义的错误，接下来将 `LoadConfig` 函数中的内容为空的错误改为自定义错误类型。

```
func LoadConfig() (string, error) {
    filename := "./config.json"
  // ...
    if len(content) == 0 {
        return "", &FileEmptyError{
            Filename: filename,
            Err:      errors.New("内容为空"),
        }
    }
    return content, nil
}
```

`FileEmptyError` 是自定义的错误类型，同样的实现 `error` 接口。

```
type FileEmptyError struct {
    Filename string
    Err      error
}

func (e *FileEmptyError) Error() string {
    return fmt.Sprintf("%s %v", e.Filename, e.Err)
}
```

现在调用 `LoadConfig` 函数，现在假设 "config.json" 文件存在，但内容为空，结果如下：

```
content, err := LoadConfig()
if err != nil {
    if v, ok := err.(*FileEmptyError); ok {
        fmt.Println("Filename:", v.Filename)
    }
    log.Fatal(err)
}

// 输出
Filename: ./config.json
2021/09/27 14:36:40 ./config.json 内容为空
```

- 将 `err` 变量的接口类型推断为`*FileEmptyError` 类型，并输出 `Filename` 字段。
- 打印自定义错误内容。

如果想使用 `errors.Unwrap` 函数 , 就需要实现 `Wrapper` 接口，`fmt.Errorf` 函数中的 `%w` 占位符底层实现好了此接口。

```
type Wrapper interface {
   // Unwrap returns the next error in the error chain.
   // If there is no next error, Unwrap returns nil.
   Unwrap() error
}
```

只需要我们实现了 `Unwrap` 方法就可以。

```
func (e *FileEmptyError) Unwrap() error {
    return e.Err
}
```

下来自行去实验，我在这就不罗嗦了。

## 错误判断

对于一个函数或方法，返回的错误常常不止一个错误结果，如果对于不同的错误结果你想有不同的处理逻辑，那这个时候就要对错误结果进行判断。

```
// ...
var (
    ErrNotFoundRequest = errors.New("404")
    ErrBadRequest      = errors.New("请求异常")
)

func GetError() error {
    // ...
    // 错误 1
    return ErrNotFoundRequest

    // ...
    // 错误 2
    return ErrBadRequest

    // ...
    // 错误 3
    path := "https://printlove.com"
    return fmt.Errorf("%s:%w", path, ErrNotFoundRequest)

    // ...
}
```

`GetError` 函数没有写具体逻辑，只展示了 3 个错误的返回，下来看看如何对这几种情况进行判断。

### 1. 最简单"=="

最简单的就是使用”==“判断错误结果。

```
err := GetError()
if err == ErrNotFoundRequest {
    // 错误 1
} else if err == ErrBadRequest {
    // 错误 2
}
```

对于这种判断方式有个问题，“错误 3"是不符合这两个 `if` 判断的，但从错误分类上说，它属于 `ErrNotFoundRequest` 错误，只是拼接了请求地址数据 `path` ，下来往下看另外一种判断方式。

### 2. errors.Is

使用 `errors.Is` 函数解决“错误 3”的判断问题，下来分析为什么？

“错误 3”中使用了占位符`%w`，它就是将  `ErrNotFoundRequest` 错误嵌入其中，`**Is` 函数的作用就是一层层的对错误进行剥离判断**，直到成功或没有嵌套的错误为止，没明白的话可以结合上面的图。

函数定义如下：

```
func Is(err, target error) bool
```

- `err` 参数为要判断的错误。
- `target` 参数为要比对的错误对象。

使用如下：

```
err := GetError()
if errors.Is(err, ErrNotFoundRequest) {
    // 错误 1,错误3
} else if errors.Is(err, ErrBadRequest) {
    // 错误 2
}
```

- 在具体项目中，调用函数或方法时，我们不知道底层是否进行了错误嵌套，如果不明确的话就统一使用 `Is` 函数。
- `错误 1` 和 `错误 2` 没有嵌套也可以判断。

### 3. errors.As

这个和 `errors.Is` 函数类似，不同点就是该函数只判断错误类型，而 `errors.Is` 函数不仅判断类型，也要判断值（错误消息）。

```
// error/as.go
package main

import (
    "errors"
    "fmt"
)

type ErrorString struct {
    s string
}

func (e *ErrorString) Error() string {
    return e.s
}

func main() {
    var targetErr *ErrorString
    err := fmt.Errorf("new error:[%w]", &ErrorString{s: "target err"})

    fmt.Println(errors.As(err, &targetErr))
}

// 输出
true
```

`targetErr` 变量有几点要求：

- 无需初始化。
- 必须是指针类型，并且实现了 `error` 接口。
- `As`函数不接受`nil`，因此不能直接使用`targetErr`变量，要使用其引用`&targetErr`。

## 什么是异常

错误就是上述所讲的，它的出现不会导致程序异常退出。

那什么情况会异常退出呢？比如：

- 下标越界
- 除数为 0
- 等等

通常该异常是你没留意到代码问题而抛出，当然你可以可以主动抛出。

## panic

使用 `panic` 函数可以主动抛出异常，该函数格式如下：

```
func panic(v interface{})
```

- `v` 参数为空接口类型，那就说明可以接受任意类型数据。

例：

```
panic("我是异常")

// 输出
panic: 我是异常

goroutine 1 [running]:
main.main()
        C:/workspace/go/src/gobasic/panic/panic.go:4 +0x45
exit status 2
```

从输出的结果上可以看出几点：

- 打印出具体的异常位置，这些信息称作堆栈信息。
- 程序终止，退出码为 2。

## 处理异常

不管是主动抛出异常，还是你程序哪块有 bug 被动抛出异常，这些在我们写项目时都是很严重的问题，因为它会导致我们的程序异常退出。

在其它语言中，通过`try/catch` 机制可以捕捉异常来保证程序的正常运行，那在 Go 语言中使用 `recover` 函数捕捉异常。

在学习这个函数前，先要了解另外一个关键字 `defer` 。

### 1. defer

它不是函数，只是一个关键字。该关键字后面所跟的语句将延迟执行，在所在函数或方法正常结束时或出现异常中断时，再提前执行。

```
package main

import "fmt"

func main() {
    defer func() {
        fmt.Println("defer")
    }()
    fmt.Println("main")
    panic("panic")
}

// 输出
main
defer
panic: panic
...
```

- `defer` 关键字后面跟了一个匿名函数调用，有名字的函数当然也可以。
- 遇到 `defer` 关键字，后面的语句会延迟执行，因此先输出"main"字符串。
- `panic` 抛出异常，因此在退出前先执行 `defer` 语句。
- 如果调用了 `os.Exit` 函数，`defer` 后的语句将不会执行。

如果在一个函数或方法中出现了**多个 `defer` 语句，那会采用先进后出原则**，即先出现的 `defer` 语句后执行，后出现的先执行。

例：

```
package main

import "fmt"

func Fun1() {
    fmt.Println("猜我啥时候输出？")
}

func main() {
    defer func() {
        fmt.Println("defer")
    }()
    defer Fun1()
    fmt.Println("main")
    panic("panic")
}

// 输出
main
猜我啥时候输出？
defer
panic: panic
...
```

现在讲了 `defer` 关键字的概念，可能还不知道实际中什么地方用到，我现在举个例子。

```
// defer/main.go

// 拷贝文件
// srcName 路径文件拷贝到 dstName 路径文件 
func CopyFile(srcName, dstName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

代码中有两个 `defer` 关键字，后面的语句用来关闭文件释放资源。

如果不用此关键字，关闭文件的`Close`函数就必须写在 `io.Copy` 函数后，因为该函数还要使用文件资源，提前关闭了，就完蛋了。所以，使用 `defer` 关键字后就会延迟执行，因此就不需要考虑文件什么时候被使用。

当然也不是只有这一种情况应用，只要记住，在函数或方法结束前你才想处理的语句都可以使用 `defer` 关键字。

### 2. recover

了解了 `defer` 关键字后就明白了，在程序出现异常之前 `defer` 语句先被执行，因此在 `defer` 后的语句就可以提前拦截异常。

```
// panic/recover.go

package main

import "fmt"

func main() {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("我捕捉的：", err)
            fmt.Println("我好好的")
        }
    }()
    panic("我是异常")
}

// 输出
我捕捉的： 我是异常
我好好的
```

- `recover` 返回异常信息。

不管是主动抛出异常，还是被动的，`recover` 函数都能捕捉，这样以保证程序的正常进行。

假如函数 `A` 调用了很多函数，这些函数又调用了很多，只要下面被调用的函数出现异常，函数 `A` 中的 `recover` 函数将都可以捕捉到。但其中还是有个特例， `Goroutine` 中出现的异常是无法被捕捉到的，必须在新的 `Goroutine` 中重新使用 `recover` 函数，这个让人期待的知识点后续会讲。

## 总结

本篇讲解了两个概念，”错误“和”异常“ ，这两个概念很重要，一定要掌握明白。还有一点，在处理错误时，有时候发觉错误结果对于我们没用，那这个时候就可以忽略掉。
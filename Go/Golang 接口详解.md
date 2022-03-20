# gGolang 接口详解

## 前言

在之前的文章中我们说过，Golang 是通过 `结构体(struct)-方法(method)-接口(interface)` 的组合使用来实现面向对象的思想。在前文 [Golang 复合类型](http://mp.weixin.qq.com/s?__biz=MzU2OTMwNzY2NQ==&mid=2247483697&idx=1&sn=5d925907850dc338cffb08d1922aaf1f&chksm=fc81e64fcbf66f59027b5debf39acc20e4fa7f267e2ea7d963cbba1e564bfa9317043197f264&scene=21#wechat_redirect) 和 [Golang 方法详解](http://mp.weixin.qq.com/s?__biz=MzU2OTMwNzY2NQ==&mid=2247483718&idx=1&sn=2ebcd577f912e9b909506e3d47a417ab&chksm=fc81e638cbf66f2e13ad037fbeeac0d2179cb2028ee1cf26ec716ddfa19948dd38e65fd99897&scene=21#wechat_redirect) 已经详细介绍过 `struct` 和 `method`，本文将介绍 Golang 面向对象的另一个重要组成部分：`接口(interface)`。

## 接口

### 接口概念

接口是一种抽象的类型，描述了一系列方法的集合，作用是对一系列具有联系的方法做出抽象和概括。接口只定义方法名和参数，而不包含具体的实现，这种抽象的方式可以让程序变得更加灵活更加通用。

在很多语言中，接口都是`侵入式`的，侵入式接口的意思是实现类需要明确声明自己实现了某个接口，这就带来了一个很矛盾的问题，比如 A 调用了 B 的接口，那么 A 一定会希望接口被设计成自己想要使用的样子，但是 B 才是接口的实现方，基于模块设计的单向依赖原则，B 在实现自身的业务时，不应该关心某个具体使用方的要求，一个接口被定义的时候，并不知道自己的方法会被谁实现，也不知道会被怎么样实现。因此，侵入式接口一直是面向对象编程中一个经常遭受质疑的特性。

不同的是，Golang 的接口是一种 `非侵入式` 的接口，一个类型不需要明确声明，只要实现了接口的所有方法，这个类型就实现了该接口，这个类型的对象就是这个接口类型的实例。因此，在 Golang 中，不再需要定义类的继承关系，在定义接口时候，只需要关心自己需要提供哪些方法，其他的方法有使用方按需定义即可。

### 接口定义

```go
/* 定义接口 */
type interface_name interface {
   method_name1(input_paras...) [return_type]
   method_name2(input_paras...) [return_type]
   method_name3(input_paras...) [return_type]
}

/* 定义结构体 */
type struct_name struct {
   /* variables */
}

/* 实现接口方法 */
func (struct_name_variable struct_name) method_name1(input_paras...) [return_type] {
   /* 方法实现 */
}

func (struct_name_variable struct_name) method_name2(input_paras...) [return_type] {
   /* 方法实现*/
}

func (struct_name_variable struct_name) method_name3(input_paras...) [return_type] {
   /* 方法实现*/
}
```

go语言的源码中大量使用到了接口，比如说在前面的文章中多次使用到的 error 类型

```go
// The error built-in interface type is the conventional interface for
// representing an error condition, with the nil value representing no error.
type error interface {
  Error() string
}
```

### 封装性

接口是 Golang 封装性的重要一环，接口可以封装具体类型和类型的值，即使一个类型还有别的方法，接口的实例也只能调用接口暴露出来的方法。如下：

```go
type HelloInterface interface {
  Hello()
}

type User struct {
}

func (f *User) Hello() {
  fmt.Println("hello")
}

func (f *User) Bye() {
  fmt.Println("bye")
}

func InterfaceTest() {
  u := &User{}
  u.Hello()   // ok
  u.Bye()    // ok

  var user HelloInterface = new(User) // 接口实例化
  user.Hello() // ok
  user.Bye() // Compile error: user.Bye undefined (type HelloInterface has no field or method Bye)
}
```

注意，使用一个接口对象必须要先实例化，否则接口对象的值为 nil，调用 nil 对象的任何方法都会产生空指针 panic。

### 接口查询

和查询某个元素是否在 map 中类似，Golang 也内置了接口查询，可以使用和 map 类似的语法来检查对象实例是否实现了接口，如下：

```go
var user HelloInterface = new(User) // 接口实例化
if u1, ok := user.(HelloInterface); ok {
  fmt.Println(u1)   // yes
}

if u2, ok := user.(Reader); ok {
  fmt.Println(u2)   // no
}
```

也可以查询对象是否是某个类型

```go
if u3, ok := user.(*User); ok {
  fmt.Println(u3)
}
```

Golang 还可以使用断言和反射来进行类型查询，这两个内容会在后续的文章中介绍。

### 接口赋值

#### 将对象实例赋值给接口

要将对象实例赋值给接口，要求该对象实例实现了接口要求的所有方法。如：

```go
type Integer int

func (a Integer) Less(b Integer) bool {
  return a < b
} 

func (a *Integer) Add(b Integer) {
  *a += b
}

type LessAdder interface {
  Less(b Integer) bool
  Add(b Integer)
}

var a Integer = 1
var b LessAdder = &a
```

注意，此处赋值时用 `&a` 而不是 `a`, 因为 Go 会自动为 *Integer 生成一个新的 Less 方法

```go
func (a *Integer) Less(b Integer) bool {     
  return (*a).Less(b)
}
```

从而让 *Integer 既存在 Less()，又存在 Add(), 满足接口 LessAdder

#### 将一个接口赋值给另一个接口

在Go语言中，只要两个接口拥有相同的方法列表(不用考虑顺序)，那么它们就是等同的，可以相互赋值。

```go
package one

type ReadWriter1 interface {
  Read(buf []byte) (n int, err error) 
  Write(buf []byte) (n int, err error)
}
 
// 第二个接口位于另一个包中:
package two

type ReadWriter2 interface {
  Write(buf []byte) (n int, err error) 
  Read(buf []byte) (n int, err error)
}

// 可以相互赋值
var file1 two.ReadWriter2 = new(File) 
var file2 one.ReadWriter1 = file1 
var file3 two.ReadWriter2 = file2
```

接口赋值并不要求两个接口必须等价。如果接口 A 的方法列表是接口 B 的方法列表的子集， 那么接口 B可以赋值给接口 A，但是 A 不可以赋值给 B。（大接口可以赋值给小接口）

### 接口组合

类似于结构内嵌，接口的组合也是使用匿名机制实现的，如下：

```go
type Reader interface {
  Read(p []byte) (n int, err error)
}

type Writer interface {
  Write(p []byte) (n int, err error)
}

// 将 Read 和 Write 方法组合
// ReadWriter 接口既能做 Reader 接口的所有事情，又能做 Writer 接口的所有事情。

type ReadWriter interface {
  Reader
  Writer 
}

// 与下面的写法完全等价
type ReadWriter interface {
  Read(p []byte) (n int, err error) 
  Write(p []byte) (n int, err error)
}
```

## Any 类型

Go语言中任何对象实例都满足空接口 `interface{}`，所以可以把 `interface{}` 看作可以指向任何对象的 `Any` 类型，当函数可以接受任意的对象实例时，我们会将其声明为 `interface{}`，从而可以接受任意类型的对象，然后再使用类型断言来对该参数进行转换，再做后续的处理（具体内容参看后续类型断言的博客）。

最典型的例子是标准库 `fmt` 中 PrintXXX 系列的函数，例如:

```
func Printf(fmt string, args ...interface{})
func Println(args ...interface{})
```

## sort interface 示例

接下来，让我们通过介绍内置的 sort 包来加深一下对接口的理解，顺便了解一下这个常用包的使用。

Golang 的 sort 包中通过接口的方式内置了可以对任何类型的列表进行快排的功能，下面我们一起来看看它是如何使用的。

首先我们要先了解 `sort.Interface` 源码中定义了哪些方法：

```
// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package. The methods require that the
// elements of the collection be enumerated by an integer index.
type Interface interface {
  // Len is the number of elements in the collection.
  Len() int
  // Less reports whether the element with
  // index i should sort before the element with index j.
  Less(i, j int) bool
  // Swap swaps the elements with indexes i and j.
  Swap(i, j int)
}
```

可以看到，我们需要先定义三个方法：

1. 计算列表长度的方法
2. 比较两个元素的方法
3. 交换两个元素的方法

因此，我们需要定义一种类型，这种类型要同时具有以上三种方法，比如一个简单的 Student 类

```
type Student struct {
  ID int64
  Name string
}

type StudentSlice []*Student

func (s StudentSlice) Len() int {
  return len(s)
}

func (s StudentSlice) Less(i, j int) bool {
  return s[i].ID < s[j].ID
}

func (s StudentSlice) Swap(i, j int) {
  s[i], s[j] = s[j], s[i]
}
```

接下来，对一个 Student 进行初始化

```
s1 := &Student{ID:1, Name:"A"}
s2 := &Student{ID:2, Name: "B"}
s3 := &Student{ID:3, Name: "C"}
students := []*Student{s3, s1, s2}
```

准备工作已经做好，接下来我们先来看一下 `sort.Sort` 函数的源码

```
// Sort sorts data.
// It makes one call to data.Len to determine n, and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
func Sort(data Interface) {
  n := data.Len()
  quickSort(data, 0, n, maxDepth(n))
}
```

可以看到，函数的入参是一个 `sort.Interface` 类型的对象，然后对这个对象进行快排操作，所以，要使用这个函数，我们还需要把 `[]*Student` 类型转换成 `StudentSlice`，由于 `StudentSlice` 实现了 `sort.Interface` 的所有方法，所以 `StudentSlice` 的对象就是 `sort.Interface` 类型的对象。

```
sort.Sort(StudentSlice(students))
```

完成后，打印 students，我们就可以看到排好序的列表了。

```
{ID:1 Name:A}
{ID:2 Name:B}
{ID:3 Name:C}
```

对于自定义的类型，要进行排序就要完成上述的所有操作，为了方便使用，对于常用基本类型，go 源码已经为我们准备好了一系列可以直接调用的方法。

```go
// Ints sorts a slice of ints in increasing order.
func Ints(a []int) { Sort(IntSlice(a)) }

// Float64s sorts a slice of float64s in increasing order
// (not-a-number values are treated as less than other values).
func Float64s(a []float64) { Sort(Float64Slice(a)) }

// Strings sorts a slice of strings in increasing order.
func Strings(a []string) { Sort(StringSlice(a)) }

// IntsAreSorted tests whether a slice of ints is sorted in increasing order.
func IntsAreSorted(a []int) bool { return IsSorted(IntSlice(a)) }

// Float64sAreSorted tests whether a slice of float64s is sorted in increasing order
// (not-a-number values are treated as less than other values).
func Float64sAreSorted(a []float64) bool { return IsSorted(Float64Slice(a)) }

// StringsAreSorted tests whether a slice of strings is sorted in increasing order.
func StringsAreSorted(a []string) bool { return IsSorted(StringSlice(a)) }
```

我们可以直接使用这些方法对基本类型 slice 进行排序，如：

```go
ids := []int{5,1,7,1,3,8,7,4}
names := []string{"qqq", "www", "ee", "aa", "rr", "ba"}

sort.Ints(ids)
sort.Strings(names)

fmt.Println(ids)  // [1 1 3 4 5 7 7 8]
fmt.Println(names)  // [aa ba ee qqq rr www]
```
# Go 语言异常处理

## Error 与 Exception

Go [error](http://golang.org/pkg/builtin/#error) 是一个普通的接口，这在基础库（比如 [bufio](http://golang.org/src/pkg/bufio/bufio.go)）中被大量使用。

实践中通常使用 `errors.New()` 返回 [error](https://golang.org/src/pkg/errors/errors.go) 对象。

```go
type error interface {
    Error() string
}

type errorString struct {
    s string
}

func (e * errorString) Error() string {
    return e.s
}
```

内置的 `errors.New()` 会返回内部 `errorString` 对象的指针。

```go
func New(text string) error {
    return &errorString{text}
}
```

以 string 或 struct 指针的形式返回异常：

```go
// type errorString struct {
//     s string
// }

// type (e errorString) Error() string {
//     return e.s
// }

// func NewError(text string) error {
//     return errorString{text}
// }

// var ErrType = NewError("EOF")

type errorString string

// Error 实现 error 接口，返回 errorString。
func (e errorString) Error() string {
    return string(e)
}

// New 返回 error 对象。
func New(text string) error {
    return errorString(text)
}

var ErrNamedType = New("EOF")

var ErrorStructType = errors.New("EOF")

func main() {
    // true
    if ErrNamedType == New("EOF") {
        fmt.Println("Named Type Error")
    }
    // false
    if ErrorStructType == errors.New("EOF") {
        fmt.Println("Struct Type Error")
    }
}
```

不论是返回 string 还是返回封装了 string 的 struct，在判断时都是判断 string。为了避免意外的、判断相等的情况，应当返回 struct 指针。

另外实践中也建议在返回 `error` 时带上包名 + 具体错误信息：

```go
errors.New("bufio: buffer full")
```

error 即值，通过多参数返回支持同时返回 **异常信息** 与 **返回值**，实现处理异常逻辑（而不是引入 exception）。即建议在一次函数调用时返回错误要立即处理，而不是在整块代码中分别处理各种异常。

**人为控制 error** 可 **避免隐藏的控制流**，在函数签名中带上实现了 error interface 的对象，交由调用者来判定。如果一个函数返回了 `(value, error)`，则不能对这个 value 做任何假设，必须先判定 error。如果连 value 也不关心则可以忽略 error。

而 `panic` 意味着 fatal error。不能假设调用者来解决 panic，意味着代码不能继续运行（需要被 `recover` 兜底，否则是 runtime 退出）。使用 **多个返回值** 和 **简单的约定**，可让程序员知道什么时候出问题，并为真正的异常情况保留 panic（只用于索引越界、除数为 0、栈溢出等不可恢复 fatal error，不建议人为抛出 panic）。

## 异常类型

### Sentinel errors

即预定义的特定错误，源于计算机编程中使用特定值来表示不可能进行进一步处理的做法。

对于 Go 而言即使用特定的值来表示错误，类似 `io.EOF`，`syscall.ENOENT`。

```go
type Error string

func (e Error) Error() string {
    return string(e)
}
```

这种做法很不灵活，且存在以下问题：

- 使用 sentinel 值 **无法携带更多的详细信息**，调用方必须使用 == 将结果与预先声明的值进行比较。当想要提供更多上下文时，返回不同的错误将破坏相等性检查。一些有意义的 `fmt.Errorf` 携带上下文也会破坏调用者的 == ，调用者将被迫查看 `error.Error()` 方法的输出，以判断它是否与特定的字符串匹配。
- **sentinel errors 成为 API 公共部分**：如果公共函数返回特定值的错误，则该值必须是公共的、要有文档记录，使得 API 变得复杂。如果 API 定义了一个返回特定错误的 interface，则该接口的所有实现都将被限制为仅返回该错误（即使可以提供更具描述性的错误，比如 `io.Reader`）。像 `io.Copy` 这类函数需要 reader 的实现者告知调用者没有更多数据（比如返回 `io.EOF`），然而这不是错误。
- **sentinel errors 在包之间创建依赖**：例如检查错误是否等于 `io.EOF`，代码必须导入 io 包。当项目中的许多包导出错误值时就存在耦合，项目中的其他包必须导入这些错误值才能检查特定的错误条件（导致循环依赖）。

建议不依赖检查 `error.Error` 的输出：`error` 接口的 `Error` 方法主要是方便开发者调试用，而不是程序本身（编写测试可能会依赖这个返回），输出字符串用于记录日志、输出到 stdout 等。

更建议尽可能避免使用 sentinel errors。

### Error types

Error type 是实现了 `error` 接口的自定义类型，其一大改进是可封装底层错误、提供更多上下文。

比如 `os.PathError` 提供了底层执行的操作、哪个路径出了什么问题；下列代码中的 `MyError` 类型记录了文件和行号，调用者可断言转换成该类型，以获取更多的上下文信息：

```go
type MyError struct {
    Msg string
    File string
    Line int
}

func (e *MyError) Error() string {
    return fmt.Sprintf("%s:%d: %s", e.File, e.Line, e.Msg)
}

func test() error {
    return &MyError{"Something happened", "server.go", 42}
}

func main() {
    err := test() 
    swaitch err := err.(type) {
    case nil:
        // ...
    case *MyError:
        // ...
    default:
        // ...
    }
}
```

调用者要使用类型断言和类型 switch，就要让自定义的 error 变为 public。这种模型依然导致和调用者的强耦合，从而导致 API 变得脆弱。

建议尽量避免使用 error types，或至少避免作为公共 API 的一部分。

### Opaque errors

即不透明错误处理，只需返回错误而不假设其内容。它要求代码和调用者之间的耦合最少：作为调用者只需要知道操作的结果是否起作用，无需关心错误的内部。

```go
func fn() error {
    x, err := bar.Foo()
    if err != nil {
        return err
    }
    // x...
}
```

在少数情况下二分错误处理方法是不够的，所以应 **根据行为而不是类型去判断**。例如与进程外界进行交互，需调用方调查错误性质以确定重试该操作是否合理。

在这种情况下，可断言错误实现了特定的行为，而不是断言错误是特定的类型或值。

```go
// ========== 断言类型 ==========
// 对外暴露 Error 类型，需调用方断言是否该类型、并执行 Timeout、Temporary 完成完整判断。
type Error interface {
    error
    Timeout() bool
    Temporary() bool
}

if nerr, ok := err.(net.Error); ok && nerr.Temporary() {
    time.Sleep(1e9)
    continue
}
if err != nil {
    log.Fatal(err)
}

// ========== 断言行为 ==========
// temporary 接口非对外暴露，而是暴露 IsTemporary 方法以供调用方判断。
type temporary interface {
    Temporary() bool
}

func IsTemporary(err error) bool {
    te, ok := err.(temporary)
    return ok && te.Temporary()
}
```

由于只对它的行为感兴趣，可在不导入定义错误的包，或者不了解 err 底层类型的情况下实现。

## 处理异常

无错误的正常流程代码，没有多个层次。

```go
func parse(r io.Reader) (*Point, error) {
    var p Point
    if err := binary.Read(r, binary.BigEndian, &p.Longitude); err != nil {
        return nil, err
    }
    if err := binary.Read(r, binary.BigEndian, &p.Latitude); err != nil {
        return nil, err
    }
    if err := binary.Read(r, binary.BigEndian, &p.Distance); err != nil {
        return nil, err
    }
}
```

利用闭包可以封装通用错误的判断逻辑：

```go
func parse(r io.Reader) (*Point, error) {
    var p Point
    var err error
    read := func(data interface{}) {
        if err != nil {
            return
        }
        err = binary.Read(r, binary.BigEndian, data)
    }
    read(&p.Longitude)
    read(&p.Latitude)
    read(&p.Distance)
    if err != nil {
        return &p, err
    }
    return &p, nil
}
```

或清除中间不必要的 error，在最后统一返回：

```go
func CountLines(r io.Reader) (int error) {
    var (
        br = bufio.NewReader(r)
        lines int
        err error
    )
    for {
        _, err = br.ReadString('\n')
        // 先记录行数，再判断错误，否则会少记录一行。
        lines++
        if err != nil {
            break
        }
    }
    if err != io.EOF {
        return 0, err
    }
    return lines, nil
}

// 上述代码可优化为（只是选用更合适的 API）。
func CountLines(r io.Reader) (int error) {
    sc := bufio.NewScanner(r)
    lines := 0
    for sc.Scan() {
        lines++
    }
    return lines, sc.Err()
}
```

参考上面的 `sc`，在操作底层 I/O 时循环中没有错误判断逻辑，而是在最后统一返回 `sc.Err()`（使用结构体）。

```go
type Reader struct {
    r   io.Reader
    err error
}

func (r *Reader) read(data interface{}) {
    if r.err == nil {
        r.err = binary.Read(r.r, binary.BigEndian, data)
    }
}

func parse(input io.Reader) (*Point, error) {
    var p Point
    r := Reader{r: input}
    r.read(&p.Longitude)
    r.read(&p.Latitude)
    r.read(&p.Distance)
    if r.err != nil {
        return nil, r.err
    }
    return &p, nil
}
```

优化为流式接口：

```go
package main

import (
    "bytes"
    "encoding/binary"
    "fmt"
)

var b = []byte {0x48, 0x61, 0x6f, 0x20, 0x43, 0x68, 0x65, 0x6e, 0x00, 0x00, 0x2c}

var r = bytes.NewReader(b)

type Person struct {
    Name [10]byte
    Age uint8
    err error
}
func (p *Person) read(data interface{}) {
    if p.err == nil {
        p.err = binary.Read(r, binary.BigEndian, data)
    }
}

func (p *Person) ReadName() *Person {
    p.read(&p.Name)
    return p
}
func (p *Person) ReadAge() *Person {
    p.read(&p.Age)
    return p
}
func (p *Person) Print() *Person {
    if p.err == nil {
        fmt.Printf("Name=%s, Age=%d\n",p.Name, p.Age)
    }
    return p
}

func main() {
    p := Person{}
    p.ReadName().ReadAge().Print()
    fmt.Println(p.err)
}
```

如果涉及多个业务对象，将会丢失错误信息。

另外如果方法返回异常，调用方只是不断把异常返回到上层，最终在程序的顶部将错误输出到日志，此时没有异常调用堆栈的跟踪，难以发现触发错误的具体位置（而且无法再进行等值判断，也即与 setinel errors、type assertions 的做法不兼容）。因此建议遵循以下原则：

首先 **error 应该只处理一次**：而不是像下列代码判断异常、输出日志再向上抛（可能造成每层调用都做类似处理）。

```go
func WriteAll(w io.Writer, buf []byte) error {
    _, err := w.Write(buf)
    if err != nil {
        log.Println("unable to write:", err)
        return err
    }
    return nil
}
```

**避免部分执行结果与异常判断不一致**：应用程序处理错误要保证 100% 完整性。

下列代码中如果 JSON 序列化失败，buf 的内容未知，可能不包含任何内容或半写的 JSON 片段。

由于开发者在检查并记录错误后没有 return，损坏的缓冲区将被传递给 `WriteAll`，这可能会成功、配置文件被错误地写入，但函数返回正确。

```go
func WriteCOnfig(w io.Writer, conf *Config) error {
    buf, err := json.Marshal(conf)
    if err != nil {
        log.Printf...
    }
    if err := WriteAll(w, buf); err != nil {
        log.Println...
        return err
    }
    return nil
}
```

> Go 中的错误处理约定：**在出现错误的情况下，不能对其他返回值的内容做出任何假设**。

**避免无效、冗余的日志输出**：日志记录与错误无关且对调试没有帮助的信息应被视为噪音，错误应该被日志记录且此后不再报告。

### 异常封装

封装 error 能保存完整的堆栈信息，同时避免重复的处理。

由于目前 Go 没有内置更好的处理方法。利用第三方库 [pkg/errors](https://github.com/pkg/errors) 为错误值添加上下文，可由开发者或机器检查。

```go
func ReadFile(path string) ([] byte, error) {
    f, err := os.Open(path)
    if err != nil {
        // 包装错误根源信息。
        return nil, errors.Wrap(err, "open failed")
    }
}

func ReadConfig() ([]byte, error) {
    home := os.Getenv("HOME")
    config, err := ReadFile(filepath.Join(home, ".settings.xml"))
    return config, errors.WithMessage(err, "could not read config")
}

func main() {
    _, err := ReadConfig()
    if err != nil {
        // 得到错误根源信息：可用于 sentinel、type 判断。
        fmt.Printf("original error: %T %v\n", errors.Cause(err), errors.Cause(err))
        // 得到完整堆栈信息。
        fmt.Printf("stack trace: \n%+v\n", err)
    }
    // if err != nil {
    //     fmt.Println(err)
    //     os.Exit(1)
    // }
}
```

有了 [pkg/errors](https://github.com/pkg/errors)，调用内层函数时可以不记录日志，而直接逐层往上抛、直到最上层函数，再输出完整的日志。

使用须知：

- 应用代码中，使用 `errors.New` 或者 `errros.Errorf` 返回错误（`return errors.Errorf("...")`）。
- 如果调用其他包内的函数，可简单的直接返回（`if err != nil return err`）；如果和标准库或其他库进行协作，考虑使用 `errors.Wrap` 或者 `errors.Wrapf` 保存堆栈信息（`return errors.Wrap(err, "...")`）。
- 内层函数直接返回错误，而不是每个错误产生的地方到处打日志；在程序的顶部或者是工作的 goroutine 顶部（请求入口），使用 `%+v` 把堆栈详情记录；或使用 `errors.Cause` 获取 root error，再进行和 sentinel error 判定。
- 只建议在业务应用中 wrap error，具有最高可重用性的包只能返回根错误值（避免重复输出异常堆栈）。此机制与 Go 标准库中使用的相同（kit 库的 `sql.ErrNoRows`）。
- 如果函数不打算处理错误，要用 wrap errors 要封装足够的上下文并将其返回到调用堆栈中（例如额外的上下文可以是使用的输入参数或失败的查询语句）。
- 一旦确定函数处理错误，返回的就不再是错误。如果函数仍然需要返回，则它不能返回错误值。它应该只返回零（比如降级处理中返回了降级数据，然后需要 return nil）。

> 在业务代码中，比如操作数据库时 DAO 层出现 sql.ErrNoRows 等错误，不应该将这个 error 封装（Wrap）再抛给上层。
>
> 应该把 ErrNoRows 在 DAO 层就处理好，替换为一个自定义的错误返回，其它错误 Wrap 返回。

## Go 1.13 errors

### 1.13 之前

最简单的错误检查：

```go
if err != nil {
    //...
}
```

setinel error：

```go
var ErrNotFound = errors.New("not found")
// ...
if err == ErrNotFound {
    //...
}
```

使用实现了 error interface 的自定义 error struct，断言获取更丰富的上下文

```go
type NotFoundError struct {
    Name string
}

func (e *NotFoundError) Error() string {
    return e.Name + ": not found"
}

if e, ok : err.(*NotFoundError); ok {
    //...
}
```

函数在调用栈中添加信息向上传递错误，例如对错误发生时发生的情况的简要描述。

```go
if err != nil {
    return fmt.Errorf("decompress %v: %v", name, err)
}
```

创建新错误 `fmt.Errorf`，丢弃原始错误中除文本外的所有内容。有时可能需要定义一个包含底层错误的新错误类型，并将其保存以供代码检查：

```go
type QueryError struct {
    Query string
    Err error
}

if e, ok := err.(*QueryError); ok && e.Err == ErrPermission {
    // ...
}
```

### 1.13 引入

1.13+ 为 errors 和 fmt 标准库包引入了新特性以简化处理包含其他错误的错误，包含另一个错误的 error 可以实现返回底层错误的 `Unwrap` 方法。比如 e1 包装 e2：`e1.Unwrap()` 返回 e2 ，并可以展开 e1 以获得 e2。

按照此约定可以为 `QueryError` 类型指定一个 `Unwrap` 方法，返回其包含的错误：

```go
func (e *QueryError) Unwrap() error {
    return e.Err
}
```

errors 包还包含两个用于检查错误的新函数：`Is` 和 `As`。

```go
// 展开，找到根源错误（内部做了 Unwrap）
if errors.Is(err, ErrNotFound) {
    //...
}

// 类型转换（内部做了 Unwrap）
var e *QueryError
if errors.As(err, &e) {
    //...
}
```

使用 `fmt.Errorf` 向错误添加附加信息：

```go
if err != nil {
    return fmt.Errorf("drcompress %v: %w", name, err)
}

// %w 包装错误可用于 errors.ls 和 errors.As
err := fmt.Errorf("access denied: %w", ErrPermission)
if errors.Is(err, ErrPermission) {
    //...
}
```

`%w` 包装后，其内部处理是：

```go
type wrapError struct {
    msg string
    err error
}

func (e *wrapError) Error() string {
    return e.msg
}

func (e *wrapError) Unwrap() error {
    return e.err
}
```

由于 1.13 的 errors 依然不支持输出堆栈信息，所以需要结合 [pkg/errors](https://github.com/pkg/errors) 使用：

```go
import (
    "errors"
    "fmt"
    xerrors "github.com/pkg/errors"
)

var errMy = errors.New("My")

func test0() error {
    // 要获取堆栈信息，应使用 Wrapf 代替 %w。
    return xerrors.Wrapf(errMy, "test0 failed")
}

func test1() error {
    return test0()
}

func test2() error {
    return test1()
}

func main() {
    err := test2()
    fmt.Printf("main: %+v\n", err)
}
```

## Go 2 Error Inspection

- [Proposal: Go 2 Error Inspection](https://go.googlesource.com/proposal/+/master/design/29934-error-values.md)
- [Error Handling — Draft Design](https://go.googlesource.com/proposal/+/master/design/go2draft-error-handling.md)
- [Error Handling — Problem Overview](https://go.googlesource.com/proposal/+/master/design/go2draft-error-handling-overview.md)

## 参考

- [Go 语言的错误处理机制引发争议](https://www.infoq.cn/news/2012/11/go-error-handle/)
- [Frequently Asked Questions (FAQ) - The Go Programming Language](https://golang.org/doc/faq#exceptions)
- [Error Handling In Go, Part I](https://www.ardanlabs.com/blog/2014/10/error-handling-in-go-part-i.html)
- [Error Handling In Go, Part II](https://www.ardanlabs.com/blog/2014/11/error-handling-in-go-part-ii.html)
- [Design Philosophy On Logging](https://www.ardanlabs.com/blog/2017/05/design-philosophy-on-logging.html)
- [Error Handling in Go. Mastering pragmatic error handling in your Go code](https://medium.com/gett-engineering/error-handling-in-go-53b8a7112d04)
- [Error Handling in Go 1.13. Keep up with the latest additions to Go error handling](https://medium.com/gett-engineering/error-handling-in-go-1-13-5ee6d1e0a55c)
- [Why Go’s Error Handling is Awesome](https://rauljordan.com/2020/07/06/why-go-error-handling-is-awesome.html)
- [Effective error handling in Go](https://morsmachine.dk/error-handling)
- [Go 1.13: xerrors](https://crawshaw.io/blog/xerrors)
- [Why Go gets exceptions right](https://dave.cheney.net/2012/01/18/why-go-gets-exceptions-right)
- [Errors and Exceptions, redux](https://dave.cheney.net/2015/01/26/errors-and-exceptions-redux)
- [Error handling vs. exceptions redux](https://dave.cheney.net/2014/11/04/error-handling-vs-exceptions-redux)
- [Inspecting errors](https://dave.cheney.net/2014/12/24/inspecting-errors)
- [Constant errors](https://dave.cheney.net/2016/04/07/constant-errors)
- [Eliminate error handling by eliminating errors](https://dave.cheney.net/2019/01/27/eliminate-error-handling-by-eliminating-errors)
- [Stack traces and the errors package](https://dave.cheney.net/2016/06/12/stack-traces-and-the-errors-package)
- [Don’t just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)
- [Errors are values](https://blog.golang.org/errors-are-values)
- [Error handling and Go](https://blog.golang.org/error-handling-and-go)
- [Working with Errors in Go 1.13](https://blog.golang.org/go1.13-errors)
- [command center: Error handling in Upspin](https://commandcenter.blogspot.com/2017/12/error-handling-in-upspin.html)
- [Golang Error Handling lesson by Rob Pike](https://jxck.hatenablog.com/entry/golang-error-handling-lesson-by-rob-pike)
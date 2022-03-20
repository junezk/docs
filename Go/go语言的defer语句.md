# go语言的defer语句

## defer的语法

defer后面必须是函数调用语句，不能是其他语句，否则编译器会出错。



```go
package main

import "log"

func foo(n int) int {
    defer n++
    //defer log.Println("n=", n)
    return n
}

func main() {
    var i int = 100
    foo(i)
}
```

这个例子中defer后面使用的是n++指令，不是一个函数调用语句，编译器就报错：



```go
# command-line-arguments
./main.go:6: expression in defer must be function call
./main.go:6: syntax error: unexpected ++ at end of statement
```

## defer的基本功能

defer后面的函数在defer语句所在的函数执行结束的时候会被调用；我们查看一下汇编吗，看看defer是在什么时候被执行的：
 定义两个函数foo1和foo2，功能和代码都是一样，只是其中一个包含defer语句，另一个没有。



```go
func foo1(i int) int {
    i = 100
    i = 200
    return i
}

func foo2(i int) int {
    i = 100
    defer foo()
    i = 200
    return i
}
```

这是foo1的汇编代码：



```bash
func foo1(i int) int {
  44d660:   48 c7 44 24 10 00 00    movq   $0x0,0x10(%rsp)
  44d667:   00 00
    i = 100
  44d669:   48 c7 44 24 08 64 00    movq   $0x64,0x8(%rsp)
  44d670:   00 00
    i = 200
  44d672:   48 c7 44 24 08 c8 00    movq   $0xc8,0x8(%rsp)
  44d679:   00 00
    return i
  44d67b:   48 c7 44 24 10 c8 00    movq   $0xc8,0x10(%rsp)
  44d682:   00 00
  44d684:   c3                      retq
  ...
}
```

再看foo2的汇编代码：



```bash
func foo2(i int) int {
  44d690:   64 48 8b 0c 25 f8 ff    mov    %fs:0xfffffffffffffff8,%rcx
  44d697:   ff ff
  44d699:   48 3b 61 10             cmp    0x10(%rcx),%rsp
  44d69d:   76 70                   jbe    44d70f <main.foo2+0x7f>
  44d69f:   48 83 ec 18             sub    $0x18,%rsp
  44d6a3:   48 89 6c 24 10          mov    %rbp,0x10(%rsp)
  44d6a8:   48 8d 6c 24 10          lea    0x10(%rsp),%rbp
  44d6ad:   48 c7 44 24 28 00 00    movq   $0x0,0x28(%rsp)
  44d6b4:   00 00
    i = 100
  44d6b6:   48 c7 44 24 20 64 00    movq   $0x64,0x20(%rsp)
  44d6bd:   00 00
    defer foo()
  44d6bf:   c7 04 24 00 00 00 00    movl   $0x0,(%rsp)
  44d6c6:   48 8d 05 93 fb 01 00    lea    0x1fb93(%rip),%rax        # 46d260 <go.func.*+0x41>
  44d6cd:   48 89 44 24 08          mov    %rax,0x8(%rsp)
  44d6d2:   e8 e9 3e fd ff          callq  4215c0 <runtime.deferproc>
  44d6d7:   85 c0                   test   %eax,%eax
  44d6d9:   75 24                   jne    44d6ff <main.foo2+0x6f>
  44d6db:   eb 00                   jmp    44d6dd <main.foo2+0x4d>
    i = 200
  44d6dd:   48 c7 44 24 20 c8 00    movq   $0xc8,0x20(%rsp)
  44d6e4:   00 00
    return i
  44d6e6:   48 c7 44 24 28 c8 00    movq   $0xc8,0x28(%rsp)
  44d6ed:   00 00
  44d6ef:   90                      nop
  44d6f0:   e8 6b 48 fd ff          callq  421f60 <runtime.deferreturn>
  44d6f5:   48 8b 6c 24 10          mov    0x10(%rsp),%rbp
  44d6fa:   48 83 c4 18             add    $0x18,%rsp
  44d6fe:   c3                      retq
  ...
}
```

通过比较很容易看出foo2有两处需要注意，第一处是defer foo()语句的翻译，这个翻译我没有细看懂，我猜是准备foo的函数参数(如果有)，然后保存这些参数值和foo的地址，注册到系统(runtime.deferproc)；另一处是return指令的翻译，return指令的执行分三步，第一步拷贝return值到返回值内存地址，第二步会调用runtime.deferreturn去执行前面注册的defer函数，第三部再执行ret汇编指令。

有两个常见的defer语句应用场景是：

- file对象打开后的自动关闭

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
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

    // other codes
    return io.Copy(dst, src)
}
```

在打开输入文件输出文件后，不管后面的代码流程如何影响，这两个文件能够被自动关闭。

- mutex对象锁住后的自动释放



```go
func foo(...) {
    mu.Lock()
    defer mu.Unlock()

    // code logic
}
```

确保mu锁能够在函数foo退出之后自动释放。

## 注意0：如何让defer函数在宿主函数的执行中间执行

我们注意到defer函数的执行是在defer指令所在函数的运行结束之后，那么如何才能在所在函数的中间就释放呢，比如前面例子，在foo入口锁住了lock，而如果foo后半段的代码运行时间比较长，而此时又不需要继续保持住锁，该怎么办呢？



```go
func foo() {
  mu.Lock();
  defer mu.Unlock();

  object, ok := map[key];
  if (!ok) {
      return
  }
  // time-consuming operating with object
  ...
}
```

我们希望能在time-consuming operation 之前就释放锁，而不是等到整个foo返回。这有两个办法，一个是根据逻辑，把foo拆分两部分，前半部分需要锁，后半部分不需要锁；另一个办法是使用匿名函数：



```go
package main

import "log"
import "time"
import "sync"

var mu sync.Mutex


func lock() {
    mu.Lock()
    log.Printf("lock")
}

func unlock() {
    mu.Unlock()
    log.Printf("unlock")
}

func foo() int {
    lock()

    func() {
        log.Printf("entry inner")
        defer unlock()
        log.Printf("exit inner")
    }()

    time.Sleep(1 * time.Second)
    log.Printf("return")
    return 0;
}

func main() {
    r := foo()
    log.Println("r=",r)
}
```

运行结果：



```kotlin
$ ./main 
2017/09/30 22:18:58 lock
2017/09/30 22:18:58 inner
2017/09/30 22:18:58 unlock
2017/09/30 22:18:59 return
2017/09/30 22:18:59 r= 0
```

从日志我们可以看出mu锁在sleep语句之前已经被释放了，而不是需要等到foo函数结束的时候才释放。

## 注意1：多个defer的执行顺序

如果函数里面有多条defer指令，他们的执行顺序是反序，即后定义的defer先执行。



```go
package main

import "log"
import "time"

func foo(n int) int {
    defer log.Println("1111")
    time.Sleep(1 * time.Second)
    defer log.Println("2222")
    time.Sleep(1 * time.Second)
    defer log.Println("3333")
    time.Sleep(1 * time.Second)
    return n
}

func main() {
    var i int = 100
    foo(i)
}
```

运行结果如下，可以看出他们的调用顺序：



```undefined
2017/09/30 19:22:03 3333
2017/09/30 19:22:03 2222
2017/09/30 19:22:03 1111
```

## 注意2：defer函数参数的计算时间点

defer函数的参数是在defer语句出现的位置做计算的，而不是在函数运行的时候做计算的，即所在函数结束的时候计算的。



```go
package main

import "log"

func foo(n int) int {
    log.Println("n1=", n)
    defer log.Println("n=", n)
    n += 100
    log.Println("n2=", n)
    return n
}

func main() {
    var i int = 100
    foo(i)
}
```

其运行结果是：



```undefined
2017/09/30 19:25:10 n1= 100
2017/09/30 19:25:10 n2= 200
2017/09/30 19:25:10 n= 100
```

可以看到defer函数的位置时n的值为100，尽管在函数foo结束的时候n的值已经是200了，但是defer语句本身所处的位置时刻，即foo函数入口时n为100，所以最终defer函数打印出来的n值为100。

## 注意3：如何在defer语句里面使用多条语句

前面我们提到defer后面只能是一条函数调用指令；而实际情况下经常会需要逻辑运行，会有分支，条件，而不是简单的一个log.Print指令；那怎么处理这种情况呢，我们可以把这些逻辑指令一起定义成一个函数，然后再调用这些函数就行了，命名函数或者匿名函数都可以，下面是一个匿名函数的例子：



```go
package main

import "log"
import _ "time"

func foo(n int) int {
    log.Println("n1=", n)
    defer func() {
        n += 100
        log.Println("n=", n)
    }()
    n += 100
    log.Println("n2=", n)
    return n
}

func main() {
    var i int = 100
    foo(i)
}
```

运行结果：



```undefined
2017/09/30 19:30:58 n1= 100
2017/09/30 19:30:58 n2= 200
2017/09/30 19:30:58 n= 300
```

眼尖的同学会发现其中的问题；为什么n打印出来是300呢，不是明明说好defer函数的参数值在它出现时候计算，而不是在运行的时候计算的吗，n应该打印出200才对啊？
 同学，仔细看一下原文：defer函数的参数在defer语句出现的位置计算，不是在defer函数运行的时刻计算；人家明明说的很清楚，defer函数的参数，请问这里n是参数吗，不是哎，这里引用的是宿主函数的局部变量，而不是参数；所以它拿到的是运行时刻的值。

这就引发出下一个注意事项。

## 注意4：defer函数会影响宿主函数的返回值



```go
package main

import "log"

func foo1(i *int) int {
    *i += 100
    defer func() { *i += 200 }()
    log.Printf("i=%d", *i)
    return *i
}

func foo2(i *int) (r int) {
    *i += 100
    defer func() { r += 200 }()
    log.Printf("i=%d", *i)
    return *i
}

func main() {
    var i, r int

    i,r = 0,0
    r = foo1(&i)
    log.Printf("i=%d, r=%d\n", i, r)

    i,r = 0,0
    r = foo2(&i)
    log.Printf("i=%d, r=%d\n", i, r)
}
```

运行结果为：



```go
$ go build main.go && ./main 
2017/09/30 20:01:00 i=100
2017/09/30 20:01:00 i=300, r=100
2017/09/30 20:01:00 i=100
2017/09/30 20:01:00 i=100, r=300
```

这个例子其实有一点拗口的。
 foo1 return指令前(i==100, ret==0)，return指令后(i==100, ret=100)，然后调用defer函数后(i==300，r==100)，defer函数增加了i；main函数收到(i==300, r==100)
 foo2 return指令前(i==100, ret==0)，return指令后(i==100, ret=100)，然后调用defer函数后(i==100，r==300)，defer函数增加了ret；main函数收到(i==100, r==300)
# 聊聊Go应用输出日志的工程实践

[本文永久链接](https://tonybai.com/2022/03/05/go-logging-practice) – https://tonybai.com/2022/03/05/go-logging-practice

Go隶属于后端语言，以开发各类服务、中间件和系统平台见长。日常学习Go语言时，日志不是不可或缺的，甚至是无需考虑的，但是一旦到真正的Go的工程实践中，输出日志便是我们绕不过去的、必须面对的问题。

Go开发的服务大多具有连续和自主运行的属性，通常都是7×24小时运行的，开发运维人员不可能一直盯着服务的运行状态，这就需要记录服务的运行日志。而日志的本质正是**留档待查**。通过日志，我们可以：

- 查看系统运行状态
- 发现异常
- 审计行为
- 辅助问题的诊断

在如今云原生时代，可观测性(Observability)大行其道，日志被称为[“可观测性的三大支柱”](https://www.oreilly.com/library/view/the-future-of/9781098118433/)之一（另两个是[度量(metrics)](https://mp.weixin.qq.com/s/pFTLGOsyl5DgohbSTBSr-w)与跟踪(tracing)），是云原生系统devops不可或缺的重要组成部分。

那么在Go工程实践中，Go对日志输出支持的现状是怎样的？存在哪些问题？常用的问题解决套路是什么？以及社区的努力与期望又是什么？在这一篇文章中，我就围绕上述这些问题来聊一聊。

## 一. Go日志现状

[Go是“自带电池”的语言](https://www.imooc.com/read/87/article/2341)，开箱即用。这就意味着在Go标准库中有现成的[log包](https://pkg.go.dev/log)可供开发者使用。

### Go标准库log包

作为Go标准库的一部分，Go log包使用起来十分简单，你只需导入log包并无需做任何创建与设置操作即可输出log到标准错误(stderr)，这是因为log包内置了一个名为std的默认logger：

```go
// $GOROOT/src/log/log.go
var std = New(os.Stderr, "", LstdFlags)
```

这种内置一个默认logger实例的模式也被很多第三方log包所效仿。下面是使用默认logger输出日志的简单例子：

```go
import "log"

func main() {
    log.Println("hello, go log")
}
```

上面示例程序输出：

```
2022/02/27 14:40:41 hello, go log
```

我们看到：go log提供的是printf-like风格的log API，这也意味着将代码中的log换成fmt，代码同样可以正常运行。go log支持三类API：

```
Println/Printf
Panicln/Panicf // 相当于PrintX + panic
Fatalln/Fatalf // 相当于PrintX + exit
```

go log包通过下面三个导出方法可以实现对logger的一些“定制”：

```
func SetFlags(flag int)       // 主要用于设置日志的时间戳格式，也可以设置前缀字符串在log输出中的位置，默认flag为LstdFlags
func SetOutput(w io.Writer)   // 设置日志的输出目的地，比如文件、网络socket、syslog等
func SetPrefix(prefix string) // 设置日志前缀
```

但go log包不支持设置log level并按level输出，内置std logger同样不支持log level。如果对log level有需求，可基于go log包进行二次封装。下面就是一个简单的封装demo：

```go
package main

import (
    "io"
    "log"
    "os"
    "sync/atomic"
)

// log level
const (
    LDEBUG = iota + 1 // 1
    LWARN             // 2
    LINFO             // 3
    LERROR            // 4
    LFATAL            // 5
)

type myLogger struct {
    level       int64
    w           io.Writer
    debugLogger *log.Logger
    warnLogger  *log.Logger
    infoLogger  *log.Logger
    errLogger   *log.Logger
    fatalLogger *log.Logger
}

func New(w io.Writer, level int64, flag int) *myLogger {
    if w == nil {
        w = os.Stderr
    }

    if flag <= 0 {
        flag = log.LstdFlags
    }

    return &myLogger{
        w:           w,
        level:       level,
        debugLogger: log.New(w, "[DEBUG] ", flag|log.Lmsgprefix),
        warnLogger:  log.New(w, "[WARN] ", flag|log.Lmsgprefix),
        infoLogger:  log.New(w, "[INFO] ", flag|log.Lmsgprefix),
        errLogger:   log.New(w, "[ERROR] ", flag|log.Lmsgprefix),
        fatalLogger: log.New(w, "[FATAL] ", flag|log.Lmsgprefix),
    }
}

func (l *myLogger) SetLevel(level int64) {
    if level < LDEBUG || level > LFATAL {
        return
    }

    atomic.StoreInt64(&l.level, level)
}

func (l *myLogger) Debugln(v ...interface{}) {
    if atomic.LoadInt64(&l.level) > LDEBUG {
        return
    }
    l.debugLogger.Println(v...)
}

func (l *myLogger) Debugf(format string, v ...interface{}) {
    if atomic.LoadInt64(&l.level) > LDEBUG {
        return
    }
    l.debugLogger.Printf(format, v...)
}

func (l *myLogger) Infoln(v ...interface{}) {
    if atomic.LoadInt64(&l.level) > LINFO {
        return
    }
    l.infoLogger.Println(v...)
}

func (l *myLogger) Infof(format string, v ...interface{}) {
    if atomic.LoadInt64(&l.level) > LINFO {
        return
    }
    l.infoLogger.Printf(format, v...)
}

func main() {
    logger := New(nil, LWARN, 0)
    logger.Infoln("info level log demo")
    logger.Debugln("debug level log demo")
}
```

运行这个demo，输出日志如下：(由于设置的是LWARN级别，所以仅会输出info级别的日志，debug级别日志将被忽略)：

```
2022/02/27 15:41:01 [INFO] info level log demo
```

### glog

github的golang组织下还提供了一个Google内部使用的、Go版本的[glog](https://github.com/golang/glog)。

> 注：glog是google内部自用的log包，不接受外部的添加feature的PR。

glog支持分级日志，并提供了按特定log level输出日志的专用API，比如：InfoXx、WarningXx、ErrorXx、FatalXx等，下面是使用glog的一个示例：

```go
package main

import (
    "flag"

    "github.com/golang/glog"
)

func main() {
    flag.Parse()
    glog.Info("Prepare to repel boarders")

    if glog.V(2) {
        glog.Info("Starting transaction...")
    }
    glog.V(2).Infoln("Processed", 5, "elements")
    glog.Flush()
}
```

glog通过flag来指定log level，所以在使用glog输出日志前必须先调用[Go标准库flag包](https://mp.weixin.qq.com/s/5ZrZ35ycwtJE2if6rs9dEQ)的Parse函数解析命令行flag。执行这个示例，输出结果如下：

```
$go run main.go -logtostderr -v=2
I0227 16:14:35.968922   40869 main.go:12] Prepare to repel boarders
I0227 16:14:35.969024   40869 main.go:15] Starting transaction...
I0227 16:14:35.969027   40869 main.go:17] Processed 5 elements
```

> 注意：使用-logtostderr使得日志输出到标准错误，否则默认输出到系统临时文件目录的临时文件中；如果要输出日志到特定目录，可以去掉-logtostderr，并指定-log_dir，不过我们无法指定日志文件名，glog有一套内定的日志文件自动命名机制。

虽然glog与标准库log包有很多不同点，但它们总体来说还是属于一类log包，它们具有一个共同的特点：**开发者需自己将各个字段通过类Printf API组成一条非结构化的日志，这样的日志便于human阅读，但不便于机器阅读(解析)**。

### 结构化日志包

好了，这就引出了另一类log包，它们的特点与上面的标准库log、golang/glog正好相反：**开发者使用这一类log包时无需自组织日志格式，只需通过key-value的方式给出要输出的字段，log包就会将这些字段自动整理为一条类似下面这样的结构化的日志**：

```
// 以zap包输出为例：

{"level":"info","ts":1646256841.204795,"caller":"zap/testzap.go:13","msg":"failed to fetch URL","url":"http://tonybai.com","attempt":3,"backoff":1}
```

我们很容易看出这是一条合法的json数据，这类log包多以json作为结构化日志的输出形式。并且，即便某些包提供了类Printf的API，其实也只是将自组的字符串作为一个特定key(比如zap包使用msg)的value。

**Go社区开源的第三方log包绝大多数都属于此类结构化日志包**，包括github排名最靠前、采用最为广泛的两个包[sirupsen/logrus](http://tonybai.com/2018/04/06/the-problems-i-encountered-when-writing-go-code-issue-3rd)和[uber-go/zap](https://mp.weixin.qq.com/s/cU5y465F7bhzVk6cHp0qVA)：

![img](https://tonybai.com/wp-content/uploads/go-logging-practice-3.png)

笔者在生产中使用的是uber/zap，关于zap包的使用方法，我在之前的一篇[《一文告诉你如何用好uber开源的zap日志》](https://mp.weixin.qq.com/s/cU5y465F7bhzVk6cHp0qVA)有详细讲解，这里就不赘述了。

结构化日志包之所以大受欢迎，主要还是因为其容易被机器“阅读”的特点，**如今将日志“灌入”类[ELK](http://tonybai.com/2017/03/03/implement-kubernetes-cluster-level-logging-with-fluentd-and-elasticsearch-stack)的集中日志平台中备查已经不是最佳实践了，而是必经之路**。

有人说性能也是考量因素。也没错，至少我在选型时会重点考虑性能高的log包。比如：在并发benchmark中，zap远胜std log：

```
goos: linux
goarch: amd64
pkg: demo
cpu: Intel(R) Core(TM) i7-9700 CPU @ 3.00GHz
BenchmarkGolog-4         4126830        290.7 ns/op    24 B/op      1 allocs/op
BenchmarkGooglelog-4     711428    1427 ns/op      216 B/op     2 allocs/op
BenchmarkZap-4      11572719        97.08 ns/op     0 B/op      0 allocs/op
PASS
```

不过你可能并不知晓：在单goroutine下，zap性能其实不如std log：

```
goos: linux
goarch: amd64
pkg: demo
cpu: Intel(R) Core(TM) i7-9700 CPU @ 3.00GHz
BenchmarkGolog-4         4470559        273.4 ns/op    24 B/op      1 allocs/op
BenchmarkGooglelog-4     789846    1363 ns/op      216 B/op     2 allocs/op
BenchmarkZap-4       2934202        400.4 ns/op     0 B/op      0 allocs/op
PASS
```

不过，如今哪个作为后端服务的Go应用不启动几个goroutine呢，并发才是log应用的主场景。

无论是选择非结构化的std log包，还是结构化的、高性能的zap log包，在应用过程中依然会遇到一些问题。接下来，我们就来看看在log的工程实践中存在的主要问题。

## 二. 工程实践中存在的主要问题

工程实践中，在处理log这块儿你会遇到很多问题，比如：如何日志选型、如何评估日志性能、如何进行日志输出设置（包括：设置日志输出的目的io.Writer、日志格式、日志轮转(rotate)与归档(archive)）等。

但这些都不是主要问题，日志选型可参考社区使用最广泛的log包；日志性能用简单的benchmark就可判断；日志设置呢，看看各个包提供的doc与example大多都可以搞定。

那主要问题是什么呢？其实是**log适配**。为什么来会有log适配问题呢？我来举一个例子大家就明白了。

现在我有一个Go项目P1，P1依赖uber/zap包输出log。通过前面对zap的了解，我们知道zap输出的日志样式是这样的：

```
{"level":"warn","logtime":"2022-02-27T15:24:57+08:00","caller":"xx/log.go:19","msg":"[f:1,l:33372,t:4,c:33372,a:33372] [00103:00001] t24 cast vote from n00003 index 33373 term 24, log term: 5"}
```

现在P1要基于[dragonboat包](https://github.com/lni/dragonboat)实现分布式强一致的数据同步，不过dragonboat采用了自定义的logger，其内部模块输出的日志的样式是这样的：

```
2022-02-27 16:25:40.259479 I | raft: [f:1,l:33375,t:26,c:33374,a:33374] [00101:00001] t26 became leader
```

我们看到：问题出现了：**P1输出日志一些是zap log格式的，一些是dragonboat自定义格式的，不仅human阅读困难，送给机器也无法解析**。

好在dragonboat包比较厚道，对外部提供了设置其内部logger的导出函数**SetLoggerFactory**，但是前提是设置的logger实例的类型应该满足下面接口类型：

```
// dragonboat包中的logger/logger.go
type ILogger interface {
    SetLevel(LogLevel)
    Debugf(format string, args ...interface{})
    Infof(format string, args ...interface{})
    Warningf(format string, args ...interface{})
    Errorf(format string, args ...interface{})
    Panicf(format string, args ...interface{})
}
```

好在zap包提供了“Sugar”方式的Printf-like的API，可以用于适配上述ILogger接口，我们以dragonboat提供的[dragonboat-example](https://github.com/lni/dragonboat-example)中的helloworld为例，让该项目适配zap风格的log，我们为helloworld项目提供一个log.go文件，源码如下：

```go
//helloworld/log.go

package main

import (
    "github.com/lni/dragonboat/v3/logger"
    "go.uber.org/zap"
)

type LogLevel = logger.LogLevel

type Mylogger struct {
    *zap.Logger
}

func (l *Mylogger) SetLevel(level LogLevel) {
}

func (l *Mylogger) Warningf(format string, args ...interface{}) {
    l.Logger.Sugar().Warnf(format, args...)
}

func (l *Mylogger) Debugf(format string, args ...interface{}) {
    l.Logger.Sugar().Debugf(format, args...)
}

func (l *Mylogger) Errorf(format string, args ...interface{}) {
    l.Logger.Sugar().Errorf(format, args...)
}

func (l *Mylogger) Infof(format string, args ...interface{}) {
    l.Logger.Sugar().Infof(format, args...)
}

func (l *Mylogger) Panicf(format string, args ...interface{}) {
    l.Logger.Sugar().Panicf(format, args...)
}

var _ logger.ILogger = (*Mylogger)(nil)

var factory = func(pkgName string) logger.ILogger {
    logger, _ := zap.NewProduction()
    return &Mylogger{
        Logger: logger,
    }
}
```

然后在helloworld的main.go中使用dragonboat提供的SetLoggerFactory重新设置一下dragonboat内部所使用的logger实例：

```go
func main() {
    ... ...
    logger.SetLoggerFactory(logger.Factory(factory))

    // change the log verbosity
    logger.GetLogger("raft").SetLevel(logger.ERROR)
    logger.GetLogger("rsm").SetLevel(logger.WARNING)
    logger.GetLogger("transport").SetLevel(logger.WARNING)
    logger.GetLogger("grpc").SetLevel(logger.WARNING)
    ... ...
}
```

这样修改后再运行helloworld这个示例，我们便可以看到它输出的日志样式已经变成了zap风格的了：

```
{"level":"warn","ts":1646300185.9349403,"caller":"helloworld/log.go:18","msg":"[f:1,l:4,t:2,c:4,a:4] [00128:00001] t3 received 2 votes and 0 rejections, quorum is 2"}
{"level":"info","ts":1646300185.9352179,"caller":"helloworld/log.go:30","msg":"[f:1,l:5,t:3,c:4,a:4] [00128:00001] t3 became leader"}
```

到这里，有人可能会说：**适配一个log包似乎也还好吧**！其实碰到像dragonboat这样的包，只能说明我们运气好，它提供了供你去适配的logger接口，如果它没有提供这样的接口呢？如果zap没有提供Printf-like的API呢？我们都无法像上面这样将zap与dragonboat相融在一起。

此外，我要说这个问题的严重性在于其**不确定性**，这种不确定来自于**无法预料将来与某个新引入的第三方包是否存在日志接口格式上的兼容性**，而问题的根源则在于[**目前Go官方没有一个统一log接口供所有项目去遵守**](https://github.com/golang/go/issues/13182)。

在前面的log现状中我们说过，Go标准库的Logger是一个结构体，是一个具体的实现，不具备解耦的作用。

## 三. 临时解决方法、社区努力与期望

那遇到此类工程问题，我们该如何做呢？在Go没有提供统一log接口的情况下，我们可能遇到下面的两种情况(项目P，依赖D)：

- 如果依赖D像上面dragonboat提供了log接口用于适配，且log接口的风格与P自用log包的风格兼容（比如都支持Printf-like API），那么我们需要像上面例子中那样，在项目P中提供一个用于适配D的logger实现；
- 如果依赖D没有提供log接口用于适配，且其输出的日志风格与P自用log风格不一致或是D的log API风格与P自用log包 API不兼容，这种情况下，要么向D库作者提issue或pr（多数情况可能被拒），要么更实际一点，自己fork D项目并二次开发，替换其中的logger或增加用于适配的接口。

要一劳永逸的解决log工程上的问题，还是需要Go官方定义统一的log接口，社区在这方面不能说不努力，2017年Go社区一批大神就成立了统一log接口委员会，并[商讨](https://groups.google.com/g/golang-dev/c/F3l9Iz1JX4g)出了一个[proposal](https://docs.google.com/document/d/1shW9DZJXOeGbG9Mr9Us9MiaPqmlcVatD_D8lrOXRNMU/edit)试图在Go标准库中增加统一的log接口，但提案没有没Go核心团队接纳(accept)。之后，该话题逐渐沉寂下来，形成如今的现状。

打不进核心，Go社区只能走外围路线，于是出现了许多Go社区版的类java的[SLF4J](https://www.slf4j.org/)(Simple Logging Facade for Java)的项目，比如[logr](https://github.com/go-logr/logr)、[slf4go](https://github.com/go-eden/slf4go)等。目前比较活跃的log API是logr项目，它提供了大部分主流log包的适配实现。如果你没有更好的办法，可以考虑一下采用logr的API规范。

## 四. 小结

log在工程实践中的问题虽然没有被单独拿出来放在[Go年度用户调查结果](https://go.dev/blog/survey2020-results)中，但它确是客观存在的，我觉得它应该属于“Missing or immature libraries”调查项的一部分，这样的问题其实越早解决掉越好，将会给开发者带来很大的便利，节省很多花在实现log适配上的时间。
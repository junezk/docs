# Go Web系统中的错误处理(可能是最佳)实践

在讨论所有的错误处理实践之前首先要接受一个事实，那就是每调用一个函数/方法都要检查一下返回值中的`err`是否为空的操作是无法避免的，这可能是很多从Java转过来的开发者们除语法习惯以外最大的不适应。当然Go里也可以通过`panic`来模拟抛出"异常"然后中断执行流跳转到错误处理函数的效果，但这显然是政治错误的，原因主要有三：

- 万一panic没接好，整个程序都是要挂的；
- panic相对慢很多；
- 你都用go了，为啥还要把其他语言里的习惯强加到这里呢？入乡随俗吧。

对`if nil != err`达成一致后，剩下的问题就在于判断出有错误之后应该干什么了。

## 不要直接无脑返回error

这可能是最开始写go的朋友最容易犯的错误了。

```go
if nil != err {
        return err
}
```

这种写法最严重的问题在于出错以后你完全不知道这个错误最初是在哪里产生的，因为错误信息中没有任何可以用于定位的现场信息。为什么很多朋友没有在错误中添加现场信息的习惯呢？其实是被java/python这类语言惯坏了，反正出错以后会自动捕捉保存调用栈信息，谁还会再画蛇添足呢。

不过这种简单粗暴的处理方式也是有适用场景的，那就是当你只是想写一个类似于脚本这种代码短小、功能单一的小程序时，直接返回 error 无可厚非。

## 使用runtime.Caller()添加代码位置信息

当出现错误时最好的方法就是直接在日志或error对象中携带源代码的文件和行号信息，这样的功能在其他现代语言中一般都是语言级自带的，不过在go里我们需要自己实现。调用`runtime`包下的`Caller(int)`函数，可以获取到当前协程的调用栈、源代码文件名和行号信息。为了能把这些信息封装到`error`中，我们就必须自己实现一个创建`error`的函数，例如：

```go
func NewBizErrorf(format string, args ...interface{}) *BizError {
  _, srcName, line, _ := runtime.Caller(1) // (1)
    // [源文件名:行号]
    prefix := "[" + srcName + ":" + strconv.Itoa(line) + "] "
    msg := fmt.Sprintf(prefix + format, args...)

    return &BizError{msg:msg}
}
```

> (1): Caller(1)表示跳过调用栈的最顶层栈帧，如果传0，返回的源文件和行号就会永远是NewBizErrorf()函数所在的源文件/行号。此外在生产环境下需要判断第4个返回值是否为true

使用方式:

```go
if 0 == req.TemplateId || 0 == req.CommunityId || 0 == req.CateId || 0 == req.ExpireInHours {
        return NewBizErrorf("缺少模板|社区|分类id|过期时间")
}
```

打印出来以后是这样的：

```text
[/Users/whf/projects/private/moon/gomoon/biz/post-service.go:132] 缺少模板|社区|分类id|过期时间
```

到这里可能有些朋友会担心`Caller()`函数的性能问题，确实这样要比你直接在代码中手动把文件名/行号打印出来(笨)慢一个数量级，但可以这样想，只有在出错的情况下才会走这条路径，既然出错了，那首要目的应该是尽可能的记录错误信息方便排查，性能什么的就不重要了。当然，如果真的这块变成了瓶颈，那么可以不生成此类信息，手动在代码中传一些现场信息，比如函数名之类的，但这样就非常不灵活，而且后期维护很容易出错。

> 在 Java 中可以通过覆盖`Throwable`的`fillInStackTrace`方法，或把`RuntimeException`四参数的构造方法中的后一个bool参数传false来避免生成调用栈信息来提高性能。

## 使用错误嵌套保存原始错误

光有单条error信息还不够，我们还希望能够溯源，即找到造成此error的上一个error，类似于java中的`cause()`方法。这里最好用go1.13中标准库新加的错误嵌套机制了。比如，每当你遇到一个error时前先将这个error包一层，添加一些信息，然后再返回：

```go
return fmt.Errorf("xxx出错了！ => %w", err)
```

最后在打印error时，用`%+v`：

```go
Log.Errorf("get error: %+v",  err)
```

效果如下：

```text
get error:  invalid arg =>  缺少模板|社区|分类id|过期时间
```

可以看到，有调用栈的意思了，但还有其他问题，那就是我们并不知道每一层错误具体在源文件的什么位置。这里就需要用到前面说的方式了，我们不要直接使用`fmt.Errorf()`封装错误，而是自己写一个函数，添加调用者信息后再封装：

```go
func Errorf(format string, args ...interface{}) error {
    srcName, line := utils.GetSourceInfo(2)

    prefix := "[" + srcName + ":" + strconv.Itoa(line) + "] "
    return fmt.Errorf(prefix + format, args...)
}
```

这样打印出来以后就长这样：

```text
get error: [/Users/whf/projects/private/moon/gomoon/biz/post-service.go:51] invalid arg => [/Users/whf/projects/private/moon/gomoon/biz/post-service.go:132] 缺少模板|社区|分类id|过期时间
```

## 错误的分类

站在纯业务的角度来看错误可以分成两类，一类是可以让用户看到的友好的错误信息，另一类是提供给开发人员用来按图索骥还原现场排查原因的。

一般来讲，如果我们的web系统是直接跟用户交互的边缘服务，那么就需要有一层来统一识别业务代码返回的错误信息，比如如果发现是一个系统级错误，比如数据库连不上了，磁盘满了，网络断了等，就要给用户返回一个友好的提示而不能暴露出详细信息，比如"系统内部错误，请联系xxx"。如果是正常的业务错误，比如让你传int却传了字符串，或者要查找的目标不存在，这种就可以直接把错误原因返回给用户，而不能笼统的说系统错误。这里go1.13里新加的错误api就断能派上用场了。

我们可以自定义一种错误类型专门来表示前面说的业务错误：

```go
// 业务错误
type BizError struct {
    msg string
}
```

此类错误并不是通过下层错误引发的，即并不是判断了`if nil != err`为真后返回的，所以并不需要嵌套一个`cause`字段。当执行流跑到最顶层的web层后(再向后执行就要返回信息给客户端了)，这个`BizError`往往并不是最顶层的错误，可能已经被嵌套了很多次了。这时候就可以调用`errors.As()`函数来把这条错误链条里的`BizError`取出来，将`BizError`里的msg取出直接返回给客户端。例如：

```go
var bizErr *biz.BizError
    if errors.As(err, &bizErr) {
        return vo.NewNoDataCommonResponse(errcode.BIZ_ERR, bizErr.Error())
    }
```

如果`errors.As()`函数返回了false, 则说明这层错链中没有`BizError`，即这应该是个系统级的错误，则可以返回一个笼统的系统错误之类的提示语。

这里我们详细讨论了一下在生产环境中如何恰当的处理错误，同时又不丢失现场信息方便排查。但其实还有一个大的主题就是日志如何打印，比如是否每次打印日志都需要行号(未必)？这就是另一个主题了。
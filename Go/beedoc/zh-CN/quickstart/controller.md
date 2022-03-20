---
name: Controller 运行机制
sort: 3
---

# controller 逻辑

前面我们了解了如何把用户的请求分发到控制器，这小节我们就介绍大家如何来写控制器，首先我们还是从源码分析入手：

```
package controllers

import (
        "github.com/beego/beego/v2/server/web"
)

type MainController struct {
        web.Controller
}

func (this *MainController) Get() {
        this.Data["Website"] = "beego.me"
        this.Data["Email"] = "astaxie@gmail.com"
        this.TplName = "index.tpl"
}
```

上面的代码显示首先我们声明了一个控制器 `MainController`，这个控制器里面组合了 `web.Controller`，这就是 Go 的组合方式，也就是 `MainController` 自动拥有了所有 `web.Controller` 的方法。

而 `web.Controller` 拥有很多方法，其中包括 `Init`、`Prepare`、`Post`、`Get`、`Delete`、`Head` 等方法。我们可以通过重写的方式来实现这些方法，而我们上面的代码就是重写了 `Get` 方法。

我们先前介绍过 beego 是一个 RESTful 的框架，所以我们的请求默认是执行对应 `req.Method` 的方法。例如浏览器的是 `GET` 请求，那么默认就会执行 `MainController` 下的 `Get` 方法。这样我们上面的 Get 方法就会被执行到，这样就进入了我们的逻辑处理。（用户可以改变这个行为，通过注册自定义的函数名，更加详细的请参考[路由设置](../mvc/controller/router.md#%E8%87%AA%E5%AE%9A%E4%B9%89%E6%96%B9%E6%B3%95%E5%8F%8A-restful-%E8%A7%84%E5%88%99)）

里面的代码是需要执行的逻辑，这里只是简单的输出数据，我们可以通过各种方式获取数据，然后赋值到 `this.Data` 中，这是一个用来存储输出数据的 map，可以赋值任意类型的值，这里我们只是简单举例输出两个字符串。

最后一个就是需要去渲染的模板，`this.TplName` 就是需要渲染的模板，这里指定了 `index.tpl`，如果用户不设置该参数，那么默认会去到模板目录的 `Controller/<方法名>.tpl` 查找，例如上面的方法会去 `maincontroller/get.tpl` ***(文件、文件夹必须小写)***。

用户设置了模板之后系统会自动的调用 `Render` 函数（这个函数是在 `web.Controller` 中实现的），所以无需用户自己来调用渲染。

当然也可以不使用模版，直接用 `this.Ctx.WriteString` 输出字符串，如：

```
func (this *MainController) Get() {
        this.Ctx.WriteString("hello")
}
```

至此我们的控制器分析基本完成了，接下来让我们看看如何来编写 model。

[model 逻辑](./model.md)

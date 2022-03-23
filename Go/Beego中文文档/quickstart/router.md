---
name: 路由设置
sort: 2
---

# 项目路由设置

前面我们已经创建了 beego 项目，而且我们也看到它已经运行起来了，那么是如何运行起来的呢？让我们从入口文件先分析起来吧：

	package main

	import (
		_ "quickstart/routers"
		"github.com/astaxie/beego"
	)

	func main() {
		beego.Run()
	}

我们看到 main 函数是入口函数，但是我们知道 Go 的执行过程是如下图所示的方式：

![](../images/init.png)

这里我们就看到了我们引入了一个包 `_ "quickstart/routers"`,这个包只引入执行了里面的 init 函数，那么让我们看看这个里面做了什么事情：

```
package routers

import (
	"quickstart/controllers"
	"github.com/astaxie/beego"
)

func init() {
    beego.Router("/", &controllers.MainController{})
}

```

路由包里面我们看到执行了路由注册 `beego.Router`, 这个函数的功能是映射 URL 到 controller，第一个参数是 URL (用户请求的地址)，这里我们注册的是 `/`，也就是我们访问的不带任何参数的 URL，第二个参数是对应的 Controller，也就是我们即将把请求分发到那个控制器来执行相应的逻辑，我们可以执行类似的方式注册如下路由：

	beego.Router("/user", &controllers.UserController{})

这样用户就可以通过访问 `/user` 去执行 `UserController` 的逻辑。这就是我们所谓的路由，更多更复杂的路由规则请查询 [beego 的路由设置](../mvc/controller/router.md)

再回来看看 main 函数里面的 `beego.Run`， `beego.Run` 执行之后，我们看到的效果好像只是监听服务端口这个过程，但是它内部做了很多事情：

- 解析配置文件

	beego 会自动解析在 conf 目录下面的配置文件 `app.conf`，通过修改配置文件相关的属性，我们可以定义：开启的端口，是否开启 session，应用名称等信息。

- 执行用户的 hookfunc

	beego 会执行用户注册的 hookfunc，默认的已经存在了注册 mime，用户可以通过函数 `AddAPPStartHook` 注册自己的启动函数。

- 是否开启 session

	会根据上面配置文件的分析之后判断是否开启 session，如果开启的话就初始化全局的 session。

- 是否编译模板

	beego 会在启动的时候根据配置把 views 目录下的所有模板进行预编译，然后存在 map 里面，这样可以有效的提高模板运行的效率，无需进行多次编译。

- 是否开启文档功能

	根据 EnableDocs 配置判断是否开启内置的文档路由功能

- 是否启动管理模块

	beego 目前做了一个很酷的模块，应用内[监控模块](../advantage/monitor.md)，会在 8088 端口做一个内部监听，我们可以通过这个端口查询到 QPS、CPU、内存、GC、goroutine、thread 等统计信息。

- 监听服务端口

	这是最后一步也就是我们看到的访问 8080 看到的网页端口，内部其实调用了 `ListenAndServe`，充分利用了 goroutine 的优势

一旦 run 起来之后，我们的服务就监听在两个端口了，一个服务端口 8080 作为对外服务，另一个 8088 端口实行对内监控。

通过这个代码的分析我们了解了 beego 运行起来的过程，以及内部的一些机制。接下来让我们去剥离 Controller 如何来处理逻辑的。

[controller 运行机制](controller.md)


# Golang Web开发之Gin路由、控制器、分组路由

在本文中，我将为您介绍 Gin框架中路由以及它的控制器分别是什么概念给出一些阐述， 并引出它的分组路由知识。

## Gin路由与控制器

### 1. 怎么理解路由？

通常，在web开发领域中，我们将路由理解为——一个URL与函数之间的映射关系。也可以理解为，路由是一个由客户端所发起的一个http请求，按照这种理解，路由实际上是一个过程，再细化到后端，就是在接到这个请求时，如何找到对应后端业务逻辑函数的过程。

在Gin框架里，它的路由是基于httprouter来实现的。

### 2.Gin的路由规则

它的一条路由规则是通过三个关键元素组成的。它们是：

● HTTP请求方法

● URL路径（端点）

● 控制器函数

#### 2.1. HTTP请求方法

在我们日常的web开发中，使用的HTTP请求方法主要有下面4种:

● GET：获取资源；

● POST：创建或更新资源；

● PUT：更新资源；

● DELETE：删除资源；

这四种请求方法分别对应着数据库表的SELECT、INSERT、UPDATE、DELETE语句。利用这个特点，可以编写符合Restful风格的API。

#### 2.2. URL路径

一般地，有三种url路径写法。

- 静态url路径。即不带任何参数的url路径。例如：

```go
/user/233
```

- 带路径参数的url路径。这种url路径中带有参数，参数由冒号（:）跟着一个字符串定义。路径参数值可以是数值，也可以是字符串。例如：

```go
// 定义参数:id，可以匹配/user/233, /user/zhangsan
/user/:id
```

- 带星号（*）模糊匹配参数的url路径：这个*代表匹配任意路径的意思, 必须在*号后面指定一个参数名，后面可以通过这个参数获取*号匹配的内容。

```go
// 以/users/ 开头的所有路径都匹配，匹配：/users/1，/users/200, /users/1/20, /users/1/1/
/users/*path
```

#### 2.3. 控制器函数

所谓控制器函数，它主要负责执行实际处理的HTTP请求并对请求作出响应

定义一个控制器函数：func HandlerFunc(c *gin.Context)

其中，HandlerFunc 是控制器函数的名称，控制器函数接受一个上下文参数，()中这个上下文参数c，它的类型是*gin.Context。利用通过上下文参数，我们可以获取到HTTP请求参数以及HTTP请求作出响应。

```go
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    // 路由定义post请求, url路径为：/login, 绑定 login 控制器函数
    r.POST("/login", login)
    r.Run()
}
// 控制器函数
func login(c *gin.Context) {
    // 省略
}
```

### 3. Gin路由示例

上面说了这么多，现在，看几个路由定义的例子：

```go
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    r.POST("/users", saveUser)
    r.GET("/users/:id", getUser)
    r.PUT("/users/:id", updateUser)
    r.DELETE("/users/:id", deleteUser)
    r.Run()
}
// 控制器函数实现
func saveUser(c *gin.Context) {
}

func getUser(c *gin.Context) {
}

func updateUser(c *gin.Context) {
}

func deleteUser(c *gin.Context) {
}
```

在实际开发中，一般不会把路由定义、控制器函数写在一个go源文件里面，这会导致随着项目越来越大，代码会越来越多，不方便维护。可以按照自己公司的项目结构，规划自己的业务模块。

## Gin分组路由

### 1. 分组路由是？

顾名思义，路由分组，就是将路由分为了一组一组的，其实就是设置了同一类路由的url前缀。

比如，当我们在做API开发时，若要支持API的多个版本，我们可以通过分组路由来实现对API版本处理。

另外，利用分组这个特点，可以为一组路由API设置要鉴权，其他组API不设置鉴权，等等。

### 2. 示例展示

新建go文件，源代码如下：

```go
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {
    router := gin.Default()
    // v1路由组
    v1 := router.Group("/api/v1")
    {
        // 注册路由
        v1.POST("/register", registerEndpoint1)
        v1.POST("/login", loginEndpoint1)
    }
    // v2路由组
    v2 := router.Group("/api/v2")
    {
        // 注册路由
        v2.POST("/register", registerEndpoint2)
        v2.POST("/login", loginEndpoint2)
    }

    router.Run()
}

func registerEndpoint1(c *gin.Context) {
    // 省略
}

func loginEndpoint1(c *gin.Context) {
    // 省略
}

func registerEndpoint2(c *gin.Context) {
    // 省略
}

func loginEndpoint2(c *gin.Context) {
    // 省略
}
```

运行上述示例，注意观察终端中输出的日志：

```
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.
[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)
[GIN-debug] POST   /api/v1/register          --> main.registerEndpoint1 (3 handlers)
[GIN-debug] POST   /api/v1/login             --> main.loginEndpoint1 (3 handlers)
[GIN-debug] POST   /api/v2/register          --> main.registerEndpoint2 (3 handlers)
[GIN-debug] POST   /api/v2/login             --> main.loginEndpoint2 (3 handlers)
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
```

可以看到，日志中输出了我们注册了分组的路由。是不是很方便呢~~

## 总结

本文主要为您介绍了什么是路由、路由规则以及它的控制器， 并介绍了Gin的分组路由概念、场景与示例。
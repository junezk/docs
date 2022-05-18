# [go-zero之web框架](https://segmentfault.com/a/1190000038223666)



go-zero 是一个集成了各种工程实践的 web 和 rpc 框架，其中rest是web框架模块，基于Go语言原生的http包进行构建，是一个轻量的，高性能的，功能完整的，简单易用的web框架

### 服务创建

go-zero中创建http服务非常简单，官方推荐使用[goctl](https://link.segmentfault.com/?enc=htn5Smi640IxOcBfw7Hp6Q%3D%3D.zNMVa4cBqqENPzzmHn%2F6RjM1BtH0Ktt%2F8Kq2Jg2oY7Qn28AYluYDcHbA0JPjktkx7nOoyBxB81dDaHkHtyb6Nw%3D%3D)工具来生成。为了方便演示，这里通过手动创建服务，代码如下

```go
package main

import (
    "log"
    "net/http"

    "github.com/tal-tech/go-zero/core/logx"
    "github.com/tal-tech/go-zero/core/service"
    "github.com/tal-tech/go-zero/rest"
    "github.com/tal-tech/go-zero/rest/httpx"
)

func main() {
    srv, err := rest.NewServer(rest.RestConf{
        Port: 9090, // 侦听端口
        ServiceConf: service.ServiceConf{
            Log: logx.LogConf{Path: "./logs"}, // 日志路径
        },
    })
    if err != nil {
        log.Fatal(err)
    }
    defer srv.Stop()
    // 注册路由
    srv.AddRoutes([]rest.Route{ 
        {
            Method:  http.MethodGet,
            Path:    "/user/info",
            Handler: userInfo,
        },
    })
    
    srv.Start() // 启动服务
}

type User struct {
    Name  string `json:"name"`
    Addr  string `json:"addr"`
    Level int    `json:"level"`
}

func userInfo(w http.ResponseWriter, r *http.Request) {
    var req struct {
        UserId int64 `form:"user_id"` // 定义参数
    }
    if err := httpx.Parse(r, &amp;req); err != nil { // 解析参数
        httpx.Error(w, err)
        return
    }
    users := map[int64]*User{
        1: &amp;User{"go-zero", "shanghai", 1},
        2: &amp;User{"go-queue", "beijing", 2},
    }
    httpx.WriteJson(w, http.StatusOK, users[req.UserId]) // 返回结果
}
```

通过rest.NewServer创建服务，示例配置了端口号和日志路径，服务启动后侦听在9090端口，并在当前目录下创建logs目录同时创建各等级日志文件

然后通过srv.AddRoutes注册路由，每个路由需要定义该路由的方法、Path和Handler，其中Handler类型为http.HandlerFunc

最后通过srv.Start启动服务，启动服务后通过访问[http://localhost](https://link.segmentfault.com/?enc=y8%2F1x9O2z4VtUFjkx%2BLIaQ%3D%3D.%2BDIqRroctRzp3Wu6jhzlhTOfJG4H0A%2BpcMKXRSgqlDY%3D):9090/user/info?user_id=1可以看到返回结果

```json
{
    name: "go-zero",
    addr: "shanghai",
    level: 1
}
```

到此一个简单的http服务就创建完成了，可见使用rest创建http服务非常简单，主要分为三个步骤：创建Server、注册路由、启动服务

### JWT鉴权

鉴权几乎是每个应用必备的能力，鉴权的方式很多，而jwt是其中比较简单和可靠的一种方式，在rest框架中内置了jwt鉴权功能，jwt的原理流程如下图

<img src="go-zero之web框架.assets/up-cd9dc0dbd93e7be4b46a3e8cba1f3438ccd-16528941907015.png" alt="jwt" style="zoom:50%;">

rest框架中通过rest.WithJwt(secret)启用jwt鉴权，其中secret为服务器秘钥是不能泄露的，因为需要使用secret来算签名验证payload是否被篡改，如果secret泄露客户端就可以自行签发token，黑客就能肆意篡改token了。我们基于上面的例子进行改造来验证在rest中如何使用jwt鉴权

#### 获取jwt

第一步客户端需要先获取jwt，在登录接口中实现jwt生成逻辑

```go
srv.AddRoute(rest.Route{
        Method:  http.MethodPost,
        Path:    "/user/login",
        Handler: userLogin,
})
```

为了演示方便，userLogin的逻辑非常简单，主要是获取信息然后生成jwt，获取到的信息存入jwt payload中，然后返回jwt

```go
func userLogin(w http.ResponseWriter, r *http.Request) {
    var req struct {
        UserName string `json:"user_name"`
        UserId   int    `json:"user_id"`
    }
    if err := httpx.Parse(r, &amp;req); err != nil {
        httpx.Error(w, err)
        return
    }
    token, _ := genToken(accessSecret, map[string]interface{}{
        "user_id":   req.UserId,
        "user_name": req.UserName,
    }, accessExpire)

    httpx.WriteJson(w, http.StatusOK, struct {
        UserId   int    `json:"user_id"`
        UserName string `json:"user_name"`
        Token    string `json:"token"`
    }{
        UserId:   req.UserId,
        UserName: req.UserName,
        Token:    token,
    })
}
```

生成jwt的方法如下

```go
func genToken(secret string, payload map[string]interface{}, expire int64) (string, error) {
    now := time.Now().Unix()
    claims := make(jwt.MapClaims)
    claims["exp"] = now + expire
    claims["iat"] = now
    for k, v := range payload {
        claims[k] = v
    }
    token := jwt.New(jwt.SigningMethodHS256)
    token.Claims = claims
    return token.SignedString([]byte(secret))
}
```

启动服务后通过cURL访问

```shell
curl -X "POST" "http://localhost:9090/user/login" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "user_name": "gozero",
  "user_id": 666
}'
```

会得到如下返回结果

```json
{
  "user_id": 666,
  "user_name": "gozero",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MDYxMDgwNDcsImlhdCI6MTYwNTUwMzI0NywidXNlcl9pZCI6NjY2LCJ1c2VyX25hbWUiOiJnb3plcm8ifQ.hhMd5gc3F9xZwCUoiuFqAWH48xptqnNGph0AKVkTmqM"
}
```

#### 添加Header

通过rest.WithJwt(accessSecret)启用jwt鉴权

```go
srv.AddRoute(rest.Route{
        Method:  http.MethodGet,
        Path:    "/user/data",
        Handler: userData,
}, rest.WithJwt(accessSecret))
```

访问/user/data接口返回 401 Unauthorized 鉴权不通过，添加Authorization Header，即能正常访问

```shell
curl "http://localhost:9090/user/data?user_id=1" \
      -H 'Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MDYxMDgwNDcsImlhdCI6MTYwNTUwMzI0NywidXNlcl9pZCI6NjY2LCJ1c2VyX25hbWUiOiJnb3plcm8ifQ.hhMd5gc3F9xZwCUoiuFqAWH48xptqnNGph0AKVkTmqM'
```

#### 获取信息

一般会将用户的信息比如用户id或者用户名存入jwt的payload中，然后从jwt的payload中解析出我们预存的信息，即可知道本次请求时哪个用户发起的

```go
func userData(w http.ResponseWriter, r *http.Request) {
    var jwt struct {
        UserId   int    `ctx:"user_id"`
        UserName string `ctx:"user_name"`
    }
    err := contextx.For(r.Context(), &amp;jwt)
    if err != nil {
        httpx.Error(w, err)
    }
    httpx.WriteJson(w, http.StatusOK, struct {
        UserId   int    `json:"user_id"`
        UserName string `json:"user_name"`
    }{
        UserId:   jwt.UserId,
        UserName: jwt.UserName,
    })
}
```

#### 实现原理

jwt鉴权的实现在authhandler.go中，实现原理也比较简单，先根据secret解析jwt token，验证token是否有效，无效或者验证出错则返回401 Unauthorized

```go
func unauthorized(w http.ResponseWriter, r *http.Request, err error, callback UnauthorizedCallback) {
    writer := newGuardedResponseWriter(w)

    if err != nil {
        detailAuthLog(r, err.Error())
    } else {
        detailAuthLog(r, noDetailReason)
    }
    if callback != nil {
        callback(writer, r, err)
    }

    writer.WriteHeader(http.StatusUnauthorized)
}
```

验证通过后把payload中的信息存入http request的context中

```go
ctx := r.Context()
for k, v := range claims {
  switch k {
    case jwtAudience, jwtExpire, jwtId, jwtIssueAt, jwtIssuer, jwtNotBefore, jwtSubject:
    // ignore the standard claims
    default:
    ctx = context.WithValue(ctx, k, v)
  }
}

next.ServeHTTP(w, r.WithContext(ctx))
```

### 中间件

web框架中的中间件是实现业务和非业务功能解耦的一种方式，在web框架中我们可以通过中间件来实现诸如鉴权、限流、熔断等等功能，中间件的原理流程如下图

![img](go-zero之web框架.assets/bVcKxSR.png)

rest框架中内置了非常丰富的中间件，在rest/handler路径下，通过[alice](https://link.segmentfault.com/?enc=aSNLxXTGyDmnXrHJf0%2B2Cw%3D%3D.lzUBk6jYw2ZQ%2BPknJvKiXTjBq59v6mgj%2Bb5o6oYVwqbhmhMVVeXRkHGjO%2Bll%2BJWN)工具把所有中间件链接起来，当发起请求时会依次通过每一个中间件，当满足所有条件后最终请求才会到达真正的业务Handler执行业务逻辑，上面介绍的jwt鉴权就是通过authHandler来实现的。由于内置中间件比较多篇幅有限不能一一介绍，感兴趣的伙伴可以自行学习，这里我们介绍一下prometheus指标收集的中间件PromethousHandler，代码如下

```go
func PromethousHandler(path string) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            startTime := timex.Now() // 起始时间
            cw := &amp;security.WithCodeResponseWriter{Writer: w}
            defer func() {
        // 耗时
                metricServerReqDur.Observe(int64(timex.Since(startTime)/time.Millisecond), path)
        // code码
                metricServerReqCodeTotal.Inc(path, strconv.Itoa(cw.Code))
            }()
            
            next.ServeHTTP(cw, r)
        })
    }
}
```

在该中间件中，在请求开始时记录了起始时间，在请求结束后在defer中通过prometheus的Histogram和Counter数据类型分别记录了当前请求path的耗时和返回的code码，此时我们通过访问[http://127.0.0.1](https://link.segmentfault.com/?enc=VhtR23ZaEWUNtNvB%2BWHHrg%3D%3D.D3aTvu0ZP2u1n%2F6aRFmGLeFwvLRo6WswL97N%2FhI4VGQ%3D):9101/metrics即可查看相关的指标信息

![img](go-zero之web框架.assets/bVcKxST.png)

### 路由原理

rest框架中通过AddRoutes方法来注册路由，每一个Route有Method、Path和Handler三个属性，Handler类型为http.HandlerFunc，添加的路由会被换成featuredRoutes定义如下

```go
featuredRoutes struct {
        priority  bool // 是否优先级
        jwt       jwtSetting  // jwt配置
        signature signatureSetting // 验签配置
        routes    []Route  // 通过AddRoutes添加的路由
    }
```

featuredRoutes通过engine的AddRoutes添加到engine的routes属性中

```go
func (s *engine) AddRoutes(r featuredRoutes) {
    s.routes = append(s.routes, r)
}
```

调用Start方法启动服务后会调用engine的Start方法，然后会调用StartWithRouter方法，该方法内通过bindRoutes绑定路由

```go
func (s *engine) bindRoutes(router httpx.Router) error {
    metrics := s.createMetrics()

    for _, fr := range s.routes { 
        if err := s.bindFeaturedRoutes(router, fr, metrics); err != nil { // 绑定路由
            return err
        }
    }

    return nil
}
```

最终会调用patRouter的Handle方法进行绑定，patRouter实现了Router接口

```go
type Router interface {
    http.Handler
    Handle(method string, path string, handler http.Handler) error
    SetNotFoundHandler(handler http.Handler)
    SetNotAllowedHandler(handler http.Handler)
}
```

patRouter中每一种请求方法都对应一个树形结构，每个树节点有两个属性item为path对应的handler，而children为带路径参数和不带路径参数对应的树节点, 定义如下：

```go
node struct {
  item     interface{}
  children [2]map[string]*node
}

Tree struct {
  root *node
}
```

通过Tree的Add方法把不同path与对应的handler注册到该树上我们通过一个图来展示下该树的存储结构，比如我们定义路由如下

```go
{
  Method:  http.MethodGet,
  Path:    "/user",
  Handler: userHander,
},
{
  Method:  http.MethodGet,
  Path:    "/user/infos",
  Handler: infosHandler,
},
{
  Method:  http.MethodGet,
  Path:    "/user/info/:id",
  Handler: infoHandler,
},
```

路由存储的树形结构如下图

<img src="go-zero之web框架.assets/up-fc2d96766688c6ce4f652588786cd46021d.png" style="zoom:40%;">

当请求来的时候会调用patRouter的ServeHTTP方法，在该方法中通过tree.Search方法找到对应的handler进行执行，否则会执行notFound或者notAllow的逻辑

```go
func (pr *patRouter) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    reqPath := path.Clean(r.URL.Path)
    if tree, ok := pr.trees[r.Method]; ok {
        if result, ok := tree.Search(reqPath); ok { // 在树中搜索对应的handler
            if len(result.Params) &gt; 0 {
                r = context.WithPathVars(r, result.Params)
            }
            result.Item.(http.Handler).ServeHTTP(w, r)
            return
        }
    }

    allow, ok := pr.methodNotAllowed(r.Method, reqPath)
    if !ok {
        pr.handleNotFound(w, r)
        return
    }

    if pr.notAllowed != nil {
        pr.notAllowed.ServeHTTP(w, r)
    } else {
        w.Header().Set(allowHeader, allow)
        w.WriteHeader(http.StatusMethodNotAllowed)
    }
}
```

### 总结

本文从整体上介绍了rest，通过该篇文章能够基本了解rest的设计和主要功能，其中中间件部分是重点，里面集成了各种服务治理相关的功能，并且是自动集成的不需要我们做任何配置，其他功能比如参数自动效验等功能由于篇幅有限在这里就不做介绍了，感兴趣的朋友可以自行查看官方文档进行学习。go-zero中不光有http协议还提供了rpc协议和各种提高性能和开发效率的工具，是一款值得我们深入学习和研究的框架。
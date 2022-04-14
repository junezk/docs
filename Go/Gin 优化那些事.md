# Gin 优化那些事

本文介绍gin的一些知识点,如自定义Response，中间件等。初级的使用方式不介绍了，具体请查阅官方文档。

官方地址：`https://github.com/gin-gonic/gin`

以下介绍基于gin开发项目的一些常用模块。

## 自定义Response

每个公司都会自定义接口的数据结构。故我们需要基于`Json()`自定义一个更方便好用的response

```go
// Response 数据结构体
type Response struct {
	// StatusCode 业务状态码
	StatusCode int `json:"status_code"`
	// Message 提示信息
	Message    string      `json:"message"`
	// Data 数据，用interface{}的目的是可以用任意数据
	Data       interface{} `json:"data"`
	// Meta 源数据,存储如请求ID,分页等信息
	Meta       Meta        `json:"meta"`
	// Errors 错误提示，如 xx字段不能为空等
	Errors     []ErrorItem `json:"errors"`
}

// Meta 元数据
type Meta struct {
	RequestId    string    `json:"request_id"`
	// 还可以集成分页信息等
}

// ErrorItem 错误项
type ErrorItem struct {
	Key   string `json:"key"`
	Value string `json:"error"`
}

// New return response instance
func New() *Response {
	return &Response{
		StatusCode: 200,
		Message:    "",
		Data:       nil,
		Meta: Meta{
			RequestId: uuid.NewV4().String(),
		},
		Errors: []ErrorItem{},
	}
}
```

封装gin.Context以自定义一些方便的方法

```go
// Wrapper include context
type Wrapper struct {
	ctx *gin.Context
}

// WrapContext
func WrapContext(ctx *gin.Context) *Wrapper {
	return &Wrapper{ctx:ctx}
}

// Json 输出json,支持自定义response结构体
func (wrapper *Wrapper) Json(response *Response) {
	wrapper.ctx.JSON(200, response)
}

// Success 成功的输出
func (wrapper *Wrapper) Success( data interface{}) {
	response := New()
	response.Data = data
	wrapper.Json(response)
}

// Error 错误输出
func (wrapper *Wrapper) Error( statusCode int, message string) {
	response := New()
	response.StatusCode = statusCode
	response.Message = message
	wrapper.Json(response)
}
```

使用：

```go
package main

import (
	"github.com/gin-gonic/gin"
	uuid "github.com/satori/go.uuid"
)

func main() {
	router := gin.Default()
	router.GET("/", func(ctx *gin.Context) {
		WrapContext(ctx).Success("hello,world")
	})

	router.Run(":8088")
}
```

通过`go run main.go`运行后，浏览器访问`localhost:8088`

## 绑定 JSON 参数使用 jsoniter

Gin 框架中，处理 JSON 格式的参数绑定时，默认采用的标准包 encoding/json，然而标准包不能满足我们的一些要求，比如兼容字符串整型、PHP 空数组、时间格式等。

### 最简单的方式

开发 API 时，需要用到 ShouldBindJSON 绑定传入的参数到结构体：

```go
// github.com/gin-gonic/gin@v1.6.3/context.go:643
// ShouldBindJSON is a shortcut for c.ShouldBindWith(obj, binding.JSON).
func (c *Context) ShouldBindJSON(obj interface{}) error {
    return c.ShouldBindWith(obj, binding.JSON)
}
```

Gin 默认采用 `encoding/json` 包：

```go
// github.com/gin-gonic/gin@v1.6.3/internal/json/json.go
// +build !jsoniter
package json
import "encoding/json"
var (
    // Marshal is exported by gin/json package.
    Marshal = json.Marshal
    // Unmarshal is exported by gin/json package.
    Unmarshal = json.Unmarshal
    // MarshalIndent is exported by gin/json package.
    MarshalIndent = json.MarshalIndent
    // NewDecoder is exported by gin/json package.
    NewDecoder = json.NewDecoder
    // NewEncoder is exported by gin/json package.
    NewEncoder = json.NewEncoder
)
```

同时我们看到还支持了 `jsoniter`

```go
// github.com/gin-gonic/gin@v1.6.3/internal/json/jsoniter.go
// +build jsoniter
package json
import "github.com/json-iterator/go"
var (
    json = jsoniter.ConfigCompatibleWithStandardLibrary
    // Marshal is exported by gin/json package.
    Marshal = json.Marshal
    // Unmarshal is exported by gin/json package.
    Unmarshal = json.Unmarshal
    // MarshalIndent is exported by gin/json package.
    MarshalIndent = json.MarshalIndent
    // NewDecoder is exported by gin/json package.
    NewDecoder = json.NewDecoder
    // NewEncoder is exported by gin/json package.
    NewEncoder = json.NewEncoder
)
```

那我们怎么才能使用到 `jsoniter` 呢？源码中已经明确了编译 tag：

所以，我们只需在编译时带上这个 tag 就可以了，例如：

```shell
go build -tags=jsoniter main.go
// 或者
go run -tags=jsoniter main.go
```

### 自定义的方式

Gin 框架支持的 `jsoniter` 是默认配置 `jsoniter.ConfigCompatibleWithStandardLibrary`。当我们需要其他配置或添加一些自定义扩展（比如时间处理）时，就难受了。于是我们就要自己动手了~

翻开源码，我们能看到 `binding.JSON` 其实使用的是 `jsonBinding{}` 这个结构体：

```go
// github.com/gin-gonic/gin@v1.6.3/binding/binding.go:73
// These implement the Binding interface and can be used to bind the data
// present in the request to struct instances.
var (
    JSON          = jsonBinding{}
    // 其他省略了...
)
```

翻开 `jsonBinding` 源码看看：

```go
// github.com/gin-gonic/gin@v1.6.3/binding/json.go
type jsonBinding struct{}
func (jsonBinding) Name() string {
    return "json"
}
func (jsonBinding) Bind(req *http.Request, obj interface{}) error {
    if req == nil || req.Body == nil {
        return fmt.Errorf("invalid request")
    }
    return decodeJSON(req.Body, obj)
}
func (jsonBinding) BindBody(body []byte, obj interface{}) error {
    return decodeJSON(bytes.NewReader(body), obj)
}
```

发现实现了 `BindingBody` 这个接口：

```go
// github.com/gin-gonic/gin@v1.6.3/binding/binding.go:36
// Binding describes the interface which needs to be implemented for binding the
// data present in the request such as JSON request body, query parameters or
// the form POST.
type Binding interface {
    Name() string
    Bind(*http.Request, interface{}) error
}
// BindingBody adds BindBody method to Binding. BindBody is similar with Bind,
// but it reads the body from supplied bytes instead of req.Body.
type BindingBody interface {
    Binding
    BindBody([]byte, interface{}) error
}
```

那接下来就简单了，我们只要实现了这个接口即可，例如：

```go
package custom
import (
    "bytes"
    "fmt"
    "io"
    "net/http"
    jsoniter "github.com/json-iterator/go"
    "github.com/gin-gonic/gin/binding"
)
// BindingJSON 替换Gin默认的binding，支持更丰富JSON功能
var BindingJSON = jsonBinding{}
// 可以自定义jsoniter配置或者添加插件
var json = jsoniter.ConfigCompatibleWithStandardLibrary
type jsonBinding struct{}
func (jsonBinding) Name() string {
    return "json"
}
func (jsonBinding) Bind(req *http.Request, obj interface{}) error {
    if req == nil || req.Body == nil {
        return fmt.Errorf("invalid request")
    }
    return decodeJSON(req.Body, obj)
}
func (jsonBinding) BindBody(body []byte, obj interface{}) error {
    return decodeJSON(bytes.NewReader(body), obj)
}
func decodeJSON(r io.Reader, obj interface{}) error {
    decoder := json.NewDecoder(r)
    if binding.EnableDecoderUseNumber {
        decoder.UseNumber()
    }
    if binding.EnableDecoderDisallowUnknownFields {
        decoder.DisallowUnknownFields()
    }
    if err := decoder.Decode(obj); err != nil {
        return err
    }
    return validate(obj)
}
func validate(obj interface{}) error {
    if binding.Validator == nil {
        return nil
    }
    return binding.Validator.ValidateStruct(obj)
}
```

自定义 `jsonBinding` 已经写好了，可使用有 2 种方式：

```go
// binding.JSON 替换成自定义的
ctx.ShouldBindWith(ms, binding.JSON)
ctx.ShouldBindBodyWith(ms, binding.JSON)
```

上述自定义的方式，还可以用于其他包，不仅限于 `iterator`。从这个方面体现出了 Gin 框架良好的接口设计👍。

## 中间件

介绍一些常用的中间件，如跨域、Jwt校验、请求日志等。

### 备注

引入中间件比如在注册路由之前,谨记!

### 跨域中间件

```go
package middleware
import (
 "github.com/gin-gonic/gin"
)
// CORS 跨域中间件
func CORS(ctx *gin.Context) {
 method := ctx.Request.Method

 // set response header
 ctx.Header("Access-Control-Allow-Origin", ctx.Request.Header.Get("Origin"))
 ctx.Header("Access-Control-Allow-Credentials", "true")
 ctx.Header("Access-Control-Allow-Headers", "Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With")
 ctx.Header("Access-Control-Allow-Methods", "GET,POST,PUT,PATCH,DELETE,OPTIONS")

    // 默认过滤这两个请求,使用204(No Content)这个特殊的http status code
 if method == "OPTIONS" || method == "HEAD" { 
  ctx.AbortWithStatus(204)
  return
 }

 ctx.Next()
}
```

使用如下：

```go
func main() {
    router := gin.Default()
    router.Use(CORS)
    router.GET("/", func(ctx *gin.Context) {
     WrapContext(ctx).Success("hello,world")
    })

    router.Run(":8088")
}
```

### Jwt校验

```go
package main

import (
 "errors"
 "github.com/dgrijalva/jwt-go"
 "github.com/gin-gonic/gin"
 "strings"
 "time"
)

var (
 TokenNotExist       = errors.New("token not exist")
 TokenValidateFailed = errors.New("token validate failed")
 ClaimsKey = "uniqueClaimsKey"
 SignKey = "test"
)

// JwtAuth jwt
type JwtAuth struct {
 SignKey []byte
}

// ParseToken parse token
func (jwtAuth JwtAuth) ParseToken(token string) (jwt.Claims, error) {
 tokenClaims, err := jwt.Parse(token, func(token *jwt.Token) (interface{}, error) {
  return jwtAuth.SignKey, nil
 })

 if err != nil {
  return nil, err
 }

 if tokenClaims.Claims == nil || !tokenClaims.Valid {
  return nil, TokenValidateFailed
 }

 return tokenClaims.Claims, nil
}

// GenerateToken
func (jwtAuth JwtAuth) GenerateToken(tokenExpireTime int64 /* 过期时间 */, iss string /* key*/) (string, error) {
 now := time.Now().Unix()
 exp := now + tokenExpireTime
 claim := jwt.MapClaims{
  "iss": iss,
  "iat": now,
  "exp": exp,
 }
 token := jwt.NewWithClaims(jwt.SigningMethodHS256, claim)
 tokenStr, err := token.SignedString(jwtAuth.SignKey)
 return tokenStr, err
}

// JWT gin的jwt中间件
func JWT(ctx *gin.Context) {
 // 解析token
 if err := validateToken(ctx); err != nil {
  WrapContext(ctx).Error(401, err.Error())
  ctx.Abort()
  return
 }

 ctx.Next()
}

// validateToken 验证token
func validateToken(ctx *gin.Context) error {
 // 获取token
 tokenStr := ctx.GetHeader("Authorization")
 kv := strings.Split(tokenStr, " ")
 if len(kv) != 2 || kv[0] != "Bearer" {
  return TokenNotExist
 }

 jwtAuth := &JwtAuth{SignKey: []byte(SignKey)}
 claims, err := jwtAuth.ParseToken(kv[1])
 if err != nil {
  return err
 }

 // token存入context
 ctx.Set(ClaimsKey, claims)
 return nil
}
```

使用如下：

```go
func main()  {
 router := gin.Default()
 router.GET("/", func(ctx *gin.Context) {
  WrapContext(ctx).Success("hello,world")
 })

    // 指定user这组路由都需要校验jwt
 user := router.Group("/user").Use(JWT)
 {
  user.GET("/info", func(ctx *gin.Context) {
   claims, exist := ctx.Get(ClaimsKey)
   if !exist {
    WrapContext(ctx).Error(1001, "获取用户信息失败")
   }
   WrapContext(ctx).Success(claims)
  })
 }


 router.Run(":8088")
}
```

请求测试：

```shell
curl  "localhost:8088/user/info"
// 输出:
// {"status_code":401,"message":"token not exist","data":null,"meta":{"request_id":"e69361cf-1fd4-42e4-8af8-d18fac1e70fb"},"errors":[]}

// 通过GenerateToken()生成一个token
curl -H "Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1ODU4MjQ2NzgsImlhdCI6MTU4NTgyMTA3OCwiaXNzIjoiYWEifQ.Eyo8KptVUgGfnRG8zsjDilAJOBmaXMtjqJxw__a32HY"  localhost:8088/user/info
// 输出：
{"status_code":200,"message":"","data":{"exp":1585824678,"iat":1585821078,"iss":"aa"},"meta":{"request_id":"464743de-1033-4656-96f8-36c1529f13e0"},"errors":[]}
```

### 请求日志

记录每个请求的重要信息

```go
import (
 "bytes"
 "fmt"
 "github.com/gin-gonic/gin"
 "io/ioutil"
 "log"
 "net/http"
 "time"
)

// bodyLogWriter 定义一个存储响应内容的结构体
type bodyLogWriter struct {
 gin.ResponseWriter
 body *bytes.Buffer
}

// Write 读取响应数据
func (w bodyLogWriter) Write(b []byte) (int, error) {
 w.body.Write(b)
 return w.ResponseWriter.Write(b)
}

// RequestLog gin的请求日志中间件
func RequestLog(c *gin.Context) {
 // 记录请求开始时间
 t := time.Now()
 blw := &bodyLogWriter{body: bytes.NewBufferString(""), ResponseWriter: c.Writer}
 // 必须!
 c.Writer = blw

 // 获取请求信息
 requestBody := getRequestBody(c)

 c.Next()

 // 记录请求所用时间
 latency := time.Since(t)

 // 获取响应内容
 responseBody := blw.body.String()

 logContext := make(map[string]interface{})
 // 日志格式
 logContext["request_uri"] = c.Request.RequestURI
 logContext["request_method"] = c.Request.Method
 logContext["refer_service_name"] = c.Request.Referer()
 logContext["refer_request_host"] = c.ClientIP()
 logContext["request_body"] = requestBody
 logContext["request_time"] = t.String()
 logContext["response_body"] = responseBody
 logContext["time_used"] = fmt.Sprintf("%v", latency)
 logContext["header"] = c.Request.Header

 log.Println(logContext)
}

// getRequestBody 获取请求参数
func getRequestBody(c *gin.Context) interface{} {
 switch c.Request.Method {
 case http.MethodGet:
  return c.Request.URL.Query()

 case http.MethodPost:
  fallthrough
 case http.MethodPut:
  fallthrough
 case http.MethodPatch:
  var bodyBytes []byte // 我们需要的body内容
        // 可以用buffer代替ioutil.ReadAll提高性能
  bodyBytes, err := ioutil.ReadAll(c.Request.Body)
  if err != nil {
   return nil
        }
        // 将数据还回去
  c.Request.Body = ioutil.NopCloser(bytes.NewBuffer(bodyBytes))

  return string(bodyBytes)

 }

 return nil
}
```

使用

```go
router.Use(ReqeustLog)
```

今天就到这儿吧，还有一些比如全局ID中间件，后面来写。

原文： https://hongker.github.io/2020/04/01/golang-gin/

## 基于context扩展gin框架

大家用过gin的话，应该了解gin框架是通过一个叫做*gin.Context的对象传递请求数据的，一个简单的例子如下：

```go
r := gin.Default()
r.GET("/ping", func(c *gin.Context) {
    c.JSON(200, gin.H{
    	"message": "pong",
    })
})
r.Run() // 监听并在 0.0.0.0:8080 上启动服务
```

使用gin.Context可以获取请求参数，返回应答，这通常没什么问题。

但是如果希望把gin扩展成一个完备的MVC框架，作为公司级的统一开发标准，那么就需要做更顶层的统一设计。

此时，我们只是将Gin作为路由层的一个非常好用的组件，然而为了达成MVC框架的整体设计目标，我们应该对围绕Gin外围做一些适配和扩展，从而更好的服务框架整体设计。

### 典型需求

#### 超时控制

作为一个MVC框架，我希望能对每一个请求进行全生命期的超时控制。

在golang中，我们只有一种选择就是使用context.Context参数，在代码全流程中向下透传到各个类库/第三方类库。

只要我们在请求的入口生成withTimeout的context，那么对于那些支持并处理了context传参的类库来说，就可以实现请求生命期的超时中断，避免请求因为个别调用点阻塞而导致协程大量打满。

#### 调用链埋点

在微服务化时代，我们非常依赖于调用链trace能力。

在golang中，一个请求到来后经过一系列数据库/网络调用直到最终得到应答的过程，全部需要主动进行trace埋点。

作为一个公司级的MVC框架，会对诸如rpc、mysql、redis等各个类库进行侵入式埋点的实现，而为了将同一个请求的所有调用过程顺序记录下来，就需要将同一个trace上下文对象在各个类库之间持续透传，因此还是只能依靠每个请求的context.Context上下文对象来携带trace对象。

从上面2个典型需求来看，一个MVC框架最重要的就是每个请求要有一个完全可控的context.Context对象，这需要我们在框架层面生成并从controller层首次传入，并由开发者手动向下逐层透传。同时，例如rpc、mysql、redis等类库均需要接受context传参，以便从context中获取trace对象完成自定义埋点。

当然，context.Context接口支持操作多个key-value数据，因此例如trace对象只是其中的一种场景，我们还可以注入更多的框架层信息到context.Context中。

### gin.Context可以存储上下文数据

gin.Context实现了context.Context接口的方法，并且给context.Context的Value()方法提供了一个map的实现：

```go
type Context struct {
	writermem responseWriter
	Request   *http.Request
	Writer    ResponseWriter
 
	Params   Params
	handlers HandlersChain
	index    int8
 
	engine *Engine
 
	// Keys is a key/value pair exclusively for the context of each request.
	Keys map[string]interface{}
 
	// Errors is a list of errors attached to all the handlers/middlewares who used this context.
	Errors errorMsgs
 
	// Accepted defines a list of manually accepted formats for content negotiation.
	Accepted []string
}
```

它实现context.Context的方法，其中Value()特别实用，用于从Keys字典查询k-v：

```go
// Deadline returns the time when work done on behalf of this context
// should be canceled. Deadline returns ok==false when no deadline is
// set. Successive calls to Deadline return the same results.
func (c *Context) Deadline() (deadline time.Time, ok bool) {
	return
}
 
// Done returns a channel that's closed when work done on behalf of this
// context should be canceled. Done may return nil if this context can
// never be canceled. Successive calls to Done return the same value.
func (c *Context) Done() <-chan struct{} {
	return nil
}
 
// Err returns a non-nil error value after Done is closed,
// successive calls to Err return the same error.
// If Done is not yet closed, Err returns nil.
// If Done is closed, Err returns a non-nil error explaining why:
// Canceled if the context was canceled
// or DeadlineExceeded if the context's deadline passed.
func (c *Context) Err() error {
	return nil
}
 
// Value returns the value associated with this context for key, or nil
// if no value is associated with key. Successive calls to Value with
// the same key returns the same result.
func (c *Context) Value(key interface{}) interface{} {
	if key == 0 {
		return c.Request
	}
	if keyAsString, ok := key.(string); ok {
		val, _ := c.Get(keyAsString)
		return val
	}
	return nil
}
```

当然，gin.Context提供了一个Set方法，它不是context.Context接口定义，仅能通过gin.Context.Set调用：

```go
// Set is used to store a new key/value pair exclusively for this context.
// It also lazy initializes  c.Keys if it was not used previously.
func (c *Context) Set(key string, value interface{}) {
	if c.Keys == nil {
		c.Keys = make(map[string]interface{})
	}
	c.Keys[key] = value
}
```

我们可以通过gin.Context.Set保存针对当前请求的trace上下文对象，然后将gin.Context作为context.Context向下透传，那么例如mysql、redis等类库内部就可以通过context.Context.Value()取到trace对象，完成埋点：

```go
return func (c *gin.Context) {
    // 可以在gin.Context中设置key-value
    c.Set("trace", "假设这是一个调用链追踪sdk") 
    dbQuery(c, "select * from xxx")
```

在mysql类库内可以这样取出trace对象使用：

```go
// 模拟一个MYSQL查询
func dbQuery(ctx context.Context, sql string) {
	// 模拟调用链埋点
	trace := ctx.Value("trace").(string)
```

### context.WithTimeout附加超时能力

为了完整控制请求的超时，需要基于现有gin.Context附加timeout能力：

```go
return func (c *gin.Context) {
    // 可以在gin.Context中设置key-value
    c.Set("trace", "假设这是一个调用链追踪sdk")

    // 请求超时控制
    timeoutCtx, _ := context.WithTimeout(c, 5 * time.Second)
```

WithTimeout传入gin.Context作为parent，返回1个新的context.Context作为child。

当timeoutCtx超时后，timeoutCtx以及其下的孩子context的Done()方法都会返回，当前代码中我们还没有给timeoutCtx下面挂载任何子context。

现在timeoutCtx已经蜕化为context.Context接口了，继续向下透传就只能访问到context.Context接口中的方法（例如：Done方法），而不能再调用到gin.Context中的特有方法了。

WithTimeout返回的context继承了gin.Context的接口实现，我们看WithTimeout代码会发现它使用了一种很罕见的语法（结构体继承接口）：

```go
// A cancelCtx can be canceled. When canceled, it also cancels any children
// that implement canceler.
type cancelCtx struct {
	Context
	mu       sync.Mutex            // protects following fields
	done     chan struct{}         // created lazily, closed by first cancel call
	children map[canceler]struct{} // set to nil by the first cancel call
	err      error                 // set to non-nil by the first cancel call
}
 
type timerCtx struct {
	cancelCtx
	timer *time.Timer // Under cancelCtx.mu.
	deadline time.Time
}
 
// newCancelCtx returns an initialized cancelCtx.
func newCancelCtx(parent Context) cancelCtx {
	return cancelCtx{Context: parent}
}
```

最终WithTimeout返回的是timerCtx，其继承了cancelCtx结构体，**而cancelCtx结构体继承了Context接口，结构体继承接口的语法是非常罕见的。**

通过观察newCancelCtx函数会发现，这里parent就是gin.Context，它被赋值到结构体里继承的Context接口，这样cancelCtx就继承了gin.Context的接口实现。

而如果我们仔细再去观察，会发现timerCtx和cancelCtx两个结构体都没有实现Context的Value()方法，而通过将gin.Context赋值给结构体中继承的Context接口这样一个骚操作，timerCtx就继承了gin.Context的Value()方法，而先前我们看过gin.Context的Value方法是基于map维护k-v的。

### 封装框架层Context

经过上述操作后，timeoutCtx已经具备来自gin.Context的Value方法（可以拿到trace对象），也具备了timeout的超时能力。

但是我们也提到，timeoutCtx已经蜕化为context.Context接口，我们没法再调用到gin.Context的特有方法，如果框架层把这样一个蜕化的context.Context传给Controller，那么用户无法享受到gin.Context中的各种方法。

因此，我需要自定义一个Context，作为timerCtx的子context，并且在这个Context中保存一份gin.Context的引用，同时也不丧失此前timerCtx继承自gin.Context的能力（我们需要它的Value()实现）。

这就需要模仿WithTimeout，自定义一个框架Context结构体并且继承context.Context，并将timerCtx赋予接口：

```go
type YuerContext struct {
	context.Context
	Gin *gin.Context
}
```

完整的生成流程如下：

```go
return func (c *gin.Context) {
    // 可以在gin.Context中设置key-value
    c.Set("trace", "假设这是一个调用链追踪sdk")

    // 全局超时控制
    timeoutCtx, _ := context.WithTimeout(c, 5 * time.Second)
    // 框架自定义上下文
    yuerCtx := YuerContext{Context: timeoutCtx, Gin: c}
```

yuerCtx就是一个继承了gin.Context的Value方法以及timeoutCtx的Done方法的自定义Context了。

我把gin.Context放在了结构体的一个字段里，将yuerContext保持类型传给controller层，用户就可以很方便的得到gin，并且继续向下按context.Context类型透传也仍旧可以通过Value()方法拿到trace对象。

### 包装Controller层

为了让开发者可以使用YuerContext而不是直接使用gin.Context，我提供了一种方法来包装原有的HandleFunc并返回一个用于gin注册路由的HandleFunc：

```go
type YuerHandleFunc func (c *YuerContext)
 
func WithYuerContext(yuerHandle YuerHandleFunc) gin.HandlerFunc {
	return func (c *gin.Context) {
		// 可以在gin.Context中设置key-value
		c.Set("trace", "假设这是一个调用链追踪sdk")
 
		// 全局超时控制
		timeoutCtx, _ := context.WithTimeout(c, 5 * time.Second)
		// ZDM上下文
		yuerCtx := YuerContext{Context: timeoutCtx, Gin: c}
 
		// 回调接口
		yuerHandle(&yuerCtx)
	}
}
```

现在注册路由稍作改变：

```go
r := gin.New()
 
r.GET("/test", WithYuerContext(func(c *YuerContext) {
	// 业务层处理
     dbQuery(c, "select * from xxx")
     // 调用gin应答
     c.Gin.String(200, "请求完成")
 }))

r.Run()
```

**这样就以很小的代码修改代价，实现了自定义context参数，并且保障了框架层的扩展性和操控性。**

WithYuerContext不是必须的，原生gin的HandleFunc仍旧可以直接注册路由，只是没有办法享受到YuerContext的特有能力。

完整代码见：https://github.com/owenliang/go-advanced/blob/master/context-demo/main.go。
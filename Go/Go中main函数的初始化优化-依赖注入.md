# Go中main函数的初始化优化-依赖注入

本文介绍在golang中如何通过依赖注入(Dependency Inject，简称DI)管理全局服务。

## 什么是DI

> 把有依赖关系的类放到容器中，解析出这些类的实例，就是依赖注入。

## DI的作用

### 反面例子

现在我们有一个http应用，先来看下常规开发的main.go:

```go
func main() {
 // 生成config实例
 config := NewConfig()
 // 连接数据库
 db, err := ConnectDatabase(config)
 // 判断是否有错误
 if err != nil {
  panic(err)
 }
 // 生成repository实例，用于获取person数据，参数是db
 personRepository := repo.NewPersonRepository(db)
 // 生成service实例，用于调用repository的方法
 personService := service.NewPersonService(config, personRepository)
 // 生成http服务实例
 server := NewServer(config, personService)
 // 启动http服务
 server.Run()
}
// Server
type Server struct {
 config        *config.Config
 personService *service.PersonService
}
// Handler
func (s *Server) Handler() http.Handler {
 mux := http.NewServeMux()

 mux.HandleFunc("/people", s.people)

 return mux
}
// Run
func (s *Server) Run() {
 httpServer := &http.Server{
  Addr:    ":" + s.config.Port,
  Handler: s.Handler(),
 }

 httpServer.ListenAndServe()
}
// people
func (s *Server) people(w http.ResponseWriter, r *http.Request) {
 people := s.personService.FindAll()
 bytes, _ := json.Marshal(people)

 w.Header().Set("Content-Type", "application/json")
 w.WriteHeader(http.StatusOK)
 w.Write(bytes)
}
// NewServer
func NewServer(config *config.Config, service *service.PersonService) *Server {
 return &Server{
  config:        config,
  personService: service,
 }
}

// 其他new方法
func NewConfig() *Config {
    // ...
}
func ConnectDatabase(config *config.Config) (*sql.DB, error) {
    // ... 
}
func NewPersonRepository(database *sql.DB) *PersonRepository {
    // ... 
}
func NewPersonService(config *config.Config, repository *repo.PersonRepository) *PersonService {
    // ...
}
```

直接看`main()`,你会发现包含清晰的初始化流程。

但是仔细想想，随着业务的扩展，我们如果把所有实例都在main函数里生成，main函数将变得越来越臃肿。

而且这些基础服务的实例，如果在其他包里需要引入，你就得给每个需要用到服务的地方，通过参数的方式传递。类似`service.NewPersonService(config, personRepository)`方法，将`config`和`personRepository`传递到service包。

### 问题

- 如何让main函数变得优雅?
- 如何管理全局服务实例?
- 如何减少重复实例化对象时传递如`config`这样的基础实例?

## 安装

我使用的是uber的dig包

```shell
go get github.com/uber-go/dig
```

## 优化main函数

```go
// 构建一个DI容器
func BuildContainer() *dig.Container {
  container := dig.New()
  // 注入config的实例化方法
  container.Provide(NewConfig)
  // 注入database的实例化方法
  container.Provide(ConnectDatabase)
  // 注入repository的实例化方法
  container.Provide(repo.NewPersonRepository)
  // 注入service的实例化方法
  container.Provide(service.NewPersonService)
  // 注入server
  container.Provide(NewServer)

  return container
}

func main() {
  container := BuildContainer()
  
  err := container.Invoke(func(server *Server) {
    server.Run()
  })

  if err != nil {
    panic(err)
  }
}
```

这样的main函数不需要包含任何基础实例的初始化和参数传递的过程，可以称之：Perfect!

下面是对main函数里基础服务注入的流程说明：

- BuildContainer，只将各个基础服务的实例化方法注入到容器里，还没有调用这些方法来实例化基础服务
- container.Invoke,这里将会从容器里寻找server实例，来运行`server.Run()`。如果实例不存在，则调用其实例化的方法，也就是`NewServer`
- 因为`NewServer(config *config.Config, service *service.PersonService) *Server`依赖于`config.Config`和`service.PersonService`，故触发`NewConfig`和`NewPersonService`方法。
- `NewConfig`不依赖于任何实例，故可以成功返回`config.Config`实例。
- `NewPersonService(config *config.Config, repository *repo.PersonRepository) *PersonService`依赖`config.Config`和`repo.PersonRepository`,继而触发`repo.NewPersonRepository`去实例化`repo.PersonRepository`
- `repo.NewPersonRepository`方法依赖于`db`,故触发`ConnectDatabase`方法，用来连接数据库，实例化`db`实例
- 最后递归倒推回去，完成所有实例的初始化与注入，调用`server.Run()`方法启动http服务。

注意，有依赖的初始化方法，需要放在前置依赖注入之后，比如`container.Provide(ConnectDatabase)`就放在`container.Provide(NewConfig)`之后。如果找不到初始化需要的依赖对象，在Invoke时就会报错。

## 踩坑

之前我通过下面的方式去获取容器里的基础实例：

```go
package app
// Config 配置文件
func Config() (conf *config.Config) {
 _ = container.Invoke(func(c *config.Config) {
  conf = c
 })
 return
}

// 其他package
fmt.Println(app.Config().GetString("someKey"))
```

这样去获取基础实例是不正确的用法，因为`dig`底层是通过一个`map`来管理这些实例的，我们都知道`map`不是线程安全的，在频繁调用时`偶尔`会出现以下错误：

```shell
concurrent map writes
```

开发者回答如下：

```
Hey, dig is not intended to be invoked from your system's hot path. We expect
it to be invoked at most once during startup, and definitely not concurrently.
To discourage usage on the hot path, we have kept the APIs thread-unsafe.
```

参考官方git里的一个issue,里面的代码能重现该异常: **Invoke not concurrency safe**[1]

所以，我们在使用`dig`注入的时候，将如`repo.NewPersonRepository`这样依赖`Config`和`DB`的实例函数，在main函数里通过`container.Provide`注入进去，这样仅调用一次，保证线程安全。

### 参考资料

[1]Invoke not concurrency safe: *https://github.com/uber-go/dig/issues/241*
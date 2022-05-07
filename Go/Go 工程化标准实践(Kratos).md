# [Go 工程化标准实践](https://www.yipwinghong.com/2021/08/04/Go_engineering-standard/#)(**Kratos**)

本文参考 Go 微服务框架 [go-kratos/kratos](https://github.com/go-kratos/kratos) 的项目结构及相关最佳实践，更多技术分享可关注公众号 **Kratos 开源社区**。

> Kratos 一套轻量级 Go 微服务框架，包含大量微服务相关功能及工具。

## 项目结构

### 标准项目结构

#### /cmd

```
|-- cmd
    |-- demo
        |-- demo
        +-- main.go
    +-- demo1
        |-- demo1
        +-- main.go
```

项目的主干，每个应用程序目录名与可执行文件的名称匹配。该目录不应放置太多代码。

#### /internal

```
|-- internal
    +-- demo
        |-- biz
        |-- service
        +-- data
```

私有应用程序和库代码。该目录由 Go 编译器强制执行（更多细节请参阅 Go 1.4 release notes），在项目树的任何级别上都可以有多个 /internal 目录。

可在 /internal 包中添加额外结构，以分隔共享和非共享的内部代码。对于较小的项目而言不是必需，但最好有可视化线索显示预期的包的用途。

实际应用程序代码可放在 /internal/app 目录下（比如 /internal/app/myapp），应用程序共享代码可放在 /internal/pkg 目录下（比如 /internal/pkg/myprivlib）。

相关服务（比如账号服务内部有 rpc、job、admin 等）整合一起后需要区分 app。单一服务则可以去掉 /internal/myapp。

#### /pkg

```
|-- pkg
    |-- memcache
    +-- redis
|-- conf
    |-- dsn 
    |-- env
    |-- flagvar
    +-- paladin
.
|-- docs
|-- example
|-- misc
|-- pkg
|-- third_party
|-- tool
```

外部应用程序可以使用的库代码。可以显式地表示该目录代码对于其他人而言是安全可用的。

/pkg 目录内可参考 Go 标准库的组织方式，按照功能分类。/internal/pkg 一般用于项目内的跨应用公共共享代码，但其作用域仅在单个项目工程内。

> pkg 和 internal 目录的相关描述可以参考 [I’ll take pkg over internal](https://travisjeffery.com/b/2019/11/i-ll-take-pkg-over-internal/)。

当根目录包含大量非 Go 组件和目录时，这也是一种将 Go 代码分组到一个位置的方法，使得运行各种 Go 工具更容易组织。

### 工具包项目结构

```
|-- cache
    |-- memcache
    |   +-- test
    +-- redis
        +-- test
|-- conf
    |-- dsn 
    |-- env
    |-- flagvar
    +-- paladin
        +-- apollo
            +-- internal
                +-- mockserver
|-- container
    |-- group
    |-- pool
    +-- queue
        +-- apm
|-- database
    |-- hbase
    |-- sql
    +-- tidb
|-- ecode
    +-- types
|-- log
    +-- internal
        |-- core
        +-- filewriter
```

应当为不同的微服务建立统一的 kit 工具包项目（基础库/框架）和 app 项目。

基础库 kit 为独立项目，公司级建议只有一个。由于按照功能目录来拆分会带来不少的管理工作，建议合并整合。

其具备以下特点：

- 统一
- 标准库方式布局
- 高度抽象
- 支持插件

### 服务应用项目结构

```
.
|-- README.md
|-- api
|-- cmd
|-- configs
|-- go.mod
|-- go.sum
|-- internal
+-- test
```

#### /api

API 协议定义目录，比如 protobuf 文件和生成的 go 文件。

通常把 API 文档直接在 proto 文件中描述。

#### /configs

配置文件模板或默认配置。

#### /test

外部测试应用程序和测试数据。可随时根据需求构造 /test 目录。

对于较大的项目数据子目录是很有意义的。比如可使用 /test/data 或 /test/testdata（如果需要忽略目录中的内容）。

Go 会忽略以“.”或“_”开头的目录或文件，因此在命名测试数据目录方面有更大灵活性。

#### GitLab Project

```
|-- app
    |-- replay
    |--..
    +-- member
|-- pkg
    |-- database
    |-- ..
    +-- log
+-- ...
```

一个 GitLab project 中可以放置多个微服务 app（类似 monorepo），也可以按照 GitLab 的 group 里建立多个 project，每个 project 对应一个 app。

#### 微服务结构

```
|-- cmd                     负责程序的：启动、关闭、配置初始化等。
    |-- myapp1-admin        面向运营侧的服务，通常数据权限更高，隔离实现更好的代码级别安全。
    |-- myapp1-interface    对外的 BFF 服务，接受来自用户的请求（HTTP、gRPC）。
    |-- myapp1-job          流式任务服务，上游一般依赖 message broker。
    |-- myapp1-service      对内的微服务，仅接受来自内部其他服务或网关的请求（gRPC）。
    +-- myapp1-task         定时任务服务，类似 cronjob，部署到 task 托管平台中。
```

以下这种目录结构风格：

```
|-- service
    |-- api             API 定义（protobuf 等）以及对应生成的 client 代码，基于 pb 生成的 swagger.json。
    |-- cmd
    |-- configs         服务配置文件，比如 database.yaml、redis.yaml、application.yaml。
    |-- internal        避免有同业务下被跨目录引用了内部的 model、dao 等内部 struct。
        |-- model       对应“存储层”的结构体，是对存储的一一映射。
        |-- dao         数据读写层，统一处理数据库和缓存（cache miss 等问题）。
        |-- service     组合各种数据访问来构建业务逻辑，包括 api 中生成的接口实现。
        |-- server      依赖 proto 定义的服务作为入参，提供快捷的启动服务全局方法。
|-- ...
```

app 目录下有 api、cmd、configs、internal 目录。一般还会放置 README、CHANGELOG、OWNERS。

项目的依赖路径为：model -> dao -> service -> api，model struct 串联各个层，直到 api 做 DTO 对象转换。

另一种结构风格是将 DDD 设计思想和工程结构做了简化，映射到 api、service、biz、data 各层。

```
.
|-- CHANGELOG
|-- OWNERS
|-- README
|-- api
|-- cmd
    |-- myapp1-admin
    |-- myapp1-interface
    |-- myapp1-job
    |-- myapp1-service
    +-- myapp1-task
|-- configs
|-- go.mod
|-- internal        避免有同业务下被跨目录引用了内部的 model、dao 等内部 struct。
    |-- biz         业务逻辑组装层，类似 DDD domain（repo 接口再次定义，依赖倒置）。
    |-- data        业务数据访问，包含 cache、db 等封装，实现 biz 的 repo 接口。
    |-- pkg
    +-- service     实现了 api 定义的服务层，类似 DDD application
    处理 DTO 到 biz 领域实体的转换（DTO->DO），同时协同各类 biz 交互，不处理复杂逻辑。
```

[![image-20210801185711490](Go 工程化标准实践(Kratos).assets/image-20210801185711490.png)](https://ywh-oss.oss-cn-shenzhen.aliyuncs.com/Go_engineering-standard.assets/image-20210801185711490.png)

[![image-20210801185937316](Go 工程化标准实践(Kratos).assets/image-20210801185937316.png)](https://ywh-oss.oss-cn-shenzhen.aliyuncs.com/Go_engineering-standard.assets/image-20210801185937316.png)

#### 架构与数据模型

**松散分层架构（Relaxed Layered System）**：层间关系不太严格，每层都可能使用它下面所有层的服务（而不仅是下一层）。每层都可能是半透明的，意味着有些服务只对上一层可见，而有些服务对上面的所有层都可见。

```
[       api         ]
    |     |       | 
    | [ service ] | 
    |    |        |
  [     biz     ] | 
         |        | 
      [    data     ]
```

**继承分层架构（Layering Through Inheritance）**：高层继承并实现低层接口。需要调整各层顺序，将基础设施层移动到最高层。这依然是单向依赖，意味着领域层、应用层、表现层将不能依赖基础设施层，而基础设施层可以依赖它们。

```
  [     data     ]
  |      |       |
  |   [ api ]    |
  |      |       |
  [   service   ] 
  |      |      |
  [     biz     ]
```

数据模型：

- **失血模型**：仅包含数据定义和 getter/setter 方法，业务逻辑和应用逻辑都放到服务层中。在 Java 中称为 POJO，在 .NET 中称为 POCO。
- **贫血模型**：包含一些业务逻辑，但不包含依赖持久层的业务逻辑（会放在服务层中），领域对象不依赖于持久层。
- **充血模型**：包含所有业务逻辑，领域层依赖于持久层，简单表示就是：UI 层 -> 服务层 -> 领域层 <-> 持久层。
- **胀血模型**：和业务逻辑不想关的其他应用逻辑（如授权、事务等）放到领域模型中（反而是另外一种失血模型，服务层缺失、由领域层代劳）。

### 生命周期

考虑服务应用对象初始化和生命周期管理，所有 HTTP/gRPC 依赖的前置资源初始化（包括 data、biz、service），之后再启动监听服务。

资源初始化和关闭步骤繁琐，比较容易出错。可利用依赖注入的思路，使用 [google/wire](https://github.com/google/wire) 管理资源依赖注入，方便测试和实现单次初始化与复用。

```
svr := http.NewServer()
app := kratos.New()
app.Append(kratos.Hook{
    OnStart: func(ctx context.Context) error {
        return svr.Start()
    },
    OnStop: func(ctx context.Context) error {
        return svr.Shutdown(ctx)
    },
})
if err := app.Run(); err != nil {
    log.Printf("app failed: %v\n", err)
    return
}
```

> 另外还支持静态生成代码，便于诊断（而不是在运行时通过 reflection 实现）。

## API 设计

为了统一检索和规范 API，可在内部建立统一的仓库，整合所有对内对外 API（可参考 [googleapis/googleapis](https://github.com/googleapis/googleapis)、[envoyproxy/data-plane-api](https://github.com/envoyproxy/data-plane-api)、[istio/api](https://github.com/istio/api)）。

- API 仓库，方便跨部门协作。
- 版本管理，基于 git 控制。
- 规范化检查（API lint）。
- API design review（变更 diff）。
- 权限管理，目录 OWNERS

### gRPC

[gRPC](https://grpc.io/) 是一种高性能的开源统一 RPC 框架：

- 基于 Proto 的请求响应，支持多种语言。
- 轻量级、高性能：序列化支持 Protocol Buffer 和 JSON。
- 可插拔：支持多种插件扩展。
- IDL：基于文件定义服务，通过 proto3 生成指定语言的数据结构、服务端接口以及客户端 Stub（所有语言都是一致的，可代表文档）。
- 移动端基于标准 HTTP/2 设计，支持双向流、消息头压缩、单 TCP 多路复用、服务端推送等特性，使得 gRPC 在移动端设备上更加省电和网络流量（传输层透明，便于升级到 HTTP/3、QUIC）。

```
syntax = "proto3";

package rpc_package;

service HelloWorldService {
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}
message HelloRequest {
    string name = 1;
}
message HelloReply {
    string message = 1;
}
protoc --go_out=.--go_opt=paths=source_relative \ 
    --go-grpc_out=.--go-grpc_opt=paths=source_relative \
    helloworld/helloworld.proto
```

设计原则：

- **服务而非对象，消息而非引用**：促进微服务系统间粗粒度消息交互设计理念。
- **负载无关**：不同服务使用不同的消息类型和编码，例如 protocol buffers、JSON、XML、Thrift。
- **流**：Streaming API。
- **阻塞 / 非阻塞**：支持异步和同步处理在客户端和服务端间交互消息序列。
- **元数据交换**：常见的横切关注点，如认证或跟踪，依赖数据交换。
- **标准化状态码**：客户端以有限方式响应 API 调用返回的错误（优先使用标准的 HTTP 状态码）。

> 设计时不要过早关注性能问题，先实现标准化。

### 目录结构

参考：

```
|-- bapis
    |-- api
        |-- echo
            |-- v1
                |-- echo.proto
                |-- OWNERS 权限拥有者
    |-- rpc 
        |-- status.proto 内部状态码
    |-- metadata 框架元信息
        |-- locale
        |-- network
        |-- device
    |-- annotations 注解定义 options
    |-- third_party 第三方引用
```

### 兼容性

维护 API 需要注意总是保持向后兼容（非破坏性）的修改：

- 为服务添加 API（从协议的角度来看始终是安全的）。
- 为请求消息添加字段（客户端在新版和旧版中对字段的处理保持一致，添加请求字段就是兼容的）。
- 为响应消息添加字段（在不改变其他字段的前提下，非资源响应消息可以扩展而不必破坏客户端的兼容性。即使会引入冗余，先前在响应中填充的任何字段应继续使用相同的语义填充）。

应避免破坏性的修改（一般需要修改 major 版本号）：

- 删除或重命名服务，字段，方法或枚举值（如果客户端代码可引用的内容，删除或重命名它都是不兼容的变化）。
- 修改字段的类型（即使新类型是传输格式兼容的，也可能导致客户端生成代码发生变化，对于静态语言而言会容易引入编译错误）。
- 修改现有请求的可见行为（客户端通常依赖于 API 行为和语义，即使没有被明确支持或记录。在大多数情况下，修改 API 数据的行为或语义将被消费者视为是破坏性的。如果行为没有加密隐藏，应该假设用户已经发现并依赖于它）。
- 给资源消息添加读写字段。

### 命名规范

包名为应用的标识（appid），用于生成 gRPC 请求路径或 proto 之间引用 Message。

文件中声明的包名称应该与产品和服务名称一致，带有版本的 API 的软件包名称必须以此版本结尾。

参考（）：

|            | 示例                               |
| ---------- | ---------------------------------- |
| 产品名称   | Google Calendar API                |
| 服务名称   | calendar.googleapis.com            |
| 软件包名称 | google.calendar.v3                 |
| 接口名称   | google.calendar.v3.CalendarService |
| 来源目录   | //google/calendar/v3               |
| API 名称   | calendar                           |

请求 URL：`/package_name.version.service_name/method`

### 原始字段

gRPC 默认使用 Protobuf v3 格式，去除了 required 和 optional 关键字（默认全部是 optional）。没有赋值的字段默认是基础类型字段的默认值，比如 0 或者 “”。

```
// proto2
message Account {
    // 必须
    required string name = 1;
    // 可选，默认值改为 -1.0，有 haxXxx 方法。
    optional double profit_rate = 2 [default=-1.0];
}

// proto3
message Account {
    // 都是可选，默认值为 0 和 ""，无 hasXxx 方法。
    string name = 1
    double profit_rate = 2;
}
```

将无法区分默认值或未赋值。因此在 Protobuf v3 中建议使用：[wrappers.proto](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/wrappers.proto)。Wrapper 类型的字段即包装一个 message，使用时变为指针。

```
message DoubleValue {
    double value = 1;
}
```

> Protobuf 作为强 schema 约束的描述文件，也方便扩展，因此也可以用于配置文件定义。

### 异常处理

首先由于会为服务监控带来麻烦，明确禁止在 HTTP Status Code 中统一设置为 200、在 Body 中再定义 code 字段标记具体错误类型的做法。

**使用标准错误配合具体错误**：比如服务端使用一个标准 `google.rpc.Code.NOT_FOUND` 错误代码告知客户端无法找到特定资源（大类：404，小类：具体资源）。

- 状态空间变小降低了文档的复杂性。
- 在客户端库中提供了更好的惯用映射，降低了逻辑复杂性。
- 不限制是否包含可操作信息（`/google/rpc/error_details`）。

**错误传播**：如果 API 服务依赖于其他服务，不应盲目地将服务错误传播到客户端。在翻译错误时建议：

- 隐藏详细信息和机密信息。
- 调整负责该错误的一方。比如一个服务端从其它服务接收到 INVALID ARGUMENT 错误，应该将 INTERNAL 传播给自己的调用者。

**全局错误码** 是松散、契约易被破坏的，应在每个服务传播错误时做一次翻译，保证每个服务 + 错误枚举是唯一的，定义在 proto 中（可作为文档）。

### 设计规则

有时接口复用会带来歧义，比如一些字段给 A 方法用、另一些给 B 方法用；如果为不同方法定义 struct 又会造成冗余。

```
service LibraryService {
    rpc UpdateBook(UpdateBookRequest) returns (Book);
}
message UpdateBookRequest { Book book = 1;}
message Book {
    string name = 1;
    string author = 2;
    string title = 3;
    bool read = 4;
}
```

gRPC 推荐的做法是利用 FieldMask 的部分更新：客户端可执行需要更新的字段信息，空 FieldMask 默认应用到所有字段。

```
service LibraryService {
    rpc UpdateBook(UpdateBookRequest) returns (Book);
}

message UpdateBookRequest {
    Book book = 1;
    google.protobuf.FieldMask mask = 2;
}
```

## 配置管理

通常包括以下内容：

- **环境配置**：Region、Zone、Cluster、Environment、Color、Discovery、AppID、Host 等之类的环境变量信息，通过在线运行时平台打入到容器或物理机，供 kit 库读取使用。比如 Dev、UAT、Preprod、Prod、DR 等环境。
- **静态配置**：即资源需要初始化的配置信息，比如 HTTP/gRPC server、Redis、MySQL 等，通常不建议运行时变更（很可能会导致业务出现不可预期的事故），变更静态配置和发布 bianry app 没有区别，应该走迭代发布流程。在设计上应考虑 **协议卸载**：将有状态、需要运行时变更的业务逻辑下沉，而避免安排在接入节点层（比如TCP Server，无状态）。
- **动态配置**：应用程序可能需要比较简单的在线开关控制业务策略，会频繁的调整和使用，这类用于动态变更业务流的（比如 AB Test 的 flag，一般是基础类型 int、bool 等）配置可收归在一起，考虑结合 [expvar](https://pkg.go.dev/expvar) 使用，与配置中心打通。
- **全局配置**：通常各类依赖组件、中间件都有大量默认配置或指定配置，在各个项目里大量复制容易出现意外。所以使用配置模板来定制化常用组件，在特化应用进行局部替换。

配置传参先参考 net/http 库：

```
func main() {
    s := &http.Server{
        Addr: ":8080",
        Handler: nil,
        ReadTimeout: 10 * time.Second,
        WriteTimeout: 10 * time.Second,
        MaxHeaderBytes: 1 << 20,
    }
    log.Fatal(s.ListenAndServe())
}
```

缺点是无法获知修改公共字段是否会有副作用，字段的含义也要自行查阅文档。

改进是自行设计 config struct，建议使用 functional options：

- 符合编程直觉，可实现高度的可配置化，容易维护和扩展。
- 自文档描述，代码可读、容易上手。
- 代码直观，无歧义（比如空值）。

```
type Server struct {
    Addr     string        // required
    Port     int           // required
    Protocol string        // not null, default TCP
    Timeout  time.Duration // not null, default 30
    MaxConn  int           // not null, default 1024
    TLS      *tls.Config   //
}

type Option func(*Server)

func Protocol(p string) Option {
    return func(s *Server) {
        s.Protocol = p
    }
}
func Timeout(timeout time.Duration) Option {
    return func(s *Server) {
        s.Timeout = timeout
    }
}
func MaxConn(maxConn int) Option {
    return func(s *Server) {
        s.MaxConn = maxConn
    }
}
func TLS(tls *tls.Config) Option {
    return func(s *Server) {
        s.TLS = tls
    }
}

func NewServerFP(addr string, port int, options ...Option) (*Server, error) {
    // 有一个可变参数 options 可以传出多个上面的函数，for-loop 设置 Server 对象。
    srv := Server{
        Addr:     addr,
        Port:     port,
        Protocol: "tcp",
        Timeout:  30 * time.Second,
        MaxConn:  1000,
        TLS:      nil,
    }
    for _, option := range options {
        option(&srv)
    }
    //...
    return &srv, nil
}

func TestFunctionalOptions(t *testing.T) {
    s1, _ := NewServerFP("localhost", 1024)
    s2, _ := NewServerFP("localhost", 2048, Protocol("udp"))
    s3, _ := NewServerFP("0.0.0.0", 8080, Timeout(300*time.Second), MaxConn(1000))
    fmt.Println(s1, s2, s3)
}
```

在实践中应注意配置文件到配置数据之间映射的解耦：

- 仅保留 options API。
- config file 和 options struct 解耦：比如利用 gRPC 的 Protobuf 的强 schema 约束定义 Config 对象，实现语义验证、语法高亮和 lint、格式化。

```
[Config Web UI] <----+---------+
                     |         ↓
[Config API] --------+--> [Config Data] ----> [System]
                     |         ↑
[Config Language] <--+---------+
```

YAML：需要先转换成 JSON，再转成 Protobuf。Protobuf 的 Config 对象不能直接扩展方法，所以还需要加一个 Options 方法。

```
func ApplyYAML(s *redis.Config, yml string) error {
    js, err := yaml.YAMLToJSON([]byte(yml))
    if err != nil {
        return err
    }
    return ApplyJSON(s, string(js))
}
// Options apply config to options.
func Options(c *redis.Config) []redis.Options {
    return []redis.Options{
        redis.DialDatabase(c.Database),
        redis.DialPassword(c.Password),
        redis.DialReadTimeout(c.ReadTimeout),
    }
}
```

Protobuf：使用 wrap struct 区分是否有值。

```
syntax = "proto3";
import "google/protobuf/duration.proto";
package config.redis.v1;

// redis config.
message redis {
    string network = 1;
    string address = 2;
    int32 database = 3;
    string password = 4;
    google.protobuf.Duration read_timeout = 5;
}
```

最终实现配置注入：

```
func main() {
    // load config file from yaml.
    c := new(redis.Config)
    _ = ApplyYAML(c, loadConfig())
    r, _ := redis.Dial(c.Network, c.Address, Options(c)...)
}
```

### 最佳实践

实现代码变更系统功能是冗长且复杂的过程，往往还涉及 CR、测试等流程。而更改单个配置选项也可能对功能产生重大影响，且通常情况下修改配置还容易被忽略、未经测试就上线。

配置管理的目标：

- 避免复杂：依赖的通用基础中间件使用配置中心支持的全局配置化模板。
- 多样的配置：配置模板通过覆盖某些字段实现多样化。
- 区分必选项和可选项，向简单化努力：尽可能减少必要的配置项（最佳实践）。
- 以基础设施 -> 面向用户进行转变。
- 配置的防御编程。
- 权限和变更跟踪。
- 配置的版本和应用对齐。
- 安全的配置变更：逐步部署、回滚更改、自动回滚。

## 模块管理

Go 依赖管理是通过 Git 仓库模式实现，并随着版本的更迭逐渐完善。

早期是 **GOPATH** 模式：GOPATH 目录是所有工程的公共依赖包目录，所有需要编译的 go 工程的依赖包都放在 GOPATH 目录下。

后续引入多版本支持的 **Vendor 特性**：go 1.6 之后开启了 vendor 目录，以支持各个工程对于不同版本的依赖包使用的需求（每个工程拷贝一份代码）。

**Go Module** 管理：Go1.11 实现了依赖包的升级更新，在 Go1.13 版本后默认打开。

### GOPATH

GOPATH 为 Go 开发环境时所设置的一个环境变量。

历史版本的 go 语言开发时，需要将代码放在 GOPATH 目录的 src 文件夹下。go get 命令获取依赖，也会自动下载到 GOPATH 的 src 下。以下命令会将代码下载到 `$GOPATH/src/github.com/foo/bar`。

```
go get github.com/foo/bar
```

GOPATH 具体结构如下，必须包含三个文件夹：

```
GOPATH
|-- bin    二进制文件
|-- pkg    预编译文件（加快后续编译速度）
|-- src    源代码
    |-- github.com
```

### GO Modules

从 Go 1.11 开始初步支持，解决了依赖版本的信息管理，并且保证安全性 。

由 go.mod 和 go.sum 组成，包括依赖模块路径定义，通过 checksum 保证包的安全性，并且可以在 GOPATH 外创建和编译项目。

使用 `go mod init` 命令初始化项目，生成 go.mod 文件：

```
go mod init example.com.hello
cat go.mod
module example.com/hello

go 1.16
```

使用 `go get github.com/sirupsen/logrus` 可下载或更新依赖包：

```
module example.com/hello

go 1.16

require github.com/sirupsen/logrus v1.8.1
```

各关键字含义：

- module： 定义当前项目的模块路径。
- go： 标识当前模块的 Go 语言版本。
- require： 依赖包及其版本。
- exclude： 在使用中排除特定的模块版本。
- replace：替换 require 中声明的依赖，使用另外的依赖及其版本号。

#### Checksum

为解决 Go Modules 的包被篡改的安全隐患，引入 go.sum 文件以记录每个依赖包的哈希值，在构建时如果本地的依赖包 hash 值与 go.sum 文件中记录的不一致，则会拒绝构建。

- go.sum 文件中每行记录由 module 名、版本和哈希组成，以空格分隔。
- 引入新依赖时，通常使用 `go get` 命令获取，将包下载到本地缓存目录 `$GOPATH/pkg/mod/cache/download`，该包后缀为 .zip，并把哈希运算同步到 go.sum 文件中。
- 在构建应用时，从本地缓存中查找所有 go.mod 中记录的依赖包，并计算本地包的哈希值 ，与 go.sum 中的记录对比，如果校验失败，go 命令将拒绝构建。

### Proxy

Go 1.13 的 GOPROXY 默认为 [https://proxy.golang.org，在国内需要配置代理才能使用。GOPROXY](https://proxy.golang.xn--org%2C-ux8is5bfzj85qrnap81iff2ccld904crhfz75bsl3a8vw.goproxy/) 也可以解决公司内部的使用问题：

- 访问内网的 git server。
- 防止公网仓库变更导致线上编译失败或者紧急回退失败。
- 满足公司审计和安全需要。
- 防止内部开发人员配置不当造成 import path 泄露。
- cache 热点依赖，降低公司公网出口带宽。

```
export GOPROXY=https://goproxy.io,direct
# 不走 proxy 的私有仓库或组，以逗号分隔。
export GOPRIVATE=git.mycompany.com,github.com/my/private
```

### Private

用于控制 go 命令把某些仓库视作私有仓库，可以跳过 proxy server 和 checksum 检查，GOPRIVATE 的值同时作为 GONOPROXY 和 GONOSUMDB 默认值：

```
# 以逗号分隔。
export GOPRIVATE=*.corp.example.com,github.com/org_name
```

推荐同时配置 GOPROXY 和 GOPRIVATE 使用，GOPRIVATE 也可以识别 Git SSH KEY 进行权限效验。

### GOPROXY 编译部署

goproxy.io 是 Go Modules 开源代理，也可作为公司内部代理。

```
# 下载编译：
git clone https://github.com/goproxyio/goproxy.git
cd goproxy
go build

# 运行代理：
# ./goproxy -listen=0.0.0.0:8081 -cacheDir=/tmp/cache -proxy https://goproxy.io -exclude "github.com/private"
# 
#   -cacheDir 指定 Go 模块的缓存目录
#   -exclude proxy 模式下指定不经过上游服务器的 path
#   -listen 服务监听端口，默认 8081
#   -proxy 指定上游 proxy server，推荐 goproxy.io
```

访问内网 Git 仓库：

- 用户本地配置 `GONOSUMDB=github.com/private`
- goproxy server 配置 exclude 进行排除所代理仓库
- goproxy server 配置 SSH Key，并且在仓库添加只读权限
- goproxy server 配置 .gitconfig 把 ssh 替换成 http 方式访问

```
[url "git@github.com:"]
    insteadOf = https://github.com/
[url "git@github.com:"]
    insteadOf = https://gitlab.com/
```

## 测试

小型测试带来优秀的代码质量、良好的异常处理、优雅的错误报告；大中型测试会带来整体产品质量和数据验证。

不同类型的项目对测试的需求不同，总体上有 70/20/10 经验法则：70% 小型测试，20% 中型测试，10% 大型测试。

如果一个项目是面向用户的，拥有较高的集成度或用户接口比较复杂，就应该有更多的中型和大型测试；如果是基础平台或者面向数据的项目（例如索引或网络爬虫），则最好有大量的小型测试。

### 单元测试

单元测试的基本要求：

- 快速
- 环境一致
- 任意顺序
- 并行

基于 docker-compose 实现跨平台跨语言环境的容器依赖管理方案，以解决运行 unittest 场景下的容器依赖问题:

- 本地安装 Docker。
- 无侵入式的环境初始化。
- 快速重置环境。
- 随时随地运行(不依赖外部服务)。
- 语义式 API 声明资源。
- 真实外部依赖，而非 in-process 模拟。

包含测试的项目目录结构：

```
|-- service
    |-- api
    |-- cmd
    |-- configs
    |-- internal
    |-- test
        |-- docker-compose.yaml
        |-- database.sql
```

要满足以下原则：

- 正确地对容器内服务进行健康检测，避免测试启动时资源还未准备好。
- 应该交由 app 自己初始化数据，比如 db 的 scheme，初始 sql 数据等。为了满足测试的一致性，在每次结束后都会销毁容器。
- 在单元测试开始前导入封装好的 testing 库，方便启动和销毁容器。
- 对于 service 的单元测试，使用 gomock 等库把 mock DAO 层。在设计包时，应该面向接口编程。
- 在本地启动依赖 Docker 容器，在 CI 环境里执行单元测试，需要考虑物理机中的容器网络，或在容器里再次启动一个 Docker。

```
func TestMain(m *testing.M) {
    flag.Set("f", "./test/docker-compose.yaml")
    flag.Parse()
    if err := lich.Setup(); err != nil {
        panic(err)
    }
    defer lich.Teardown()
    if ret := m.Run(); ret != 0 {
        panic(ret)
    }
}
```

### 最佳实践

利用 go 官方提供的 Subtests + Gomock 完成整个单元测试。对于每层代码：

- /api：更适合进行集成测试，使用 API 测试框架（YApi）维护大量业务测试 case。
- /data：使用 docker compose 模拟底层基础设施，可以去掉 infra 的抽象层。
- /biz：依赖 repo、rpc client，利用 gomock 模拟 interface 实现来进行业务单元测试。
- /service：依赖 biz 实现，构建 biz 实现类传入进行单元测试。

一般的开发测试流程：

- 基于 git branch 进行 feature 开发。
- 开发过程，在本地执行单元测试。
- 提交 gitlab merge request 进行 CI 的单元测试。
- 基于 feature branch 进行构建。
- 完成功能测试之后合并 master。
- 上线前进行集成测试。
- 上线后进行回归测试。

## 参考

- [Package Oriented Design (ardanlabs.com)](https://www.ardanlabs.com/blog/2017/02/package-oriented-design.html)
- [Design Philosophy On Packaging (ardanlabs.com)](https://www.ardanlabs.com/blog/2017/02/design-philosophy-on-packaging.html)
- [golang-standards/project-layout: Standard Go Project Layout (github.com)](https://github.com/golang-standards/project-layout)
- [浅析VO、DTO、DO、PO的概念、区别和用处 - 随风而逝,只是飘零 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zxf330301/p/6534643.html)
- [阿里文娱技术专家战獒: 领域驱动设计详解之What, Why, How?_中生代技术-CSDN博客](https://blog.csdn.net/k6T9Q8XKs6iIkZPPIFq/article/details/109192475?ops_request_misc={"request_id":"160561008419724839224387","scm":"20140713.130102334.pc_all."}&request_id=160561008419724839224387&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v28-6-109192475.first_rank_ecpm_v3_pc_rank_v2&utm_term=阿里技术专家详解DDD系列&spm=1018.2118.3001.4449)
- [阿里技术专家详解 DDD 系列- Domain Primitive_chikuai9995的博客-CSDN博客](https://blog.csdn.net/chikuai9995/article/details/100723540?biz_id=102&utm_term=阿里技术专家详解DDD系列&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-100723540&spm=1018.2118.3001.4449)
- [阿里技术专家详解DDD系列 第二弹 - 应用架构_淘系技术-CSDN博客](https://blog.csdn.net/Taobaojishu/article/details/101444324?ops_request_misc={"request_id":"160561008419724838528569","scm":"20140713.130102334.."}&request_id=160561008419724838528569&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-101444324.first_rank_ecpm_v3_pc_rank_v2&utm_term=阿里技术专家详解DDD系列&spm=1018.2118.3001.4449)
- [阿里技术专家详解DDD系列 第三讲 - Repository模式_淘系技术-CSDN博客](https://blog.csdn.net/taobaojishu/article/details/106152641)
- [Errors | Cloud APIs | Google Cloud](https://cloud.google.com/apis/design/errors)
- [贫血，充血模型的解释以及一些经验_知识库_博客园 (cnblogs.com)](https://kb.cnblogs.com/page/520743/)
- [领域驱动设计 实践手册(1.Get Started) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/105466656)
- [DDD 实践手册(2. 实现分层架构) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/105648986)
- [DDD 实践手册(3. Entity, Value Object) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/106634373)
- [DDD 实践手册(4. Aggregate — 聚合) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/107347593)
- [DDD 实践手册(5. Factory 与 Repository) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/109048532)
- [DDD 实践手册(6. Bounded Context - 限界上下文) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/110252394)
- [01、DDD和微服务的关系 - 简书 (jianshu.com)](https://www.jianshu.com/p/dfa427762975)
- [Domain Driven Design in Go – Citerus](https://www.citerus.se/go-ddd/)
- [Domain Driven Design in Go: Part 2 – Citerus](https://www.citerus.se/part-2-domain-driven-design-in-go/)
- [Domain Driven Design in Go: Part 3 – Citerus](https://www.citerus.se/part-3-domain-driven-design-in-go/)
- [当中台遇上DDD，我们该如何设计微服务？](https://www.jianshu.com/p/5732b69bd1a1)
- [领域驱动设计系列文章（1）——通过现实例子显示领域驱动设计的威力 - Cat Qi - 博客园 (cnblogs.com)](https://www.cnblogs.com/qixuejia/p/10789612.html)
- [领域驱动设计系列文章（2）——浅析VO、DTO、DO、PO的概念、区别和用处 - Cat Qi - 博客园 (cnblogs.com)](https://www.cnblogs.com/qixuejia/p/4390086.html)
- [领域驱动设计系列文章（3）——有选择性的使用领域驱动设计 - Cat Qi - 博客园 (cnblogs.com)](https://www.cnblogs.com/qixuejia/p/10789621.html)
- [区分 Protobuf 中缺失值和默认值 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/46603988)
- [protobuf/wrappers.proto at master · protocolbuffers/protobuf (github.com)](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/wrappers.proto)
- [Functional options for friendly APIs – The acme of foolishness (cheney.net)](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis)
- [command center: Self-referential functions and the design of options](https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html)
- [Creating Good API Errors in REST, GraphQL and gRPC | APIs You Won’t Hate - A community that cares about API design and development. (apisyouwonthate.com)](https://apisyouwonthate.com/blog/creating-good-api-errors-in-rest-graphql-and-grpc/)
- [Clean Coder Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [GopherCon 2018: Kat Zien - How Do You Structure Your Go Apps - YouTube](https://www.youtube.com/watch?v=oL6JBUk6tj0)
- [zitryss/go-sample: Go Project Sample Layout (github.com)](https://github.com/zitryss/go-sample)
- [paper-code/packageorienteddesign.md at master · danceyoung/paper-code (github.com)](https://github.com/danceyoung/paper-code/blob/master/package-oriented-design/packageorienteddesign.md)
- [Clean Architecture using Golang. Update | by Elton Minetto | Medium](https://eminetto.medium.com/clean-architecture-using-golang-b63587aa5e3f)
- [Standard Package Layout. Addressing one of the biggest technical… | by Ben Johnson | Medium](https://medium.com/@benbjohnson/standard-package-layout-7cdbc8391fc1)
- [410 Deleted by author — Medium](https://medium.com/wtf-dial/wtf-dial-domain-model-9655cd523182)
- [Trying Clean Architecture on Golang | Hacker Noon](https://hackernoon.com/golang-clean-archithecture-efd6d7c43047)
- [Trying Clean Architecture on Golang — 2 | Hacker Noon](https://hackernoon.com/trying-clean-architecture-on-golang-2-44d615bf8fdf)
- [Applying The Clean Architecture to Go applications • Manuel Kießling (kiessling.net)](https://manuel.kiessling.net/2012/09/28/applying-the-clean-architecture-to-go-applications/)
- [katzien/go-structure-examples: Examples for my talk on structuring go apps (github.com)](https://github.com/katzien/go-structure-examples)
- [Ashley McNamara + Brian Ketelsen. Go best practices. - YouTube](https://www.youtube.com/watch?v=MzTcsI6tn-0)
- [DTO to Entity and Entity to DTO Conversion - Apps Developer Blog](https://www.appsdeveloperblog.com/dto-to-entity-and-entity-to-dto-conversion/)
- [I’ll take pkg over internal (travisjeffery.com)](https://travisjeffery.com/b/2019/11/i-ll-take-pkg-over-internal/)
- [wire/best-practices.md at main · google/wire (github.com)](https://github.com/google/wire/blob/main/docs/best-practices.md)
- [wire/guide.md at main · google/wire (github.com)](https://github.com/google/wire/blob/main/docs/guide.md)
- [Compile-time Dependency Injection With Go Cloud’s Wire - The Go Blog (golang.org)](https://blog.golang.org/wire)
- [google/wire: Compile-time Dependency Injection for Go (github.com)](https://github.com/google/wire)
- [Integration Testing in Go: Part I - Executing Tests with Docker (ardanlabs.com)](https://www.ardanlabs.com/blog/2019/03/integration-testing-in-go-executing-tests-with-docker.html)
- [Integration Testing in Go: Part II - Set-up and Writing Tests (ardanlabs.com)](https://www.ardanlabs.com/blog/2019/10/integration-testing-in-go-set-up-and-writing-tests.html)
- [Testable Examples in Go - The Go Blog (golang.org)](https://blog.golang.org/examples)
- [Using Subtests and Sub-benchmarks - The Go Blog (golang.org)](https://blog.golang.org/subtests)
- [The cover story - The Go Blog (golang.org)](https://blog.golang.org/cover)
- [Keeping Your Modules Compatible - The Go Blog (golang.org)](https://blog.golang.org/module-compatibility)
- [Go Modules: v2 and Beyond - The Go Blog (golang.org)](https://blog.golang.org/v2-go-modules)
- [Publishing Go Modules - The Go Blog (golang.org)](https://blog.golang.org/publishing-go-modules)
- [Module Mirror and Checksum Database Launched - The Go Blog (golang.org)](https://blog.golang.org/module-mirror-launch)
- [Migrating to Go Modules - The Go Blog (golang.org)](https://blog.golang.org/migrating-to-go-modules)
- [Using Go Modules - The Go Blog (golang.org)](https://blog.golang.org/using-go-modules)
- [Go Modules in 2019 - The Go Blog (golang.org)](https://blog.golang.org/modules2019)
- [Testing with GoMock: A Tutorial - codecentric AG Blog](https://blog.codecentric.de/en/2017/08/gomock-tutorial/)
- [gomock · pkg.go.dev](https://pkg.go.dev/github.com/golang/mock/gomock)
- [A GoMock Quick Start Guide. An opinionated tutorial for unit… | by Che Dan | Better Programming](https://betterprogramming.pub/a-gomock-quick-start-guide-71bee4b3a6f1?gi=e44758036c10)
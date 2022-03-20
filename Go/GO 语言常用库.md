# GO 语言常用库

### 数据库框架

在Go语言世界中，[beego orm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fastaxie%2Fbeego%2Ftree%2Fmaster%2Form)、[gorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fjinzhu%2Fgorm)、[sqlx](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fjmoiron%2Fsqlx)、[gorp](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fgo-gorp%2Fgorp)、[xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fgo-xorm%2Fxorm)是我已知在Github中stars数最多Go数据库框架，这几个都是Go语言世界中老牌数据库框架库。

其中[beego orm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fastaxie%2Fbeego%2Ftree%2Fmaster%2Form)是[beego](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fastaxie%2Fbeego)自带的orm框架库，统计star数的时候是按[beego](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fastaxie%2Fbeego)的star数统计的（[beego](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fastaxie%2Fbeego)之前还有一个数据库框架[beedb](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fastaxie%2Fbeedb)，由于谢大在2014年就未在维护此库，所以没有出现在我的统计列表中）。

而[sqlx](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fjmoiron%2Fsqlx)和[xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fgo-xorm%2Fxorm)则是笔者最喜欢和实际开发中最多使用的Go数据库框架库。值的一提的是[sqlx](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fjmoiron%2Fsqlx)还有3个扩展库，一个是[sqalx](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fheetch%2Fsqalx)，它使sqlx提供对嵌套事务的支持，另外两个名字都叫sqlt，其中第一个[sqlt](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fit512%2Fsqlt)库，使sqlx支持sql模板和类mybatis的sql配置；第二个[sqlt](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Falbert-widi%2Fsqlt)库，则使sqlx支持数据库主从数据源，读写分离；

另外一个值的一提的是[xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fgo-xorm%2Fxorm)也有一个定制增强版[xormplus/xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fxormplus%2Fxorm)，使得xorm支持sql模板和类mybatis的sql配置，支持动态sql，支持嵌套事务，支持类似Java中Spring的事务传播机制，支持数据库的读写分离（master/slave）。同时它和[xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fgo-xorm%2Fxorm)一样内置支持SQL Builder。

最后还有一个比较有意思的Go数据库框架库是[argen](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fargen)，它在具体实现中使用了annotation方式，这在Go语言开发库中是比较少见的，对笔者而言是一个值得阅读源码和学习的库，它使Go语言又多了一种思维方式。

在实际开发中笔者经常是依据业务复杂度，数据库sql复杂度，在[sqlx](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fjmoiron%2Fsqlx)、[xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fgo-xorm%2Fxorm)和[xormplus/xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fxormplus%2Fxorm)中进行数据库框架库选型，笔者大多数时候更倾向[xormplus/xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fxormplus%2Fxorm)。如果您也有好的Go语言数据库框架推荐，不妨在留言区留言，大家一起交流分享。

### 模板引擎

提起Go语言模板引擎，很多人自然会想到[pongo2](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fflosch%2Fpongo2)模板引擎，这也是笔者接触最早的Go语言模板引擎，几乎所有的Go主流流行的Web框架都对它提供支持，你甚至在[xormplus/xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fxormplus%2Fxorm)这个数据库框架库中也能看到他的身影，它是一个像django语法的Go语言模板引擎，如果你以前做过python开发，那你一定会对它无比亲切。

另一个笔者喜欢的Go语言模板引擎是[jet](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FCloudyKit%2Fjet)，它功能强大，且高效，性能相当出色。另外一大特点是IDE支持，它有一个IDEA插件可供开发者使用，Github地址：[https://github.com/jhsx/GoJetPlugin](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fjhsx%2FGoJetPlugin)。这也是目前笔者主要使用的Go语言模板引擎之一（以前还有一些老项目是使用[pongo2](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fflosch%2Fpongo2)模板引擎的，另外[xormplus/xorm](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fxormplus%2Fxorm)中的sql模板也是采用[pongo2](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fflosch%2Fpongo2)）。

还有一个笔者想提及的Go语言模板引擎库是[go-template](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fkataras%2Fgo-template)，它更像一个模板引擎适配器，它的最大特色是同时支持standard html/template、amber、django、handlebars、pug(jade)、markdown六种模板引擎。

Go语言模板引擎其实主要分两大类，一类是非预编译生成Go代码的模板引擎（如以上提及的这些），另一类则是预编译生成Go代码的模板引擎，这一类模板引擎由于先天优势，性能将更为出色一些，是否采用这类引擎，要看你项目的需求来权衡。这类引擎的佼佼者是[hero](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fshiyanhui%2Fhero)，也是一位国人开发的Go语言模板引擎库。

目前笔者使用的Go语言模板引擎主要是[pongo2](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fflosch%2Fpongo2)，[jet](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FCloudyKit%2Fjet)和[hero](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fshiyanhui%2Fhero)。

## 类型转换

### Cast

```go
import github.com/spf13/cast
```

## 配置文件

### viper

```go
import github.com/spf13/viper
```



## 未分类

### UUID

```go
import uuid "github.com/satori/go.uuid"
```

### JSON

```go
import "github.com/tidwall/gjson"	// 8.1K
```

### 散列

```go
import "crypto/md5"
```



### JWT

```go
import "github.com/dgrijalva/jwt-go"
import "github.com/gbrlsnchs/jwt/v3"
```

### 日志Log

```go
import "github.com/uber-go/zap"
import log "github.com/sirupsen/logrus"
```

### 定时任务

```bash
go get -u github.com/robfig/cron/v3
```

### 权限管理

```bash
go get github.com/casbin/casbin/v2
```

![img](GO 语言常用库.assets/1460000022931684)

### 命令行

```bash
go get -u github.com/urfave/cli/v2
```

### 系统和硬件信息

```bash
go get github.com/shirou/gopsutil
```

### 数据校验

```bash
go get gopkg.in/go-playground/validator.v10
go get github.com/asaskevich/govalidator
```

### 日期时间

```bash
go get github.com/uniplaces/carbon
```

### 绘图库

主要用于绘制数据可视化，如折线图、饼图、柱状图等。

```go
import (
  "gonum.org/v1/plot"
  "gonum.org/v1/plot/plotter"
  "gonum.org/v1/plot/plotutil"
  "gonum.org/v1/plot/vg"
)
```

### 工具类

```bash
go get github.com/polaris1119
```

### IP地址

```bash
go get inet.af/netaddr
```

### UUID

```
go get github.com/gofrs/uuid
```

### HashId   加密Id

```
github.com/speps/go-hashids
```

## 分布式文件系统

### go-FastDFS

项目地址：https://github.com/sjqzhang/go-fastdfs

### MinIO

**MinIO**;是一款开源的对象存储服务器,兼容亚马逊的S3协议 , 对Kubernetes能够友好的支持,专为AI等云原生工作负载而设计。

官网：http://www.minio.org.cn/

## 爬虫

### Colly 

```
github.com/gocolly/colly
```

官网：http://go-colly.org/

## 静态网页生成器

### Hugo 
Go 编写的静态网站生成器


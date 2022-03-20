# 结合 MySQL 构建一个基于 Go 的 REST API 微服务

本篇迷你指导中，我会结合 MySQL 数据库创建一个简单的 REST-API。完整项目的代码会在文章的最后给出。

## **准备 API**

我们在着手时需要做的第一件事是选择一个用于路由的库。路由就是将一个 URL 与一个可执行的函数连接在一起。我觉得 **[mux 库](https://link.zhihu.com/?target=https%3A//github.com/gorilla/mux)** 在路由功能上表现得很好，当然还有其他可选的库如 **[httprouter](https://link.zhihu.com/?target=https%3A//github.com/gorilla/mux)** 和 **[pat](https://link.zhihu.com/?target=https%3A//github.com/bmizerany/pat)**在性能上也差不多。在本文中我将会使用 mux。

简单起见，我们将会创建一个用于打印一条信息的端点

```go
package main

import (
    "log"
    "net/http"

    "github.com/gorilla/mux"
)

func setupRouter(router *mux.Router) {
    router.
        Methods("POST").
        Path("/endpoint").
        HandlerFunc(postFunction)
}

func postFunction(w http.ResponseWriter, r *http.Request) {
    log.Println("You called a thing!")
}

func main() {
    router := mux.NewRouter().StrictSlash(true)

    setupRouter(router)

    log.Fatal(http.ListenAndServe(":8080", router))
}
```

上面的代码创建了一个路由，将一个 URL 与一个处理函数（代码中是 postFunction）连接在一起，然后启动了一个服务，并将 8080 端口给这个路由使用。

## **连接数据库**

让我们把上面的代码和 MySQL 数据库连接起来。Go 为 SQL 数据库提供了一个接口，但它还需要一个驱动。在这个例子中我使用 **[go-sql-driver](https://link.zhihu.com/?target=https%3A//medium.com/r/%3Furl%3Dhttps%3A%2F%2Fgithub.com%2Fgo-sql-driver%2Fmysql)** 作为驱动。

```go
package db

import (
    "database/sql"

    _ "github.com/go-sql-driver/mysql"
)

func CreateDatabase() (*sql.DB, error) {
    serverName := "localhost:3306"
    user := "myuser"
    password := "pw"
    dbName := "demo"

    connectionString := fmt.Sprintf("%s:%s@tcp(%s)/%s?charset=utf8mb4&collation=utf8mb4_unicode_ci&parseTime=true&multiStatements=true", user, password, serverName, dbName)
    db, err := sql.Open("mysql", connectionString)
    if err != nil {
        return nil, err
    }

    return db, nil
}
```

上面的代码被放在另一个叫做 *db* 的包中，并且假设有一个运行在 *localhost:3306* 的名字叫 *demo* 数据库。返回的数据库变量自动持有这个数据库的连接池。

让我们更新一下上一个代码片段的 *postFunction* 来使用数据库。

```go
func postFunction(w http.ResponseWriter, r *http.Request) {
    database, err := db.CreateDatabase()
    if err != nil {
        log.Fatal("Database connection failed")
    }

    _, err = database.Exec("INSERT INTO `test` (name) VALUES ('myname')")
    if err != nil {
        log.Fatal("Database INSERT failed")
    }

    log.Println("You called a thing!")
}
```

就是这样！它相当简单，当然上面的代码还有一些问题以及功能缺失。

## **结构体和依赖**

如果你检查了上面的代码，你可能已经注意到了每次 API 调用时都会打开一个数据库连接，虽然已经被打开的数据库 **[对并发使用是安全的](https://link.zhihu.com/?target=https%3A//golang.org/pkg/database/sql/%23Open)** 。我们需要一些依赖管理手段来确保我们只打开一次数据库，为此，我们将要使用结构体。

```go
package app

import (
    "database/sql"
    "log"
    "net/http"

    "github.com/gorilla/mux"
)

type App struct {
    Router   *mux.Router
    Database *sql.DB
}

func (app *App) SetupRouter() {
    app.Router.
        Methods("POST").
        Path("/endpoint").
        HandlerFunc(app.postFunction)
}

func (app *App) postFunction(w http.ResponseWriter, r *http.Request) {
    _, err := app.Database.Exec("INSERT INTO `test` (name) VALUES ('myname')")
    if err != nil {
        log.Fatal("Database INSERT failed")
    }

    log.Println("You called a thing!")
    w.WriteHeader(http.StatusOK)
}
```

我们先创建一个叫做 *app* 的新的包来存放我们的结构体和它的 **[方法](https://link.zhihu.com/?target=https%3A//gobyexample.com/methods)**。我们的 *App* 结构体有两个字段；一个是在第 17 行被调用的 *Router*，另一个是在第 24 行被调用的 *Database*。我们同时在第 30 行方法结束的时候手动设置了返回状态码。

main 包以及其中的方法也需要一点小改变来使用新的 *App* 结构体。我们从 main 包中移除 *postFunction* 方法和 *setupRouter* 方法，因为这俩方法已经在 app 包中了。我们留下这些：

```go
package main

import (
    "log"
    "net/http"

    "github.com/gorilla/mux"
    "github.com/johan-lejdung/go-microservice-api-guide/rest-api/app"
    "github.com/johan-lejdung/go-microservice-api-guide/rest-api/db"
)

func main() {
    database, err := db.CreateDatabase()
    if err != nil {
        log.Fatal("Database connection failed: %s", err.Error())
    }

    app := &app.App{
        Router:   mux.NewRouter().StrictSlash(true),
        Database: database,
    }

    app.SetupRouter()

    log.Fatal(http.ListenAndServe(":8080", app.Router))
}
```

为了使用我们的新结构体，我们打开了一个数据库连接并创建了一个新*路由*。然后我们把他们存放到我们新的 *App* 结构体对应字段中。

恭喜！现在你已经有了一个数据库连接了，可以对即将进入的 API 请求并发调用了。

在最后一步中，我们会在路由中添加一个返回 JSON 数据的 GET 方法。我们从添加一个用于填充我们的数据的结构体开始，并且把这些字段映射为 JSON。

```go
package app

import (
    "time"
)

type DbData struct {
    ID   int       `json:"id"`
    Date time.Time `json:"date"`
    Name string    `json:"name"`
}
```

接着，我们在 *app.go* 文件中添加一个用于处理请求并且把数据写回客户端响应的新方法 *getFunction*。这个文件最后看起来是这个样子的。

```go
package app

import (
    "database/sql"
    "encoding/json"
    "log"
    "net/http"

    "github.com/gorilla/mux"
)

type App struct {
    Router   *mux.Router
    Database *sql.DB
}

func (app *App) SetupRouter() {
    app.Router.
        Methods("GET").
        Path("/endpoint/{id}").
        HandlerFunc(app.getFunction)

    app.Router.
        Methods("POST").
        Path("/endpoint").
        HandlerFunc(app.postFunction)
}

func (app *App) getFunction(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    id, ok := vars["id"]
    if !ok {
        log.Fatal("No ID in the path")
    }

    dbdata := &DbData{}
    err := app.Database.QueryRow("SELECT id, date, name FROM `test` WHERE id = ?", id).Scan(&dbdata.ID, &dbdata.Date, &dbdata.Name)
    if err != nil {
        log.Fatal("Database SELECT failed")
    }

    log.Println("You fetched a thing!")
    w.WriteHeader(http.StatusOK)
    if err := json.NewEncoder(w).Encode(dbdata); err != nil {
        panic(err)
    }
}

func (app *App) postFunction(w http.ResponseWriter, r *http.Request) {
    _, err := app.Database.Exec("INSERT INTO `test` (name) VALUES ('myname')")
    if err != nil {
        log.Fatal("Database INSERT failed")
    }

    log.Println("You called a thing!")
    w.WriteHeader(http.StatusOK)
}
```

## **数据库迁移**

我们来为项目添加最后一个功能。当数据库与一个应用或者服务耦合过深时，会造成令人头疼的问题，可以通过适当的处理数据库迁移来解决这个问题。我们会使用 **[migrate 库](https://link.zhihu.com/?target=https%3A//github.com/golang-migrate/migrate)** 来做这件事情，然后扩展我们的 *db* 包。

就是下面这些相当长的嵌入的代码片段。

```go
package db

import (
    "database/sql"
    "fmt"
    "log"
    "os"

    _ "github.com/go-sql-driver/mysql"
    "github.com/golang-migrate/migrate"
    "github.com/golang-migrate/migrate/database/mysql"
    _ "github.com/golang-migrate/migrate/source/file"
)

func CreateDatabase() (*sql.DB, error) {
    // I shortened the code here. Here is where the DB setup were made.
    // In order to save some space I've removed the connection setup, but it can
    // be seen here: https://gist.github.com/johan-lejdung/ecea9dab9b9621d0ceb054cec70ae676#file-database_connect-go

    if err := migrateDatabase(db); err != nil {
        return db, err
    }

    return db, nil
}

func migrateDatabase(db *sql.DB) error {
    driver, err := mysql.WithInstance(db, &mysql.Config{})
    if err != nil {
        return err
    }

    dir, err := os.Getwd()
    if err != nil {
        log.Fatal(err)
    }

    migration, err := migrate.NewWithDatabaseInstance(
        fmt.Sprintf("file://%s/db/migrations", dir),
        "mysql",
        driver,
    )
    if err != nil {
        return err
    }

    migration.Log = &MigrationLogger{}

    migration.Log.Printf("Applying database migrations")
    err = migration.Up()
    if err != nil && err != migrate.ErrNoChange {
        return err
    }

    version, _, err := migration.Version()
    if err != nil {
        return err
    }

    migration.Log.Printf("Active database version: %d", version)

    return nil
}
```

数据库连接打开后，我们添加的 *migrateDatabase* 函数会被调用来开始迁移过程。

我们也会添加一个 MigrationLogger 结构体来处理迁移过程中的日志，代码可以在 **[这里](https://link.zhihu.com/?target=https%3A//github.com/johan-lejdung/go-microservice-api-guide/blob/master/rest-api/db/migrationlogger.go)** 被看到，而且这个结构体在第 45 行被使用。

迁移是通过普通的 sql 语句实现的。迁移文件从第 37 行显示的文件夹中被读取。

每当数据库被打开后，所有未被应用的数据库迁移将会被应用。这样将会使数据库在不需要人为干预的情况下保持最新。

在 *docker-compose* 文件中保存了数据库使得多机开发变得很简单。

## **打包**

一个不能部署的微服务是没有用的，因此我们加一个 Dockerfile 来打包这个应用以便于能够很容易的进行的分发——*然后本文就到此结束了。*

```dockerfile
FROM golang:1.11 as builder
WORKDIR $GOPATH/src/github.com/johan-lejdung/go-microservice-api-guide/rest-api
COPY ./ .
RUN GOOS=linux GOARCH=386 Go build -ldflags="-w -s" -v
RUN cp rest-api /

FROM alpine:latest
COPY --from=builder /rest-api /
CMD ["/rest-api"]
```

构造好的镜像仅仅 **10 MB**！

下面是代码。

https://github.com/johan-lejdung/go-microservice-api-guide
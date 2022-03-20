# Go依赖注入组件-wire | 依赖倒置

为了降低代码耦合度，在处理依赖对象，通常将依赖的对象以参数形式传入，而不是在内部创建依赖的对象。

比如在A对象需要调用B->Add方法，在构造A对象时应该传入B，而不是在A内对B初始化。

在代码需要管理：

1. 依赖的初始化顺序；
2. 依赖之间的关系；

当依赖关系复杂时，深套又深套。

```go
func main(){
    config := NewConfig()
    db := NewDb(config) //依赖config
    userRepo := newUserRepo(db) //依赖db
}
```

依赖注入（DI）的机制负责**自动解决初始化时的依赖关系**

## Go的依赖注入组件：wire

wire是在编译时实现依赖注入的工具，而非在运行时通过反射等方式处理，效率上会好很多。

两个基本概念：

- provider

提供者，看作是某个对象的构造函数，让wire知道该对象如何产生、依赖哪些对象。

- injector

注入器，是由wire生成的函数，injector会按照依赖关系按顺序调用它所需要的提供者provider

```go
package main

import "fmt"

type Config struct {
   User string
   Password string
}

type DB struct {
   Config *Config
}

type UserRepo struct {
   Db *DB
}

type User struct {
   Id int
   Username string
}

func NewUserRepo(db *DB)*UserRepo{
   return &UserRepo{Db: db}
}

func NewDB(config *Config)*DB{
   return &DB{Config: config}
}

type CUser string
type CPassword string
//wire无法区分多个相同类型的参数，用类型别名代替
func NewConfig(user CUser, password CPassword)*Config{
   fmt.Println("---NewConfig ", user, password)
   return &Config{
      User:     string(user),
      Password: string(password),
   }
}

func (repo *UserRepo)Get(id int)(*User){
   user := User{}
   //repo.Db.Where("id=?",id).Find(&user)
   user.Username = "wei"
   user.Id = id

   return &user
}

func main(){
    //手动初始化依赖关系
   //config := NewConfig("root", "123")
   //db := NewDB(config)
   //userRepo := NewUserRepo(db)
   //user := userRepo.Get(2)

    //通过wire自动解决依赖关系
   userRepo := InitUserRepo("root", "123")
   user := userRepo.Get(2)

   fmt.Println("---user:", *user)
}

安装wire
go get github.com/google/wire/cmd/wire


/*wire.go*/
package main
import "github.com/google/wire"

//执行wire命令后，生成injector；
//injector会按照依赖关系依次调用provider函数（依赖对象的构造函数），返回我们所需的对象
func InitUserRepo(user CUser, password CPassword)*UserRepo{
   //将多个prover函数组合起来
   wire.Build(NewUserRepo, NewDB, NewConfig)
   return &UserRepo{}
}
```

> 执行wire命令生成依赖文件：wire_gen.go

### **接口绑定**

以上是依赖具体实现，但从更好的设计原则考虑，应该思考如何写出低耦合的代码或方便对业务逻辑进行封装，设计原则会给我们一些启示，比如依赖倒置：**对象应当依赖抽象（接口），而不是具体实现**。

## 依赖倒置原则：

1. 上层模块不应该依赖底层模块，它们都应该依赖抽象；
2. 抽象不应当依赖实现，实现应当依赖于抽象；

```go
package main

import (
   "fmt"
   "github.com/google/wire"
)
type CUser string
type CPassword string

type User struct {
   Name string
   Age int
}

type UserRepo interface {
   GetUserById(int)(*User, error)
}

type UserService struct {
   Repo UserRepo
}

func NewUserService(repo UserRepo)*UserService{
   return &UserService{
      Repo: repo,
   }
}

type MockUserRepo struct {
   Username CUser
   Password CPassword
}

func (repo *MockUserRepo)GetUserById(id int)(*User, error){
   return &User{
      Name: "1211",
      Age:  12,
   }, nil
}

/*
   Bind将接口UserRepo与具体实现MockUserRepo关联，wire无法处理抽象接口与具体实现之间的关联，需要显示声明；感觉代码上有些啰嗦，
   NewSet将接口UserRepo与对象提供者MockUserRepo进行绑定
*/
var Set = wire.NewSet(wire.Bind(new(UserRepo), new(*MockUserRepo)), NewMockUserRepo)

func NewMockUserRepo(username CUser, passwrod CPassword)*MockUserRepo{
   return &MockUserRepo{
      Username: username,
      Password: passwrod,
   }
}

func main(){
   //一、手动解决依赖
   /*
      1、通过 UserService 对象调用GetUserById， 获得User对象
      2、创建UserService时，依赖接口 UserRepo
      3、MockUserRepo对象实现了接口 UserRepo， 将MockUserRepo作为UserService的依赖对象进行初始化
   */
   //userRepo := NewMockUserRepo("root", "123456")
   //userService := NewUserService(userRepo)
   //
   //user,_ := userService.Repo.GetUserById(1)
   //fmt.Println("---", *user)

   //二、通过wire接口绑定，自动解决依赖
   userService := InitializeUserService("root", "123123")
   user,_ := userService.Repo.GetUserById(1)
   fmt.Println("---", *user)
}
```
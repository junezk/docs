# Golang领域模型-依赖倒置

**前言：**为什么要用整篇文章来写好像跟领域模型干系不大的《依赖倒置》呢？因为《依赖倒置》是六边形架构的核心！毫不夸张的说，不理解《依赖倒置》的程序员只能写功能，没法写出框架来！不论是依赖注入di或者依赖倒置dip，全部都是根据当前成员变量的类型，框架自动注入实例的。区别在于成员变量是指针接收还是接口接收。具体如何自动注入，请看<<六边形架构>>和<<资源库>>章节。

## 一、如果不进行依赖倒置会怎样？

我们先看看什么是依赖倒置，教科书式的解释就是：

> - 高层模块不应依赖于低层模块，二者应依赖于抽象。
> - 抽象不应依赖于细节，细节应依赖于抽象。

我们商品领域服务需要使用`Repository`来持久化数据，中二代码写成这样：

**代码示例**

1. 资源库具体实现基础设施（DB）的功能

1. 外部资源的实现

   ```go
   package repository
   import ...
   func init() {...}
   // Get通过id获取商品实体.
   func (repo *Goods) Get(id int) (goodsEntity *entity.Goods, e error) {...}
   // Save持久化实体.
   func (repo *Goods) Save(entity *entity.Goods) error {...}
   func (repo *Goods) Finds(ids []int) (entitys []*entity.Goods, e error) {...}
   func (repo *Goods) FindsByPage(page, pageSize int, tag string) (entitys []*entity.Goods, e error) {...}
   func (repo *Goods) New(name, tag string, price, stock int) (entityGoods *entity.Goods, e error) {...}
   ```

2. 领域模型依赖资源库，直接依赖repository

   ```go
   package domain
   import ...
   func init() {...}
   // Goods商品领域服务.
   type Goods struct {
   	Worker freedom.Worker        // 运行时，一个请求绑定一个运行时
   	GoodsRepo repository.Goods   //直接依赖repository
   }
   ```

3. 领域模型直接使用依赖的资源库实现

   ```go
   package domain
   import ...
   func init() {...}
   
   // Goods商品领域服务.
   type Goods struct {
   	Worker freedom.Worker		// 运行时，一个请求绑定一个运行时
   	GoodsRepo repository.Goods	// 直接依赖 repository
   }
   // AddStock增加商品库存
   func (g *Goods) AddStock(goodsId, num int) (e error) {
   	entity, e := g.GoodsRepo.Get(goodsld)
   	if e != nil{
   		g.Worker.Logger().Error ("商品库存失败")
            return
       }
       g.Worker.Logger().Info("增加库存")
       entity.AddStock(num)
       return g.GoodsRepo.Save(entity)
   }
   ```

#### 这样做的缺点是什么？

1. 难以维护

   内部（领域模型）通常是业务逻辑和策略，这里就是`DDD`里面的领域模型，一个软件区别于其他软件的核心就在于业务逻辑和策略实现（也就是领域模型），而外部更多的是外部资源等基础设施。PS：内部外部概念可参考前文---《Golang领域模型-六边形架构》

如果领域模型依赖基础设施，那就是业务逻辑依赖技术细节，技术细节的改变将会对业务逻辑产生影响，使其不得不改变。这样是不合理的。

2. 复用困难

   越核心的领域模型，复用价值越高，如果对基础设施进行依赖，那么复用将会变得很困难。

## 二、如何进行依赖倒置？

> 计算机科学中的所有问题都可以通过引入一个间接层得到解决。
>  All problems in computer science can be solved by another level of indirection
>  —— David Wheeler

**实现：**`domain`中引入`dependency`包定义抽象层

```
+ Domain
  > aggregate
  > cart.go
  + dependency
    > dependency.go
  > dto
  > entity
  > goods.go
  > order.go
  > po
  > user.go
```

dependency抽象层

**代码示例：**

1. 定义了商品仓储实现所需要满足的接口。

   ```go
   package dependency
   import ...
   
   // 依赖倒置的接口外部adapter负责实现
   type GoodsRepo interface (
   	Get(id int) (goodsEntity *entity.Goods, e error)
   	Finds(ids []int) (entitys []*entity.Goods, e error)
   	FindsByPage(page, pageSize int, tag string) (entitys 口*entity.Goods, e error)
   	Save(entity *entity.Goods) error
   	New(name, tag string, price, stock int) (entityGoods *entity.Goods, e error)
   }
   ```

   抽象层

2. 商品领域服务成员变量直接引用抽象接口，框架负责依赖注入

```GO
package domain
import ...
// Goods商品领域服务.
type Goods struct {
	Worker freedom.Worker					// 运行时，一个请求绑定一个运行时
	GoodsRepo dependency.GoodsRepo		 	 // 依赖倒置商品资源库
	ShopFactory *aggregate.ShopFactory		 // 依赖注入购买聚合根工厂
}
```

领域模型依赖抽象

1. 商品领域模型中使用抽象出来的`GoodsRepo`方法

```go
package domain
import ...

// Addstock增加商品库存   使用抽象出来的GoodRepo方法
func (g *Goods) AddStock(goodsId, num int) (e error) {
    entiry, e := g.GoodsRepo.Get(goodsld)
	if e != nil {
        g.Worker.Logger().Error("商品库存失败")
	    return
    }
    g.Worker.Logger().Info("增加库存")
	entity.AddStock(num)
	return g.GoodsRepo.Save(entity)
}
```

领域模型依赖抽象

1. 外部资源具体实现抽象接口

   ```go
   package repository
   import ...
   func init() {...}
   // 实现领域模型内的依赖倒置 检查接口是否实现
   var _ dependency.GoodsRepo = new(Goods)
   // Goods .
   type Goods struct {
   	freedom.Repository
   	Cache store.EntityCache //实体缓存组件
   }
   // Get通过id获取商品实体.
   func (repo *Goods) Get(id int) (goodsEntity *entity.Goods, e error) {...}
   // Save持久化实体.
   func (repo *Goods) Save(entity *entity. Goods) error {...}
   func (repo *Goods) Finds(ids []int) (entitys []*entity.Goods, e error) {...}
   func (repo *Goods) FindsByPage(page, pageSize int, tag string) (entitys []*entity.Goods, e error) {...}
   func (repo *Goods) New(name, tag string, price, stock int) (entityGoods *entity.Goods, e error) {...}
   ```

   外部资源具体实现抽象接口

**注意：** `var _ dependency.GoodsRepo = new(Goods)` 我们用来检查是否实现了接口

## 三、六边形架构核心-依赖倒置

前面讲完基础，现在开始上大戏了！

#### 为什么说六边形架构的核心是依赖倒置？

因为六边形架构不分高低层，而分内外部，严格的将基础设施和领域模型分割开来。领域模型实现很简单，但是将领域模型与DB，Redis，MQ等基础设施连接起来却很困难。

如何连接？依赖倒置！

1. main函数中安装基础设施kafka

```go
func main() {
    app := freedom.NewApplication()
	installDatabase(app)
	installRedis(app)
	installMiddleware(app)
	// 安装领域事件的基础设施
    app.InstallDomainEventInfra(kafka.GetDomainEventInfra())]
	addrRunner := app.CreateH2CRunner(conf.Get().App.Other["listen_addr"].(string))
	app.Run(addrRunner, *conf.Get().App)
}
```

安装kafka

1. 实体中抽象领域事件接口

```go
type Entity interface {
	DomainEvent(eventName string, object interface{}, header ...map[string]string)
    Identity() string
    GetWorker() Worker
	SetProducer(string)
    Marshal() []byte
}
type DomainEventInfra interface {
	DomainEvent(producer, topic string, data []byte, worker Worker, header ...map[string]string)
}
```

抽象领域事件接口

1. kafka实现领域事件接口

```go
// DomainEvent . 
func (p *ProducerImpl) DomainEvent(producer, topic string, data []byte, worker freedom.Worker, header ...map[string]string){
    msg ：= p.NewMsg(topic, data, producer)
    if len(header)  > 0{
        msg = msg.SetHeaders(header[0])
    }
    msg.SetWorker(worker).Publish()
}
```

实现领域事件接口

1. 订单实体发送领域事件

```GO
// Pay 支付
func (cmd *OrderPayCmd) Pay() error{
    if cmd.Status != entity.OrderStatusNonPayment : errors.New("未知错误")
    if cmd.userEntity.Money < cmd.TotalPrice : errors.New("余额不足")
    cmd.userEntity.AddMoney(-cmd.TotalPrice)
    cmd.Order.Pay()
    // 事务执行 修改订单状态 扣除金币
    e := cmd.tx.Execute(func() error{
        if e:= cmd.orderRepo.Save(&cmd.Order); e != nil : e
        return cmd.userRepo.Save(cmd.userEntity)
    })
    if e == nil{
        msg := dto.OrderPayMsg{
            OrderNo: 	cmd.OrderNo,
            TotalPrice: cmd.TotalPrice,
        }
        cmd.DomainEvent("order-pay", msg)
    }
    return e
}
```

发送领域事件

#### 依赖倒置的变与不变

通过第一、二小结概念的理解，观察第三小结的代码。

Kafka是个优秀的消息队列中间件，它虽然很好，但只是基础设施，不是系统的核心部分，也许不久的某一天我们就会把它替换掉。亦或是替换掉别的中间件~

如果没有依赖倒置怎么办？
 修改业务代码？将所有用到过kafka的地方全部重新写一遍？下次有变化继续写？程序员听了想打人！

有了依赖倒置怎么办？

1. 新的中间件只需要实现领域事件接口。
2. 在main中重新安装。

这就是依赖倒置的魅力，没有什么是不变的，重要的是将领域模型与基础设施解耦开来。这样替换只需要重写领域事件，让领域模型保持相对稳定，不会随着基础设施的变化而被动变化。

## 四、品一品

细品以上两种代码，第二种实现方式中，领域模型没有像原来一样直接依赖外部资源，而是将依赖关系“倒置”过来，让基础设施去依赖由领域模型定义好的接口。

回前言所问，为什么要用一篇文章来解释依赖倒置，这就是六边形的核心，外部依赖内部，内部倒置基础设施---freedom！

**总结一下：**

常用的实现方式是基础设施有自己的接口，领域模型依赖基础设施提供的接口，比如基础设施有自己的接口，领域模型依赖基础设施的接口，这样直接依赖的实现方式。

但是按照依赖倒置的原则，接口的所有权是被倒置的，表现在于接口是被领域模型的，领域模型拥有接口的所有权，基础设施实现接口。这样基础设施的改动不会影响领域模型，领域模型的复用不会依赖基础设施。

**1.依赖于构建出来的抽象，而不是具体类。**
**2.依赖倒置的关键是接口所有权的倒置。**

# 另一篇文章

依赖倒置原则有以下几个特点：

- 依赖抽象，而不是依赖实现，
- 抽象不应该依赖细节，细节应该依赖抽象
- 高层模块不能依赖低层模块，二者都应该依赖抽象，除此之外，抽象不依赖具体实现细节，具体实现细节依赖抽象。

定义解读：

- 针对接口编程，而不是针对实现编程
- 尽量不要从具体的类派生，而是以继承抽象类或实现接口来实现
- 关于高层模块与低层模块的划分可以按照决策能力的高低进行划分。业务层自然就处于上层模块，逻辑层和数据层自然归为低层。

## 依赖倒置原则

Person中使用接口Drive来定义交通工具，出行依赖的是交通工具接口，而不是具体的交通工具。
这样，可以自由选择交通工具，只要交通工具实现该接口，并将其传递给Person类中的Drive接口。
好处：可扩展性好，以后添加其他交通工具不影响代码实现，即抽象不依赖于细节。

```go
type Drive interface {
	drive()
}

type Bike struct {}
func (b *Bike) drive() {
	fmt.Println("drive by Bike!")
}

type Car struct {}
func (c *Car) drive() {
	fmt.Println("drive by Car!")
}


// Person类
type Person struct {
	drive Drive
}

func NewPerson() *Person {
	return &Person{
		drive:new(Bike),
	}
}
func (p *Person) DriveTool() {
	p.drive.drive()
}
```

## 依赖注入

依赖注入和控制反转恰恰相反，它是一种具体的编码技巧。我们不通过 new 的方式在类内部创建依赖类的对象，而是将依赖的类对象在外部创建好之后，通过构造函数、函数参数等方式传递（或注入）给类来使用。

```go
func NewPersonx(drive Drive) *Person {
	return &Person{
		drive:drive,
	}
}
```

## 控制反转

反转是指：在没有使用框架之前，程序员自己控制整个程序的执行，当使用框架之后，整个程序的执行可以通过框架来控制； 框架提供了可扩展的代码骨架，用来组装对象/管理整个执行流程，程序只需关注扩展点，就可以利用框架来驱动整个程序流程的执行。
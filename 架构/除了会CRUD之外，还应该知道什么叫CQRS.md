# 除了会CRUD之外，还应该知道什么叫CQRS

爱编程的太阳 2019-06-24 15:37:48

今天主要跟大家分享一下什么是 CQRS，以及在项目中如何去使用。

## **CRUD系统**

我们平常最熟悉的就是三层架构，通常都是通过数据访问层来修改或者查询数据，一般修改和查询使用的是相同的实体。然后通过业务层来处理业务逻辑，将处理结果封装成DTO对象返回给控制层，再通过前端渲染。反之亦然。

![程序员除了会CRUD之外，还应该知道什么叫CQRS](http://p1.pstatp.com/large/pgc-image/a0dbe1e3ac6e4585bbf18491dd19d79d)



这里基本上是围绕关系数据库构建而成的“创建、读取、更新、删除”系统（即CRUD系统），此类系统在一些业务逻辑简单的项目中可能没有什么问题，但是随着系统逻辑变得复杂，用户增多，这种设计就会出现一些性能问题。

我们经常用到的解决方案就是对数据库进行读写分离。让主数据库处理事务性的增、删、改操作，让从数据库处理查询操作，然后主从数据库之间进行同步。但是这只是从DB角度处理了读写分离，从业务或者系统层面上来说，读和写的逻辑仍然是存放在一起的，他们都是操作同一个实体对象。

这时候，CQRS 就该登场了。

## **CQRS系统**

简单的说，CQRS（Command Query Responsibility Segration）就是一个系统，从架构上把 CRUD 系统拆分为两部分：命令（Command）处理和查询（Query）处理。其中命令处理包括增、删、改。

![程序员除了会CRUD之外，还应该知道什么叫CQRS](http://p1.pstatp.com/large/pgc-image/22928e80487d4a52b23131294745ab4f)



然后命令与查询两边可以用不同的架构实现，以实现CQ两端（即Command Side，简称C端；Query Side，简称Q端）的分别优化。两边所涉及到的实体对象也可以不同，从而继续演变成下面这样。

![程序员除了会CRUD之外，还应该知道什么叫CQRS](http://p3.pstatp.com/large/pgc-image/6d2073b48f7b42ef90d647ec22a2001a)



当然了，CQRS 作为一个读写分离思想的架构，在数据存储方面，也没有做过多的约束。所以 CQRS可以有不同层次的实现。

## **CQRS 实现方式**

**CQRS 可以有两种实现方式。**

1）CQ 两端数据库共享，只是在上层代码上分离。这样做的好处是可以让我们的代码读写分离，更容易维护，而且不存在 CQ 两端的数据一致性问题，因为是共享一个数据库的。这种架构是非常实用的（也就是我上面画的那种）。

2）CQ 两端不仅代码分离，数据库也分离，然后Q数数据由C端同步过来。同步方式有两种：同步或异步，如果需要 CQ 两端的强一致性，则需要用同步；如果能接受 CQ 两端数据的最终一致性，则可以使用异步。C端可以采用Event Sourcing（简称ES）模式，所有C端的最新数据全部用 Domain Event 表达即可；而要查询显示用的数据，则从Q端的 ReadDB（关系型数据库）查询即可。

## **CQRS 的简单实现**

说了这么多，该怎么实现呢？我们以上面提到的第一种方式为例：代码层面实现分离，数据库共享。这种方式在企业里也非常实用。

首先有几个概念需要介绍一下，CQRS 模式中，首先需要有 Command，这个 Command 命令会对应一个实体和一个命令的执行类。那整个系统中肯定有很多不同的 Command，那么还需要一个 CommandBus 来做命令的分发处理。

可能大家觉得比较抽象，我来写几行示例代码，一看就明白了。假设有个订单模块，我要新增一个订单信息。那么根据上文的分析，需要有个新增命令以及对应的订单实体（并不一定和数据库的订单实体完全对应）。首先先创建一个命令接口（绑定命令对应的实体），接口内部有个该命令的处理方法。

```
public interface Command<T> {
 Object execute(T commandModel);
}
```

OK，接下来我们可以创建订单的新增命令了。

```
@Component
public class CreateOrderCommand implements Command<CreateOrderModel> {
 @Override
 public Object execute(CreateOrderModel model) {
 // 具体的逻辑
 }
}
```

到这里，我们写好了具体的创建订单命令的逻辑，那么该命令需要放到 CommandBus 中去执行，所以我们要写这个 CommandBus。

```
@Component
public class CommandBus {
 public <T> Object dispatch(Command<T> cmd, T model) {
 return cmd.excute(model);
 }
}
```

可能大家会看着有点晕，甚至有点绕，没关系，我解释一下：这个 dispatch 方法就相当于分发执行，内部根据传入的具体 Command 以及对应的 model，去执行该 Command 实现的逻辑。

好了，那我们在熟悉的 Controller 层该如何去调用呢？很简单，如下：

```
@RestController
@RequestMapping(value = "/order")
public class OrderController {
 @Resource
 private GetOrderInfoService getOrderInfoService;
 @Resource
 private CreateOrderCommand createOrderCommand;
 @Resource
 private CommandBus commandBus;
 @PostMapping(value = "/getInfo")
 public Object getOrderInfo(GetOrderInfoModel model) {
 return getOrderInfoService.getOrderInfos(model);
 }
 @PostMapping(value = "/creat")
 public Object createOrderInfo(CreateOrderModel model) {
 return commandBus.dispatch(createOrderCommand, model);
 }
}
```

我还写了一个获取订单信息的接口，大家有没有发现，查询和插入是不同的方式，插入走的是 CommandBus 分发到 CreateOrderCommand 去执行，而查询则是直接走 service 层去查。这就是 CQRS 模式。

当然了，当命令越来越多的时候，也可以将 CommandBus 抽象出接口，可以根据业务需求，实现多个不同的 CommandBus 来分发命令。

除此之外，CQRS 还可以用在任务调度模块中，不同的任务可以包含不同的 Command，实际中运用是非常广泛的。

## **总结**

CQRS 是一种思想很简单清晰的设计模式，他通过在业务上分离操作和查询来使得系统具有更好的可扩展性及性能，使得能够对系统的不同部分进行扩展和优化。在 CQRS 中，所有的涉及到对 DB 的操作都是通过发送 Command，然后特定的 Command 触发对应事件来完成操作，也可以做成异步的，主要看业务上的需求了。

CQRS 虽然在思想上简单，但是实现上相对来说复杂些，也涉及到 DDD 的一些概念了，当然了，这篇文章主要是介绍以及演示 CQRS 模式的基本实践，更多知识需要大家再深入的去学习。
# RabbitMQ 总结

RabbitMQ是一套开源（MPL）的消息队列服务软件，是由 LShift 提供的一个 Advanced Message Queuing Protocol (AMQP) 的开源实现，由以高性能、健壮以及可伸缩性出名的 Erlang 写成。

RabbitMQ是基于AMQP协议的，通过使用通用协议就可以做到在不同语言之间传递。

## 一

AMQP协议

**核心概念**

- server：又称broker，接受客户端连接，实现AMQP实体服务。
- connection：连接和具体broker网络连接。
- channel：网络信道，几乎所有操作都在channel中进行，channel是消息读写的通道。客户端可以建立多个channel，每个channel表示一个会话任务。
- message：消息，服务器和应用程序之间传递的数据，由properties和body组成。properties可以对消息进行修饰，比如消息的优先级，延迟等高级特性；body是消息实体内容。
- Virtual host：虚拟主机，用于逻辑隔离，最上层消息的路由。一个Virtual host可以若干个Exchange和Queue，同一个Virtual host不能有同名的Exchange或Queue。
- Exchange：交换机，接受消息，根据路由键转发消息到绑定的队列上。
- banding：Exchange和Queue之间的虚拟连接，binding中可以包括routing key
- routing key：一个路由规则，虚拟机根据他来确定如何路由 一条消息。
- Queue：消息队列，用来存放消息的队列。

**Exchange**

![非常强悍的 RabbitMQ 总结，写得真好](RabbitMQ 总结.assets/S1ySQEi24TOoCQ)

交换机的类型，direct、topic、fanout、headers，durability（是否需要持久化true需要）auto delete当最后一个绑定Exchange上的队列被删除Exchange也删除。

- **Direct Exchange,**所有发送到Direct Exchange的消息被转发到RouteKey 中指定的Queue,Direct Exchange可以使用默认的默认的Exchange （default Exchange），默认的Exchange会绑定所有的队列，所以Direct可以直接使用Queue名（作为routing key ）绑定。或者消费者和生产者的routing key完全匹配。
- **Toptic Exchange,**是指发送到Topic Exchange的消息被转发到所有关心的Routing key中指定topic的Queue上。Exchange 将routing key和某Topic进行模糊匹配，此时队列需要绑定一个topic。所谓模糊匹配就是可以使用通配符，“#”可以匹配一个或多个词，“”只匹配一个词比如“log.#”可以匹配“log.info.test” "log. "就只能匹配log.error。
- **Fanout Exchange:**不处理路由键，只需简单的将队列绑定到交换机上。发送到改交换机上的消息都会被发送到与该交换机绑定的队列上。Fanout转发是最快的。

## 二

**消息如何保证100％投递？**

**什么是生产端的可靠性投递？**

- 保证消息的成功发出
- 保证MQ节点节点的成功接收
- 发送端MQ节点（broker）收到消息确认应答
- 完善消息进行补偿机制

可靠性投递保障方案

消息落库，对消息进行打标

![非常强悍的 RabbitMQ 总结，写得真好](RabbitMQ 总结.assets/S1ySQG87p1wG5W)

消息的延迟投递

![非常强悍的 RabbitMQ 总结，写得真好](RabbitMQ 总结.assets/S1ySQsSAiu4cvo)

在高并发场景下，每次进行db的操作都是每场消耗性能的。我们使用延迟队列来减少一次数据库的操作。

**消息幂等性**

我对一个动作进行操作，我们肯能要执行100次1000次，对于这1000次执行的结果都必须一样的。比如单线程方式下执行update count-1的操作执行一千次结果都是一样的，所以这个更新操作就是一个幂等的，如果是在并发不做线程安全的处理的情况下update一千次操作结果可能就不是一样的，所以并发情况下的update操作就不是一个幂等的操作。对应到消息队列上来，就是我们即使受到了多条一样的消息，也和消费一条消息效果是一样的。

**高并发的情况下如何避免消息重复消费？**

\1. 唯一id+加指纹码，利用数据库主键去重。

优点：实现简单

缺点：高并发下有数据写入瓶颈。

\2. 利用Redis的原子性来实习。

使用Redis进行幂等是需要考虑的问题

- 是否进行数据库落库，落库后数据和缓存如何做到保证幂等（Redis 和数据库如何同时成功同时失败）？
- 如果不进行落库，都放在Redis中如何这是Redis和数据库的同步策略？还有放在缓存中就能百分之百的成功吗？

**confirm 确认消息、Return返回消息**

理解confirm消息确认机制

- 消息的确认，指生产者收到投递消息后，如果Broker收到消息就会给我们 的生产者一个应答，生产者接受应答来确认broker是否收到消息。

如何实现confirm确认消息。

- 在Channel上开启确认模式：channel.confirmSelect
- 在channel上添加监听：addConfirmListener，监听成功和失败的结果，具体结果对消息进行重新发送或者记录日志。

return消息机制

Return消息机制处理一些不可路由的消息，我们的生产者通过指定一个Exchange和Routinkey，把消息送达到某一个队列中去，然后我们消费者监听队列进行消费处理！

在某些情况下，如果我们在发送消息的时候当Exchange不存在或者指定的路由key路由找不到，这个时候如果我们需要监听这种不可到达的消息，就要使用Return Listener！

Mandatory 设置为true则会监听器会接受到路由不可达的消息，然后处理。如果设置为false，broker将会自动删除该消息。

**消费端自定义监听**

消费端限流

假设我们有个场景，首先，我们有个rabbitMQ服务器上有上万条消息未消费，然后我们随便打开一个消费者客户端，会出现：巨量的消息瞬间推送过来，但是我们的消费端无法同时处理这么多数据。

这时就会导致你的服务崩溃。其他情况也会出现问题，比如你的生产者与消费者能力不匹配，在高并发的情况下生产端产生大量消息，消费端无法消费那么多消息。

- rabbitMQ提供了一种qos（服务质量保证）的功能，即非自动确认消息的前提下，如果有一定数目的消息（通过consumer或者Channel设置qos）未被确认，不进行新的消费。

```
void basicQOS(unit prefetchSize,ushort prefetchCount,Boolean global)方法。
```

- prefetchSize:0 单条消息的大小限制。0就是不限制，一般都是不限制。
- prefetchCount: 设置一个固定的值，告诉rabbitMQ不要同时给一个消费者推送多余N个消息，即一旦有N个消息还没有ack，则consumer将block掉，直到有消息ack
- global：truefalse 是否将上面的设置用于channel，也是就是说上面设置的限制是用于channel级别的还是consumer的级别的。

消费端ack与重回队列

- 消费端进行消费的时候，如果由于业务异常我们可以进行日志的记录，然后进行补偿！（也可以加上最大努力次数的尝试）
- 如果由于服务器宕机等严重问题，那我们就需要手动进行ack保证消费端的消费成功！

消息重回队列

- 重回队列就是为了对没有处理成功的消息，把消息重新投递给broker！
- 实际应用中一般都不开启重回队列。

**TTL队列/消息**

```
TTL time to live 生存时间。
```

- 支持消息的过期时间，在消息发送时可以指定。
- 支持队列过期时间，在消息入队列开始计算时间，只要超过了队列的超时时间配置，那么消息就会自动的清除。

**死信队列**

```
死信队列：DLX，Dead-Letter-Exchange
```

利用DLX，当消息在一个队列中变成死信（dead message，就是没有任何消费者消费）之后，他能被重新publish到另一个Exchange，这个Exchange就是DLX。

消息变为死信的几种情况：

1. 消息被拒绝（basic.reject/basic.nack）同时requeue=false（不重回队列）
2. TTL过期
3. 队列达到最大长度

> DLX也是一个正常的Exchange，和一般的Exchange没有任何的区别，他能在任何的队列上被指定，实际上就是设置某个队列的属性。
>
> 当这个队列出现死信的时候，RabbitMQ就会自动将这条消息重新发布到Exchange上去，进而被路由到另一个队列。可以监听这个队列中的消息作相应的处理，这个特性可以弥补rabbitMQ以前支持的immediate参数的功能。

死信队列的设置

- 设置Exchange和Queue，然后进行绑定

Exchange: dlx.exchange(自定义的名字)

queue: dlx.queue（自定义的名字）

routingkey: #（#表示任何routingkey出现死信都会被路由过来）

然后正常的声明交换机、队列、绑定，只是我们在队列上加上一个参数：

arguments.put("x-dead-letter-exchange","dlx.exchange");

## 三

RabbitMQ集群模式

1. **主备模式**：实现rabbitMQ高可用集群，一般在并发量和数据不大的情况下，这种模式好用简单。又称warren模式。（区别于主从模式，主从模式主节点提供写操作，从节点提供读操作，主备模式从节点不提供任何读写操作，只做备份）如果主节点宕机备份从节点会自动切换成主节点，提供服务。
2. **集****群模式**：经典方式就是Mirror模式，保证100%数据不丢失，实现起来也是比较简单。

- 镜像队列，是rabbitMQ数据高可用的解决方案，主要是实现数据同步，一般来说是由2-3节点实现数据同步，（对于100%消息可靠性解决方案一般是3个节点）

![非常强悍的 RabbitMQ 总结，写得真好](RabbitMQ 总结.assets/S1ySQtGFcu2wBe)

**多活模式**：这种模式也是实现异地数据复制的主流模式，因为shovel模式配置相对复杂，所以一般来说实现异地集群都是使用这种双活，多活的模式，这种模式需要依赖rabbitMQ的federation插件，可以实现持续可靠的AMQP数据。

rabbitMQ部署架构采用双中心模式（多中心）在两套（或多套）数据中心个部署一套rabbitMQ集群，各中心的rabbitMQ服务需要为提供正常的消息业务外，中心之间还需要实现部分队列消息共享。

多活架构如下：

![非常强悍的 RabbitMQ 总结，写得真好](RabbitMQ 总结.assets/S1ySQto9c3fotu)

> federation插件是一个不需要构建Cluster，而在Brokers之间传输消息的高性能插件，federation可以在brokers或者cluster之间传输消息，连接的双方可以使用不同的users或者virtual host双方也可以使用不同版本的erlang或者rabbitMQ版本。federation插件可以使用AMQP协议作为通讯协议，可以接受不连续的传输。

![非常强悍的 RabbitMQ 总结，写得真好](RabbitMQ 总结.assets/S1ySQujBLaPg5l)

Federation Exchanges,可以看成Downstream从Upstream主动拉取消息，但并不是拉取所有消息，必须是在Downstream上已经明确定义Bindings关系的

Exchange,也就是有实际的物理Queue来接收消息，才会从Upstream拉取消息到Downstream。

使用AMQP协议实施代理间通信，Downstream 会将绑定关系组合在一起， 绑定/解除绑定命令将发送到Upstream交换机。

因此，Federation Exchange只接收具有订阅的消息。

> HAProxy是一款提供高可用性、负载均衡以及基于TCP (第四层)和HTTP
>
> (第七层)应用的代理软件,支持虚拟主机，它是免费、快速并且可靠的一种解决方案。
>
> HAProxy特别适用于那些负载特大的web站点，这些站点通常又需要会话保持或七层处理。HAProxy运行在时下的硬件上，完全可以支持数以万计的并发连接。
>
> 并且它的运行模式使得它可以很简单安全的整合进您当前的架构中同时可以保护你的web服务器不被暴露到网络上。

**HAProxy性能为何这么好？**

1. 单进程、事件驱动模型显著降低了上下文切换的开销及内存占用；
2. 在任何可用的情况下，单缓冲(single buffering)机制能以不复制任何数据的方式完成读写操作，这会节约大量的CPU时钟周期及内存带宽；
3. 借助于Linux 2.6 (>= 2.6.27.19). 上的splice系统调用，HAProxy可以实现零复制转发(Zero-copy forwarding),在Linux 3.5及以上的OS中还可以实现心零复制启动(zero-starting)；
4. 内存分配器在固定大小的内存池中可实现即时内存分配，这能够显著减少创建一个会话的时长；
5. 树型存储:侧重于使用作者多年前开发的弹性二叉树，实现了以O(log(N))的低开销来保持计时器命令、保持运行队列命令及管理轮询及最少连接队列

**keepAlive**

> KeepAlived软件主要是通过VRRP协议实现高可用功能的。VRRP是
>
> Virtual Router RedundancyProtocol(虚拟路由器冗余协议)的缩写,
>
> VRRP出现的目的就是为了解决静态路由单点故障问题的，它能够保证当
>
> 个别节点宕机时，整个网络可以不间断地运行所以，Keepalived - -方面
>
> 具有配置管理LVS的功能，同时还具有对LVS下面节点进行健康检查的功
>
> 能，另一方面也可实现系统网络服务的高可用功能。

keepAlive的作用

1. 管理LVS负载均衡软件
2. 实现LVS集群节点的健康检查中
3. 作为系统网络服务的高可用性(failover)

**Keepalived如何实现高可用？**

Keepalived高可用服务对之间的故障切换转移，是通过VRRP (Virtual Router

Redundancy Protocol ,虚拟路由器冗余协议)来实现的。

在Keepalived服务正常工作时，主Master节点会不断地向备节点发送( 多播的方式)心跳消息，用以告诉备Backup节点自己还活看，当主Master节点发生故障时，就无法发送心跳消息，备节点也就因此无法继续检测到来自主Master节点的心跳了，于是调用自身的接管程序，接管主Master节点的IP资源及服务。

而当主Master节点恢复时备Backup节点又会释放主节点故障时自身接管的IP资源及服务，恢复到原来的备用角色。
# RabbitMQ 之消息的可靠性投递

默认情况下，消息发送端发送消息给 RabbitMQ 后，RabbitMQ 是不会返回任何信息的。 那么我们怎么知道消息是中途丢失了还是到达了 broker 呢？

RabbitMQ 提供了两种确认消息是否投递成功的方法

- 设置 channel 为 transaction 模式，通过 AMQP 事务机制实现，这也是 AMQP 协议层面提供的解决方案
- 设置 channel 为 confirm 模式，这是 RabbitMQ 提供的解决方案

> 两种模式不能共存

## channel 的 transaction 模式

RabbitMQ 中与事务机制有关的方法有三个

- txSelect：用于将当前 channel 设置成 transaction 模式
- txCommit：用于提交事务
- txRollback：用于回滚事务

在通过 txSelect 开启事务之后，我们便可以发布消息给broker代理服务器了，如果txCommit提交成功了，则消息一定到达了broker了，如果在 txCommit 执行之前 broker 异常崩溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过 txRollback 回滚事务了

事务确实能够解决 producer 与 broker 之间消息确认的问题，只有消息成功被 broker 接收，事务提交才能成功，否则我们便可以在捕获异常进行事务回滚操作，同时进行消息重发，但是使用事务机制的话会降低RabbitMQ的性能。

RabbitMQ 提供了一个更好的方案，使用 channel 信道的 confirm 模式。

## channel 的 confirm 模式

生产者通过调用 channel 的 confirmSelect 方法将 channel 设置为 confirm 模式. 该模式下，所有在该信道上发布的消息都会被分派一个唯一的ID(从1开始)，当消息被投递到所有匹配的队列后，broker 就会发送一个(包含消息的唯一 ID 的)确认给发送端， 如果 RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条nack消息，发送端的 Confirm Listener 会去监听应答



![img](https://user-gold-cdn.xitu.io/2019/9/2/16cf208bac732914?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



broker回传给发送端的确认消息中 deliver-tag 域包含了确认消息的ID，此外 broker 也可以设置 basic.ack 的 multiple 域，表示到这个ID之前的所有消息都已经得到了处理

confirm模式最大的好处在于他是异步的，生产者可以在等信道返回的同时继续发送下一条消息。

### Confirm 的三种使用方式

Confirm 的三种实现方式：

方式一：channel.waitForConfirms()普通发送方确认模式；

方式二：channel.waitForConfirmsOrDie()批量确认模式；

方式三：channel.addConfirmListener()异步监听发送方确认模式；

- 普通确认：每发送一条消息后，调用channel.waitForConfirms()方法，同步等待服务器端confirm。实际上是一种串行confirm了。
- 批量确认：每发送一批消息后，调用channel.waitForConfirms()方法，同步等待服务器端confirm
- 异步确认：为channel添加一个监听器，rabbitmq 会回调这个方法，示例代码如下

```
// 添加一个确认监听
channel.addConfirmListener(new ConfirmListener() {
    //消息失败处理
    @Override
    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
        //deliveryTag；唯一消息标签
        //multiple：是否批量
        System.err.println("-------no ack!-----------");
    }
    //消息成功处理
    @Override
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {
        System.err.println("-------ack!-----------");
    }
});
复制代码
```

批量 confirm 模式 和 异步confirm模式效率更高些，但是批量confirm模式下，如果 rabbitMq 返回失败，那么需要重新发送这一批消息，建议最好还是选择**异步confirm模式**

### 超时

RabbitMQ 的响应可能会超时，超时可能是消息没有到达 mq ，也有可能是网络延迟导致的。

对于超时，一般的做法是重新投递，但如果超时是因为网络延迟造成的，那么重新投递就变成了重复投递，重复投递可能导致消费端**重复消费**。但重复投递又无法完全避免，因此消费端需要防止**重复消费**。

### 延伸

上面说到“**当消息被投递到所有匹配的队列后，broker 就会发送一个(包含消息的唯一 ID 的)确认给发送端**”，万一发送确认后， rabbitMq 崩溃了，消息队列中的消息就都没了，这时候发送端还以为消息还在队列中。

为了防止这种情况的发送，rabbitMq 需要对队列和消息进行持久化。

当消息和队列开启持久化之后，**确认信息会等到消息写入磁盘之后再发出**

# 消息消费端

为了保证消息从队列可靠地到达消费者，RabbitMQ提供消息确认机制(message acknowledgment)。消费者在声明队列时，可以指定noAck参数，当noAck=false时，RabbitMQ会等待消费者显式发回ack信号后才从内存(和磁盘，如果是持久化消息的话)中移去消息。否则，RabbitMQ会在队列中消息被消费后立即删除它。

采用消息确认机制后，只要令noAck=false，消费者就有足够的时间处理消息(任务)，不用担心处理消息过程中消费者进程挂掉后消息丢失的问题，因为RabbitMQ会一直持有消息直到消费者显式调用basicAck为止。

当noAck=false时，对于RabbitMQ服务器端而言，队列中的消息分成了两部分：一部分是等待投递给消费者的消息；一部分是已经投递给消费者，但是还没有收到消费者ack信号的消息。如果服务器端一直没有收到消费者的ack信号，并且消费此消息的消费者已经断开连接，则服务器端会安排该消息重新进入队列，等待投递给下一个消费者（也可能还是原来的那个消费者）。

RabbitMQ不会为未ack的消息设置超时时间，它判断此消息是否需要重新投递给消费者的唯一依据是消费该消息的消费者连接是否已经断开。这么设计的原因是RabbitMQ允许消费者消费一条消息的时间可以很久很久。

RabbitMQ管理平台界面上可以看到当前队列中Ready状态和Unacknowledged状态的消息数，分别对应上文中的等待投递给消费者的消
# RabbitMQ如何保证消息的可达性

## 一、RabbitMQ简介

AMQP，即Advanced Message Queuing Protocol,一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准,为面向消息的中间件设计。

RabbitMQ，是一个消息代理和队列服务器，它实现了AMQP标准协议。

分布式消息队列有很多应用场景，比如异步处理、应用解耦、流量削峰等。

#### 1、异步处理

用户注册后需要发送短信和邮件，传统做法是先将用户信息写入数据库，然后发送短信、发送邮件，都完成后返回。

如果用到消息队列，可以先将用户信息写入数据库，然后将注册信息写入消息队列，发送短信、发送邮件或者还有其他的业务逻辑都订阅此消息，完成发送。

#### 2、应用解耦

还是上面的例子，如果在一个大型分布式网站中，用户系统、短信系统、邮件系统可能都是独立的系统服务。

这时候，在用户注册成功后，你可以通过RPC远程调用不同的服务接口，但更好的做法还是通过消息队列，订阅自己感兴趣的数据，日后就算增加或者删减功能，主业务都不用变动。

#### 3、流量削峰

一般在秒杀或者团购活动中，流量激增，应用面临压力过大。可以在应用前端加入消息队列，通过设置队列最大长度来限制活动人数。这时候，后端服务器就可以游刃有余的处理数据了。

## 二、消息通信

在AMQP协议中，有几个基本概念，我们必须先搞明白。

#### 1、Virtual host

虚拟主机，每一个虚拟主机中包含所有的AMQP基本组件，用户、队里、交换器等都是在虚拟主机里面创建。典型的用法是，如果公司的多个产品只想用一个服务器，就可以把他们划分到不同的虚拟主机中，里面的任何信息都是独立存在，互不干扰。

#### 2、Connection

连接，应用程序和服务器之间的TCP连接。

#### 3、Channel

通道，当你的应用程序和服务器连接之后，就会创建TCP连接。一旦打开了TCP连接，就可以创建一个Channel通道，所以说Channel通道是一个TCP连接的内部逻辑单元。 这是因为，创建和销毁TCP连接是比较昂贵的开销，每一次访问都建立新的TCP连接的话，不仅是巨大浪费，而且还容易造成系统性能瓶颈。

#### 4、Queue

队列，所有的消息最终都会被送到这里，等待着被感兴趣的人取走。

#### 5、Exchange

交换器，消息到达服务的第一站就是交换器，然后根据分发规则，匹配路由键，将消息放到对应队列中。值得注意的是，交换器的类型不止一种。

- Direct 直连交换器，只有在消息中的路由键和绑定关系中的键一致时，交换器才把消息发到相应队列
- Fanout 广播交换器，只要消息被发送到广播交换器，它会将消息发到所有的队列
- Topic 主题交换器，根据路由键，通配规则(*和#)，将消息发到相应队列

#### 6、Binding

绑定，交换器和队列之间的绑定关系，绑定中就包含路由键，绑定信息被保存到交换器的查询表中，交换器根据它分发消息。

了解到这些组件相关概念后，我们总结一下来看看，一条消息在RabbitMQ中是如何流转的。

![img](RabbitMQ如何保证消息的可达性.assets/16e3b5320fe753c9)

## 三、持久化和发送方确认

#### 1、持久化

事实上，上图所示只是一个最基本的消息流转过程，交换器和队列这些组件还有一个比较重要的属性：持久化。

默认情况下，重启RabbitMQ服务器之后，我们创建的交换器和队列都会消失不见，当然了，如果里面还有未来得及消费的数据，也将难于幸免。 持久化交换器和队列，为的是在AMQP服务器重启之后，重新创建它们并绑定关系，在RabbitMQ中，设置durable属性为true即可。

不过，除了这些还不够。虽然保证了交换器和队列是安全的，但那些还未来得及消费的数据就变得岌岌可危。所以，我们还要设置消息的投递模式为持久的。

这样，如果RabbitMQ服务器重启的话，我们的策略和相关数据才会确保无忧。所以，我们说能从AMQP服务器崩溃中恢复的消息，称之为持久化消息。那么，它必须保证以下三点：

- 设置投递模式为持久的
- 交换器为持久的
- 队列为持久的

#### 2、发送方确认

到目前为止，我们已经保证了消息的安全性。但是，还有另外一个问题。由于发布操作是不返回任何信息给生产者的，我们怎么知道服务器是否正确接收消息并持久化到硬盘上了呢？

为此，我们可以将通道设置为事务模式。事务是AMQP标准中的一部分，但RabbitMQ有更好的做法，那就是发送方确认模式，publisher confirm。如果设置了confirm模式，发布的消息会被分配一个唯一的ID号，等消息被投递给匹配的队列后，通道会发送一个发送方确认模式给生产者（包含消息的唯一ID）。

## 四、与Spring整合实例

废话了这么多，只是为了下面的代码部分做下铺垫。毕竟，了解到上面内容之后，代码其实已经快要跃然纸上了。

#### 1、配置文件

配置文件中我们首先要声明RabbitMQ服务器的信息，IP地址、端口号、用户名密码等，但尤为重要的是，设置发布确认模式。

```
<bean id="rabbitConnectionFactory"
	class="org.springframework.amqp.rabbit.connection.CachingConnectionFactory">
	<constructor-arg value="127.0.0.1"/>
	<property name="username" value="shiqizhen"/>
	<property name="password" value="shiqizhen"/>
	<property name="port" value="5672"></property>
	<property name="virtualHost" value="shiqizhen"></property>
	<property name="publisherConfirms" value="true"></property>
	<property name="publisherReturns" value="true"></property>
</bean>
复制代码
```

接着，还要声明交换器和队列，记得它们是持久化的哦，durable为true。

```
<rabbit:admin connection-factory="rabbitConnectionFactory"/>
    
//队列的名字、持久化、不要自动删除、不是独享队列
<rabbit:queue name="userInfoQueue" durable="true" auto-delete="false" exclusive="false"/>
//交换器，类型为direct。并绑定交换器和队列的关系，路由键为10086
<rabbit:direct-exchange name="user-exchange" durable="true" auto-delete="false">
	<rabbit:bindings>
		<rabbit:binding queue="userInfoQueue" key="10086"/>
	</rabbit:bindings>
</rabbit:direct-exchange>
复制代码
```

最后，配置消费者和消息模板

```
//配置消费者 ref为bean的引用 queues指明了消费者与队列的关系
//重要的是acknowledge 确认模式为手动确认
<rabbit:listener-container connection-factory="rabbitConnectionFactory" acknowledge="manual">
        <rabbit:listener ref="consumerListener" queues="userInfoQueue" method="onMessage" />
</rabbit:listener-container>

//配置Spring RabbitMQ消息模板 
<bean id="rabbitTemplate" class="org.springframework.amqp.rabbit.core.RabbitTemplate">
	<constructor-arg ref="rabbitConnectionFactory"></constructor-arg>
	<property name="confirmCallback" ref="publisherConfirm"></property>
	<property name="returnCallback" ref="returnMsgCallBack"></property>
	<property name="mandatory" value="true"></property>
</bean>
复制代码
```

### 2、生产者

上面我们声明了rabbitTemplate，直接用它的send方法发送消息即可。不过它有几个参数必须先要了解下。

- exchange 交换器名称，消息发到哪个交换器上
- routingKey 路由键，交换器怎样分发消息到对应队列
- Message 消息体对象，它包含消息的主体和消息属性。消息属性包含很多附属信息，比如消息内容类型、消息ID、用户ID等。
- CorrelationData 消息相关数据，实际它只有一个ID的属性。不过很重要，在发布方确认的回调方法里，会带有这个参数。我们可以根据它很直观的看到哪条消息发送成功或失败。

```
@Controller
public class IndexController {
	
	@Autowired
	RabbitTemplate rabbitTemplate;

	@RequestMapping("/send_msg")
	@ResponseBody
	public User send_msg() {	
		String exchange = "user-exchange";
		String routingKey = "10086";
		
		User user = new User();
		String id = IdUtil.getId();
		user.setUid(id);
		user.setUsername("小小沙弥");
		user.setPassword("1234");
		user.setCreatetime(DateUtil.getDateTime(new Date()));
		
		CorrelationData correlation = new CorrelationData(id);
		Message message = new Message(JSONObject.toJSONBytes(user, SerializerFeature.WriteNullStringAsEmpty), new MessageProperties());
		logger.info("已发送消息到RabbitMQ服务器:{}",JSONObject.toJSONString(user));
		rabbitTemplate.send(exchange, routingKey,message,correlation);	
		return user;
	}
}
复制代码
```

#### 3、消费者

消费者就是上面我们配置的listener ref引用的Bean。还记得我们把确认模式设置了手动确认，所以在消费者端有个很重要的动作，就是确认消息。

- channel.basicAck(deliveryTag, false) 第一个参数是RabbitMQ内部产生的消息ID，第二个参数代表是否批量确认消息。通过这个指令我们告诉生产者端，消息已经被正确消费了，RabbitMQ就会将此消息在磁盘上删除。
- channel.basicReject(deliveryTag, false) 拒绝消息。如果消费到的消息不是我们想要的，或者处理的时候报错，我们可以将消息拒绝。但值得注意的是第二个参数。如果设置为false，说明拒绝消息并将消息从服务器上删除；如果设置为true，说明拒绝消息并将消息重新放回队列。如果你的消费者只有一个，最好不要把它设置为true，否则消息会一直重试，直到把消费者端服务器搞死。如果因为处理失败而拒绝的话，最好将消息删除，同时将消息记录到日志文件或者数据库中。

```
@Service
public class ConsumerListener implements ChannelAwareMessageListener{

	Logger logger = LoggerFactory.getLogger(this.getClass());
	
	public void onMessage(Message message, Channel channel) throws Exception {
		
		logger.info("消费者监听到RabbitMQ消息...");
		MessageProperties properties = message.getMessageProperties();
		String msg = new String(message.getBody(),"utf-8");
		logger.info("交换器：{},路由键:{}",properties.getReceivedExchange(),properties.getReceivedRoutingKey());
		logger.info("消息内容：{}",msg);	
		long deliveryTag = properties.getDeliveryTag();
		channel.basicAck(deliveryTag, false);//确认信息,false为不批量确认
		//channel.basicReject(deliveryTag, true);//true为重入队列 false为删除消息
	}
}
复制代码
```

### 4、发送方确认

我们发送消息给RabbitMQ，第一站就是交换器。RabbitMQ是否能正确接收消息，我们就靠它来反馈。这里的CorrelationData就是在生产者端设置的，我们可以将它当成消息ID，也可以直接把消息写入这里。

```
@Component
public class PublisherConfirm implements ConfirmCallback{

	Logger logger = LoggerFactory.getLogger(this.getClass());
	
	public void confirm(CorrelationData correlationData, boolean ack, String cause) {
		if (ack) {
			logger.info("消息投递成功!");
		}else {
			logger.warn("消息投递失败,原因：{}，消息ID：{}",cause,correlationData.getId());
		}
	}
}
复制代码
```

如果我们把交换器的名字写错，那么在这里，你就会得到以下信息：

```
22:57:51,635  WARN PublisherConfirm:19 - 消息投递失败,原因：
channel error; protocol method: #method<channel.close>
(reply-code=404, reply-text=NOT_FOUND - no exchange 'user-exchange_xxx' in vhost 'shiqizhen', class-id=60, method-id=40)，

消息ID：516387069669408768
  22:57:51,638 ERROR CachingConnectionFactory:1278 - Channel shutdown: 
channel error; protocol method:
 #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'user-exchange_xxx' in vhost 'shiqizhen', class-id=60, method-id=40)
复制代码
```

#### 5、返回回调

除了设置RabbitMQ的发送方确认，在Spring中还有一个publisherReturns值的我们注意。虽然我们将消息发送到了交换器，但交换器是否能正确将消息分发到对应队列，还要打个问号。如果消息无法发送到指定的队列，那么publisherReturns就会发挥作用。记住，如果想应用这个特性，需要将mandatory设置为true。

```
@Component
public class ReturnMsgCallBack implements ReturnCallback{

	Logger logger = LoggerFactory.getLogger(this.getClass());
	
	public void returnedMessage(Message message, int replyCode, 
					String replyText, String exchange, String routingKey) {
		logger.info("消息内容:{}",new String(message.getBody()));
		logger.info("回复文本:{},回复代码：{}",replyText,replyCode);
		logger.info("交换器名称:{},路由键：{}",exchange,routingKey);	
	}
}
复制代码
```

如果我们不小心写错了路由键的名字，那就会调用到这里。

```
23:24:27,813  INFO ReturnMsgCallBack:16 - 消息内容:{"createtime":"2018-11-25 23:24:24","password":"1234","role":null,"uid":"516393749815754752","username":"小小沙弥"}
23:24:27,814  INFO ReturnMsgCallBack:17 - 回复文本:NO_ROUTE,回复代码：312
23:24:27,814  INFO ReturnMsgCallBack:18 - 交换器名称:user-exchange,路由键：10086_xxx

//这里是发送方确认打印的信息 说投递到交换器成功
23:24:27,814  INFO PublisherConfirm:17 - 消息投递成功!
复制代码
```

有个问题，如同第一个例子，如果写错了路由键的名称，发送方确认会打印ack为false的异常信息，但为什么不会调用到publisherReturns呢？

如果路由键错误，说明消息压根就没有被接收到。这肯定是一个严重错误，所以RabbitMQ直接把当前通道关闭了。

```
Channel shutdown: 
channel error; protocol method:
reply-code=404, reply-text=NOT_FOUND - no exchange 'user-exchange_xxx' in vhost ...
复制代码
```

## 五、监听RabbitMQ服务器状态

如果你的RabbitMQ服务不是一个集群，那么当网络故障或其他原因导致RabbitMQ服务停掉的时候，我们怎么做呢？当然，你可以在Send方法中加入try/catch，根据catch信息返回你的状态。但有个更好的思路，可以结合使用。 在创建RabbitMQ服务连接的时候，我们要配置一个Bean，`CachingConnectionFactory`它有个方法`addConnectionListener`，我们可以利用它来监听服务器的连接状态。

```
public class RabbitMQConnectionListener implements ConnectionListener{
	public void onCreate(Connection connection) {
		System.out.println("服务器已启动...");
	}
	public void onClose(Connection connection) {
		System.out.println("服务器已关闭...");
	}
}
复制代码
```

并在合适的位置，比如Spring容器初始化方法里，加入这么一句`rabbitConnectionFactory.addConnectionListener(new RabbitMQConnectionListener());`

这样，我们就可以掌握RabbitMQ服务器的连接状态了，那么我们就可以根据此状态，在生产者方调用send方法的时候，判断此状态。如果未连接，可以先将消息保存到数据库或者缓存中。当连接到RabbitMQ，我们先把缓存的消息拿出来发送，再将此状态重置为已连接。

## 六、总结

本文简单介绍了AMQP协议标准中的相关概念，以及RabbitMQ在Spring中如何正确配置使用持久化消息、发送方模式和返回回调等机制。并在最后，介绍了在Spring中如何监听RabbitMQ的服务器连接状态。总而言之一句话，我们将要怎样使用RabbitMQ，才能保证消息不会丢失。希望本文对你使用RabbitMQ有所帮助!
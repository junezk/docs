# Python 操作 RabbitMQ 消息队列

> RabbitMQ和邮局的主要区别就是RabbitMQ接收、存储和发送的是二进制数据----消息，本篇文章给大家介绍利用Python学习RabbitMQ消息队列，对python消息队列相关知识感兴趣的朋友参考下

RabbitMQ可以当做一个消息代理，它的核心原理非常简单：即接收和发送消息，可以把它想象成一个邮局：我们把信件放入邮箱，邮递员就会把信件投递到你的收件人处，RabbitMQ就是一个邮箱、邮局、投递员功能综合体，整个过程就是：邮箱接收信件，邮局转发信件，投递员投递信件到达收件人处。

RabbitMQ和邮局的主要区别就是 RabbitMQ 接收、存储和发送的是二进制数据----消息。

## **rabbitmq基本管理命令**

一步启动Erlang node和Rabbit应用：`sudo rabbitmq-server`

在后台启动Rabbit node：`sudo rabbitmq-server -detached`

关闭整个节点（包括应用）：`sudo rabbitmqctl stop`

```bash
`add_user <UserName> <Password>``delete_user <UserName>``change_password <UserName> <NewPassword>``list_users``add_vhost <VHostPath>``delete_vhost <VHostPath>``list_vhosts``set_permissions [-p <VHostPath>] <UserName> <Regexp> <Regexp> <Regexp>``clear_permissions [-p <VHostPath>] <UserName>``list_permissions [-p <VHostPath>]``list_user_permissions <UserName>``list_queues [-p <VHostPath>] [<QueueInfoItem> ...]``list_exchanges [-p <VHostPath>] [<ExchangeInfoItem> ...]``list_bindings [-p <VHostPath>]``list_connections [<ConnectionInfoItem> ...]`
```

## **一、概念：**

Connection： 一个TCP的连接。Producer和Consumer都是通过TCP连接到RabbitMQ Server的。程序的起始处就是建立这个TCP连接。

Channels： 虚拟连接。建立在上述的TCP连接中。数据流动都是在Channel中进行的。一般情况是程序起始建立TCP连接，第二步就是建立这个Channel。

## **二、队列：**

首先建立一个Connection，然后建立Channels，在channel上建立队列

建立时指定durable参数为真，队列将持久化；指定exclusive为真，队列为临时队列，关闭consumer后该队列将不再存在，一般情况下建立临时队列并不指定队列名称，rabbitmq将随机起名，通过result.method.queue来获取队列名：

result = channel.queue_declare(exclusive=True)

result.method.queue

区别：durable是队列持久化与否，如果为真，队列将在rabbitmq服务重启后仍存在，如果为假，rabbitmq服务重启前不会消失，与consumer关闭与否无关；

而exclusive是建立临时队列，当consumer关闭后，该队列就会被删除

## **三、exchange和bind**

Exchange中durable参数指定exchange是否持久化，exchange参数指定exchange名称，type指定exchange类型。Exchange类型有direct，fanout和topic。

Bind是将exchange与queue进行关联，exchange参数和queue参数分别指定要进行bind的exchange和queue，routing_key为可选参数。

### **Exchange的三种模式：**

#### **Direct：**

任何发送到Direct Exchange的消息都会被转发到routing_key中指定的Queue

1.一般情况可以使用rabbitMQ自带的Exchange：””(该Exchange的名字为空字符串)；

2.这种模式下不需要将Exchange进行任何绑定(bind)操作；

3.消息传递时需要一个“routing_key”，可以简单的理解为要发送到的队列名字；

4.如果vhost中不存在routing_key中指定的队列名，则该消息会被抛弃。

Demo中虽然声明了一个exchange='yanfa'和queue='anheng'的bind，但是在后面发送消息时并没有使用该exchange和bind，而是采用了direct的模式，没有指定exchange，而是指定了routing_key的名称为队列名，消息将发送到指定队列。

如果一个exchange 声明为direct，并且bind中指定了routing_key,那么发送消息时需要同时指明该exchange和routing_key.

#### **Fanout:**

任何发送到Fanout Exchange的消息都会被转发到与该Exchange绑定(Binding)的所有Queue上

1.可以理解为路由表的模式

2.这种模式不需要routing_key

3.这种模式需要提前将Exchange与Queue进行绑定，一个Exchange可以绑定多个Queue，一个Queue可以同多个Exchange进行绑定。

4.如果接受到消息的Exchange没有与任何Queue绑定，则消息会被抛弃。

Demo中创建了一个将一个exchange和一个queue进行fanout类型的bind.但是发送信息时没有用到它，如果要用到它，只要在发送消息时指定该exchange的名称即可，该exchange就会将消息发送到所有和它bind的队列中。在fanout模式下，指定的routing_key是无效的 。

#### **Topic：**

任何发送到Topic Exchange的消息都会被转发到所有关心routing_key中指定话题的Queue上

1.这种模式较为复杂，简单来说，就是每个队列都有其关心的主题，所有的消息都带有一个“标题”(routing_key)，Exchange会将消息转发到所有关注主题能与routing_key模糊匹配的队列。

2.这种模式需要routing_key，也许要提前绑定Exchange与Queue。

3.在进行绑定时，要提供一个该队列关心的主题，如“#.log.#”表示该队列关心所有涉及log的消息(一个routing_key为”MQ.log.error”的消息会被转发到该队列)。

4.“#”表示0个或若干个关键字，“*”表示一个关键字。如“log.*”能与“log.warn”匹配，无法与“log.warn.timeout”匹配；但是“log.#”能与上述两者匹配。

5.同样，如果Exchange没有发现能够与routing_key匹配的Queue，则会抛弃此消息。

## **四、任务分发**

1.Rabbitmq的任务是循环分发的，如果开启两个consumer，producer发送的信息是轮流发送到两个consume的。

2.在producer端使用cha.basic_publish()来发送消息，其中body参数就是要发送的消息，properties=pika.BasicProperties(delivery_mode = 2,)启用消息持久化，可以防止RabbitMQ Server 重启或者crash引起的数据丢失。

3.在接收端使用cha.basic_consume()无限循环监听，如果设置no-ack参数为真，每次Consumer接到数据后，而不管是否处理完成，RabbitMQ Server会立即把这个Message标记为完成，然后从queue中删除了。为了保证数据不被丢失，RabbitMQ支持消息确认机制，即acknowledgments。为了保证数据能被正确处理而不仅仅是被Consumer收到，那么我们不能采用no-ack。而应该是在处理完数据后发送ack。

在处理数据后发送的ack，就是告诉RabbitMQ数据已经被接收，处理完成，RabbitMQ可以去安全的删除它了。如果Consumer退出了但是没有发送ack，那么RabbitMQ就会把这个Message发送到下一个Consumer。这样就保证了在Consumer异常退出的情况下数据也不会丢失。

这里并没有用到超时机制。RabbitMQ仅仅通过Consumer的连接中断来确认该Message并没有被正确处理。也就是说，RabbitMQ给了Consumer足够长的时间来做数据处理。

Demo的callback方法中ch.basic_ack(delivery_tag = method.delivery_tag)告诉rabbitmq消息已经正确处理。如果没有这条代码，Consumer退出时，Message会重新分发。然后RabbitMQ会占用越来越多的内存，由于RabbitMQ会长时间运行，因此这个“内存泄漏”是致命的。去调试这种错误，可以通过一下命令打印un-acked Messages：

sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged

4.公平分发：设置cha.basic_qos(prefetch_count=1)，这样RabbitMQ就会使得每个Consumer在同一个时间点最多处理一个Message。换句话说，在接收到该Consumer的ack前，他它不会将新的Message分发给它。

## **五、注意：**

生产者和消费者都应该声明建立队列，网上教程上说第二次创建如果参数和第一次不一样，那么该操作虽然成功，但是queue的属性并不会被修改。

可能因为版本问题，在我的测试中如果第二次声明建立的队列属性和第一次不完全相同，将报类似这种错406, "PRECONDITION_FAILED - parameters for queue 'anheng' in vhost '/' not equivalent"

如果是exchange第二次创建属性不同，将报这种错406, "PRECONDITION_FAILED - cannot redeclare exchange 'yanfa' in vhost '/' with different type, durable, internal or autodelete value"

如果第一次声明建立队列也出现这个错误，说明之前存在名字相同的队列且本次声明的某些属性和之前声明不同，可通过命令sudo rabbitmqctl list_queues查看当前有哪些队列。解决方法是声明建立另一名称的队列或删除原有队列，如果原有队列是非持久化的，可通过重启rabbitmq服务删除原有队列，如果原有队列是持久化的，只能删除它所在的vhost,然后再重建vhost,再设置vhost的权限（先确认该vhost中没有其他有用队列）。

```bash
`sudo` `rabbitmqctl delete_vhost /``sudo` `rabbitmqctl add_vhost /``sudo` `rabbitmqctl set_permissions -p / username ``'.*'` `'.*'` `'.*'`
```

## python 中操作 RabbitMQ

Python使用RabbitMQ需要Pika库

```bash
`sudo` `pip ``install` `pika`
```

producer.py

```python
#!/usr/bin/env python
# -*- coding: utf_ -*-
# Date: 年月日
# Author:蔚蓝行
# 博客 http://www.cnblogs.com/duanv/
import` `pika
import` `sys
#创建连接connection到localhost
con = pika.BlockingConnection(pika.ConnectionParameters(``'localhost'``))
#创建虚拟连接channel
cha = con.channel()
#创建队列anheng,durable参数为真时，队列将持久化；exclusive为真时，建立临时队列
result=cha.queue_declare(queue=``'anheng'``,durable=True,exclusive=False)
#创建名为yanfa,类型为fanout的exchange，其他类型还有direct和topic，如果指定durable为真，exchange将持久化
cha.exchange_declare(durable=False,
          ``exchange=``'yanfa'``,
          ``type``=``'direct'``,)
#绑定exchange和queue,result.method.queue获取的是队列名称
cha.queue_bind(exchange=``'yanfa'``, 
       ``queue=result.method.queue,
       ``routing_key=``''``,) 
#公平分发，使每个consumer在同一时间最多处理一个message，收到ack前，不会分配新的message
cha.basic_qos(prefetch_count=)
#发送信息到队列‘anheng'
message = ``' '``.``join``(sys.argv[:])
#消息持久化指定delivery_mode=；
cha.basic_publish(exchange=``''``,
         ``routing_key=``'anheng'``,
         ``body=message,
         ``properties=pika.BasicProperties(
          ``delivery_mode = ,
        ``))
print ``'[x] Sent %r'` `% (message,)
#关闭连接
con.close()
```

consumer.py

```python
#!/usr/bin/env python
# -*- coding: utf_ -*-
# Date: 年月日
# Author:蔚蓝行
# 博客 http://www.cnblogs.com/duanv/
import` `pika
#建立连接connection到localhost
con = pika.BlockingConnection(pika.ConnectionParameters(``'localhost'``))
#创建虚拟连接channel
cha = con.channel()
#创建队列anheng
result=cha.queue_declare(queue=``'anheng'``,durable=True)
#创建名为yanfa,类型为fanout的交换机，其他类型还有direct和topic
cha.exchange_declare(durable=False,
          ``exchange=``'yanfa'``, 
          ``type``=``'direct'``,)
#绑定exchange和queue,result.method.queue获取的是队列名称
cha.queue_bind(exchange=``'yanfa'``,
       ``queue=result.method.queue,
       ``routing_key=``''``,)
#公平分发，使每个consumer在同一时间最多处理一个message，收到ack前，不会分配新的message
cha.basic_qos(prefetch_count=)
print ``' [*] Waiting for messages. To exit press CTRL+C'
#定义回调函数
def callback(ch, method, properties, body):
  ``print ``" [x] Received %r"` `% (body,)
  ``ch.basic_ack(delivery_tag = method.delivery_tag)
cha.basic_consume(callback,
         ``queue=``'anheng'``,
         ``no_ack=False,)
cha.start_consuming()
```

**远程结果返回**

消息发送端发送消息出去后没有结果返回。如果只是单纯发送消息，当然没有问题了，但是在实际中，常常会需要接收端将收到的消息进行处理之后，返回给发送端。

处理方法描述：发送端在发送信息前，产生一个接收消息的临时队列，该队列用来接收返回的结果。其实在这里接收端、发送端的概念已经比较模糊了，因为发送端也同样要接收消息，接收端同样也要发送消息，所以这里笔者使用另外的示例来演示这一过程。

示例内容：假设有一个控制中心和一个计算节点，控制中心会将一个自然数N发送给计算节点，计算节点将N值加1后，返回给控制中心。这里用center.py模拟控制中心，compute.py模拟计算节点。

compute.py代码分析

```python
#!/usr/bin/env python
#coding=utf8
import pika
  
#连接rabbitmq服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='localhost'))
channel = connection.channel()
  
#定义队列
channel.queue_declare(queue='compute_queue')
print ' [*] Waiting for n'
  
#将n值加1
def increase(n):
  return n + 1
  
#定义接收到消息的处理方法
def request(ch, method, properties, body):
  print " [.] increase(%s)" % (body,)
  
  response = increase(int(body))
  
  #将计算结果发送回控制中心
  ch.basic_publish(exchange='',
           routing_key=properties.reply_to,
           body=str(response))
  ch.basic_ack(delivery_tag = method.delivery_tag)
  
channel.basic_qos(prefetch_count=1)
channel.basic_consume(request, queue='compute_queue')
  
channel.start_consuming()
```

计算节点的代码比较简单，值得一提的是，原来的接收方法都是直接将消息打印出来，这边进行了加一的计算，并将结果发送回控制中心。

center.py代码分析

```python
#!/usr/bin/env python
#coding=utf8
import pika
  
class Center(object):
  def __init__(self):
    self.connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
  
    self.channel = self.connection.channel()
      
    #定义接收返回消息的队列
    result = self.channel.queue_declare(exclusive=True)
    self.callback_queue = result.method.queue
  
    self.channel.basic_consume(self.on_response,
                  no_ack=True,
                  queue=self.callback_queue)
  
  #定义接收到返回消息的处理方法
  def on_response(self, ch, method, props, body):
    self.response = body
    
    
  def request(self, n):
    self.response = None
    #发送计算请求，并声明返回队列
    self.channel.basic_publish(exchange='',
                  routing_key='compute_queue',
                  properties=pika.BasicProperties(
                     reply_to = self.callback_queue,
                     ),
                  body=str(n))
    #接收返回的数据
    while self.response is None:
      self.connection.process_data_events()
    return int(self.response)
  
center = Center()
  
print " [x] Requesting increase(30)"
response = center.request(30)
print " [.] Got %r" % (response,)
```

上例代码定义了接收返回数据的队列和处理方法，并且在发送请求的时候将该队列赋值给reply_to，在计算节点代码中就是通过这个参数来获取返回队列的。

打开两个终端，一个运行代码python compute.py，另外一个终端运行center.py，如果执行成功，应该就能看到效果了。

笔者在测试的时候，出了些小问题，就是在center.py发送消息时没有指明返回队列，结果compute.py那边在计算完结果要发回数据时报错，提示routing_key不存在，再次运行也报错。用rabbitmqctl list_queues查看队列，发现compute_queue队列有1条数据，每次重新运行compute.py的时候，都会重新处理这条数据。后来使用/etc/init.d/rabbitmq-server restart重新启动下rabbitmq就ok了。

**相互关联编号correlation id**

上一遍演示了远程结果返回的示例，但是有一个没有提到，就是correlation id，这个是个什么东东呢？

假设有多个计算节点，控制中心开启多个线程，往这些计算节点发送数字，要求计算结果并返回，但是控制中心只开启了一个队列，所有线程都是从这个队列里获取消息，每个线程如何确定收到的消息就是该线程对应的呢？这个就是correlation id的用处了。correlation翻译成中文就是相互关联，也表达了这个意思。

correlation id运行原理：控制中心发送计算请求时设置correlation id，而后计算节点将计算结果，连同接收到的correlation id一起返回，这样控制中心就能通过correlation id来标识请求。其实correlation id也可以理解为请求的唯一标识码。

示例内容：控制中心开启多个线程，每个线程都发起一次计算请求，通过correlation id，每个线程都能准确收到相应的计算结果。

compute.py代码分析

和上面一篇相比，只需修改一个地方：将计算结果发送回控制中心时，增加参数correlation_id的设定，该参数的值其实是从控制中心发送过来的，这里只是再次发送回去。代码如下：

```python
#!/usr/bin/env python
#coding=utf8
import pika
  
#连接rabbitmq服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='localhost'))
channel = connection.channel()
  
#定义队列
channel.queue_declare(queue='compute_queue')
print ' [*] Waiting for n'
  
#将n值加1
def increase(n):
  return n + 1
  
#定义接收到消息的处理方法
def request(ch, method, props, body):
  print " [.] increase(%s)" % (body,)
  response = increase(int(body))
  
  #将计算结果发送回控制中心，增加correlation_id的设定
  ch.basic_publish(exchange='',
           routing_key=props.reply_to,
           properties=pika.BasicProperties(correlation_id = \
                           props.correlation_id),
           body=str(response))
  ch.basic_ack(delivery_tag = method.delivery_tag)
  
channel.basic_qos(prefetch_count=1)
channel.basic_consume(request, queue='compute_queue')
  
channel.start_consuming()
```

center.py代码分析

控制中心代码稍微复杂些，其中比较关键的有三个地方：

使用python的uuid来产生唯一的correlation_id。
发送计算请求时，设定参数correlation_id。
定义一个字典来保存返回的数据，并且键值为相应线程产生的correlation_id。
代码如下：

```python
#!/usr/bin/env python
#coding=utf8
import pika, threading, uuid
  
#自定义线程类，继承threading.Thread
class MyThread(threading.Thread):
  def __init__(self, func, num):
    super(MyThread, self).__init__()
    self.func = func
    self.num = num
  
  def run(self):
    print " [x] Requesting increase(%d)" % self.num
    response = self.func(self.num)
    print " [.] increase(%d)=%d" % (self.num, response)
  
#控制中心类
class Center(object):
  def __init__(self):
    self.connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
  
    self.channel = self.connection.channel()
  
    #定义接收返回消息的队列
    result = self.channel.queue_declare(exclusive=True)
    self.callback_queue = result.method.queue
  
    self.channel.basic_consume(self.on_response,
                  no_ack=True,
                  queue=self.callback_queue)
  
    #返回的结果都会存储在该字典里
    self.response = {}
  
  #定义接收到返回消息的处理方法
  def on_response(self, ch, method, props, body):
    self.response[props.correlation_id] = body
  
  def request(self, n):
    corr_id = str(uuid.uuid4())
    self.response[corr_id] = None
  
    #发送计算请求，并设定返回队列和correlation_id
    self.channel.basic_publish(exchange='',
                  routing_key='compute_queue',
                  properties=pika.BasicProperties(
                     reply_to = self.callback_queue,
                     correlation_id = corr_id,
                     ),
                  body=str(n))
    #接收返回的数据
    while self.response[corr_id] is None:
      self.connection.process_data_events()
    return int(self.response[corr_id])
  
center = Center()
#发起5次计算请求
nums= [10, 20, 30, 40 ,50]
threads = []
for num in nums:
  threads.append(MyThread(center.request, num))
for thread in threads:
  thread.start()
for thread in threads:
  thread.join()
```

笔者开启了两个终端，来运行compute.py，开启一个终端来运行center.py，最后结果输出截图如下：

![201663092126912.png (280×164)](https://s2.ax1x.com/2019/06/30/ZlXlZT.png)

可以看到虽然获取的结果不是顺序输出，但是结果和源数据都是对应的。

这边示例的做法就是创建一个队列，使用correlation id来标识每次请求。也有做法可以不使用correlation id，就是每请求一次，就创建一个临时队列，不过这样太消耗性能了，官方也不推荐这么做。
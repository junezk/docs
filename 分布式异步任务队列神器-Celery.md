# 分布式异步任务队列神器-Celery

最近研究了下异步任务神器-Celery，发现非常好用，可以说是高可用，假如你发出一个任务执行命令给 Celery，只要 Celery 的执行单元 (worker) 在运行，那么它一定会执行；如果执行单元 (worker) 出现故障，如断电，断网情况下，只要执行单元 (worker) 恢复运行，那么它会继续执行你已经发出的命令。这一点有很强的实用价值：假如有交易系统接到了大量交易请求，主机却挂了，但前端用户仍可以继续发交易请求，发送交易请求后，用户无需等待。待主机恢复后，已发出的交易请求可以继续执行，只不过用户收到交易确认的时间延长而已，但并不影响用户体验。

Celery 简介
---------

它是一个异步任务调度工具，用户使用 Celery 产生任务，借用中间人来传递任务，任务执行单元从中间人那里消费任务。任务执行单元可以单机部署，也可以分布式部署，因此 Celery 是一个高可用的生产者消费者模型的异步任务队列。你可以将你的任务交给 Celery 处理，也可以让 Celery 自动按 crontab 那样去自动调度任务，然后去做其他事情，你可以随时查看任务执行的状态，也可以让 Celery 执行完成后自动把执行结果告诉你。

应用场景：
-----

1.  高并发的请求任务。互联网已经普及，人们的衣食住行中产生的交易都可以线上进行，这就避免不了某些时间极高的并发任务请求，如公司中常见的购买理财、学生缴费，在理财产品投放市场后、开学前的一段时间，交易量猛增，确认交易时间较长，此时可以把交易请求任务交给 Celery 去异步执行，执行完再将结果返回给用户。用户提交后不需要等待，任务完成后会通知到用户（购买成功或缴费成功），提高了网站的整体吞吐量和响应时间，几乎不需要增加硬件成本即可满足高并发。

2.  定时任务。在云计算，大数据，集群等技术越来越普及，生产环境的机器也越来越多，定时任务是避免不了的，如果每台机器上运行着自己的 crontab 任务，管理起来相当麻烦，例如当进行灾备切换时，某些 crontab 任务可能需要单独手工调起，给运维人员造成极大的麻烦，有了 Celery ，你可以集中管理所有机器的定时任务，而且灾备无论何时切换，crontab 任务总能正确的执行。

3.  异步任务。 一些耗时较长的操作，比如 I/O 操作，网络请求，可以交给 Celery 去异步执行，用户提交后可以做其他事情，当任务完成后将结果返回用户即可，可提高用户体验。


Celery 的优点
----------

1.  纯 Python 编写，开源。这已经是站在巨人的肩膀上了，虽然 Celery 是由纯 Python 编写的，但协议可以用任何语言实现。迄今，已有 Ruby 实现的 RCelery 、node.js 实现的 node-celery 以及一个 PHP 客户端 ，语言互通也可以通过 using webhooks 实现。

2.  灵活的配置。默认的配置已经满足绝大多数需求，因此你不需要编写配置文件基本就可以使用，当然如果有个性化地定制，你可以选择使用配置文件，也可以将配置写在源代码文件里。

3.  方便监控。任务的所有状态，均在你的掌握之下。

4.  完善的错误处理。

5.  灵活的任务队列和任务路由。你可以非常方便地将一个任务运行在你指定的队列上，这叫任务路由。


Celery 的架构
----------

学习一个工具，最好先从它的架构理解，辅以快速入门的代码来实践，最深入的就是阅读他的源码了，下图是 Celery 的架构图。

![celery架构](分布式异步任务队列神器-Celery.assets/168d15a9eb44a8c5)

**任务生产者** ：调用Celery提供的API，函数，装饰器而产生任务并交给任务队列的都是任务生产者。

**任务调度 Beat**：Celery Beat进程会读取配置文件的内容，周期性的将配置中到期需要执行的任务发送给任务队列

**中间人（Broker）**：Celery 用消息通信，通常使用中间人（Broker）在客户端和 worker 之前传递，这个过程从客户端向队列添加消息开始，之后中间人把消息派送给 worker。官方给出的实现Broker的工具有：

|    名称    |  状态  | 监视 | 远程控制 |
| :--------: | :----: | :--: | :------: |
|  RabbitMQ  |  稳定  |  是  |    是    |
|   Redis    |  稳定  |  是  |    是    |
|  Mongo DB  | 实验性 |  是  |    是    |
| Beanstalk  | 实验性 |  否  |    否    |
| Amazon SQS | 实验性 |  否  |    否    |
|  Couch DB  | 实验性 |  否  |    否    |
| Zookeeper  | 实验性 |  否  |    否    |
| Django DB  | 实验性 |  否  |    否    |
| SQLAlchemy | 实验性 |  否  |    否    |
|  Iron MQ   | 第三方 |  否  |    否    |

在实际使用中我们选择 RabbitMQ 或 Redis 作为中间人即可。

**执行单元 worker**：worker 是任务执行单元，是属于任务队列的消费者，它持续地监控任务队列，当队列中有新地任务时，它便取出来执行。worker 可以运行在不同的机器上，只要它指向同一个中间人即可，worker还可以监控一个或多个任务队列， Celery 是分布式任务队列的重要原因就在于 worker 可以分布在多台主机中运行。修改配置文件后不需要重启 worker，它会自动生效。

**任务结果存储backend**：用来持久存储 Worker 执行任务的结果，Celery支持不同的方式存储任务的结果，包括AMQP，Redis，memcached，MongoDb，SQLAlchemy等。

Celery 的使用示例：
-------------

以 Python3.6.5 版本为例。

### 1\. 安装 python 库：celery，redis。

```bash
pip install celery #安装celery 
pip install celery[librabbitmq,redis,auth,msgpack] #安装celery对应的依赖
```


celery其他的依赖包如下：  
序列化：  
celery\[auth\]：使用auth序列化。  
celery\[msgpack\]：使用msgpack序列化。  
celery\[yaml\]：使用yaml序列化。  
并发：  
celery\[eventlet\]：使用eventlet池。  
celery\[gevent\]：使用gevent池。  
celery\[threads\]：使用线程池。  
传输和后端：  
celery\[librabbitmq\]：使用librabbitmq的C库.  
celery\[redis\]：使用Redis作为消息传输方式或结果后端。  
celery\[mongodb\]：使用MongoDB作为消息传输方式（实验性），或是结果后端（已支持）。  
celery\[sqs\]：使用AmazonSQS作为消息传输方式（实验性）。  
celery\[memcache\]：使用memcache作为结果后端。  
celery\[cassandra\]：使用ApacheCassandra作为结果后端。  
celery\[couchdb\]：使用CouchDB作为消息传输方式（实验性）。  
celery\[couchbase\]：使用CouchBase作为结果后端。  
celery\[beanstalk\]：使用Beanstalk作为消息传输方式（实验性）。  
celery\[zookeeper\]：使用Zookeeper作为消息传输方式。  
celery\[zeromq\]：使用ZeroMQ作为消息传输方式（实验性）。  
celery\[sqlalchemy\]：使用SQLAlchemy作为消息传输方式（实验性），或作为结果后端（已支持）。  
celery\[pyro\]：使用Pyro4消息传输方式（实验性）。  
celery\[slmq\]：使用SoftLayerMessageQueue传输（实验性）。

### 2\. 安装 Redis，以 ubuntu 操作系统为例（如果使用 RabbitMQ，自己装一下就可以）。

通过源码安装：

    $ wget http://download.redis.io/releases/redis-4.0.11.tar.gz
    $ tar xzf redis-4.0.11.tar.gz
    $ cd redis-4.0.11
    $ make


修改 redis 配置文件 redis.conf，修改bind = 127.0.0.0.1为bind = 0.0.0.0，意思是允许远程访问redis数据库。

启动 redis-server

    $ cd src
    $ ./redis-server ../redis.conf


### 3\. 第一个 celery 应用程序。

功能：模拟一个耗时操作，并打印 worker 所在机器的 IP 地址，中间人和结果存储都使用 redis 数据库。

```python
#encoding=utf-8
#filename my_first_celery.py
from celery import Celery
import time
import socket

app = Celery(''tasks'', broker='redis://127.0.0.1:6379/0',backend ='redis://127.0.0.1:6379/0' )

def get_host_ip():
    """
    查询本机ip地址
    :return: ip
    """
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        s.connect(('8.8.8.8', 80))
        ip = s.getsockname()[0]
    finally:
        s.close()
    return ip

@app.task
def add(x, y):
    time.sleep(3) # 模拟耗时操作
    s = x + y
    print("主机IP {}: x + y = {}".format(get_host_ip(),s))
    return s
```


启动这个 worker：

    celery -A my_first_celery worker -l info


这里，-A 表示我们的程序的模块名称，worker 表示启动一个执行单元，-l 是批 -level，表示打印的日志级别。可以使用 celery –help 命令来查看celery命令的帮助文档。执行命令后，worker界面展示信息如下：

```bash
aaron@ubuntu:~/project$ celery -A my_first_celery worker -l info 
 
 -------------- celery@ubuntu v4.2.1 (windowlicker)
---- **** ----- 
--- * ***  * -- Linux-4.10.0-37-generic-x86_64-with-Ubuntu-16.04-xenial 2018-08-27 22:46:00
-- * - **** --- 
- ** ---------- [config]
- ** ---------- .> app:         tasks:0x7f1ce0747080
- ** ---------- .> transport:   redis://127.0.0.1:6379/0
- ** ---------- .> results:     redis://127.0.0.1:6379/0
- *** --- * --- .> concurrency: 1 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** ----- 
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery

[tasks]
  . my_first_celery.add

[2018-08-27 22:46:00,726: INFO/MainProcess] Connected to redis://127.0.0.1:6379/0
[2018-08-27 22:46:00,780: INFO/MainProcess] mingle: searching for neighbors
[2018-08-27 22:46:02,075: INFO/MainProcess] mingle: all alone
[2018-08-27 22:46:02,125: INFO/MainProcess] celery@ubuntu ready.
```


​    

已经相当清晰了。 如果你不想使用 celery 命令来启动 worker，可直接使用文件来驱动，修改 my\_first\_celery.py （增加入口函数main）

    if __name__ == '__main__':
        app.start()


再执行

    python my_first_celery.py worker


即可。

### 4\. 调用任务

在 my\_first\_celery.py 的同级目录下编写如下脚本 start_task.py如下。

    from my_first_celery import add #导入我们的任务函数add
    import time
    result = add.delay(12,12) #异步调用，这一步不会阻塞，程序会立即往下运行
    
    while not result.ready():# 循环检查任务是否执行完毕
        print(time.strftime("%H:%M:%S"))
        time.sleep(1)
    
    print(result.get()) #获取任务的返回结果
    print(result.successful()) #判断任务是否成功执行


执行

    python start_task.py


结果如下所示：

    22:50:59
    22:51:00
    22:51:01
    24
    True


发现等待了大约3秒钟后，任务返回了结果24，并且是成功完成，此时worker界面增加的信息如下：

    [2018-08-27 22:50:58,840: INFO/MainProcess] Received task: my_first_celery.add[a0c4bb6b-17af-474c-9eab-407d593a7807]  
    [2018-08-27 22:51:01,898: WARNING/ForkPoolWorker-1] 主机IP 192.168.195.128: x + y = 24
    [2018-08-27 22:51:01,915: INFO/ForkPoolWorker-1] Task my_first_celery.add[a0c4bb6b-17af-474c-9eab-407d593a7807] succeeded in 3.067237992000173s: 24


这里的信息非常详细，其中a0c4bb6b-17af-474c-9eab-407d593a7807是taskid，只要指定了 backend，根据这个 taskid 可以随时去 backend 去查找运行结果，使用方法如下：

    >>> from my_first_celery import add
    >>> taskid= 'a0c4bb6b-17af-474c-9eab-407d593a7807'
    >>> add.AsyncResult(taskid).get()
    24
    >>>#或者
    >>> from celery.result import AsyncResult
    >>> AsyncResult(taskid).get()
    24


重要说明：如果想远程执行 worker 机器上的作业，请将 my\_first\_celery.py 和 start\_tasks.py 复制到远程主机上（需要安装 celery)，修改 my\_first\_celery.py 指向同一个中间人和结果存储，再执行 start\_tasks.py 即可远程执行 worker 机器上的作业。my\_first\_celery.add函数的代码不是必须的，你也要以这样调用任务：

    from my_first_celery import app
    app.send_task("my_first_celery.add",args=(1,3))


### 5\. 第一个 celery 项目

在生产环境中往往有大量的任务需要调度，单独一个文件是不方便的，celery 当然支持模块化的结构，我这里写了一个用于学习的 Celery 小型工程项目，含有队列操作，任务调度等实用操作，目录树如下所示：

![img](分布式异步任务队列神器-Celery.assets/168d15a9eb327088)

其中 **init**.py是空文件，目的是告诉 Python myCeleryProj 是一个可导入的包.  
app.py

    from celery import Celery
    
    app = Celery("myCeleryProj", include=["myCeleryProj.tasks"])
    
    app.config_from_object("myCeleryProj.settings")
    
    if __name__ == "__main__":
        app.start()


settings.py

    from kombu import Queue
    import re
    from datetime import timedelta
    from celery.schedules import crontab


​    
​    CELERY_QUEUES = (  # 定义任务队列
​        Queue("default", routing_key="task.#"),  # 路由键以“task.”开头的消息都进default队列
​        Queue("tasks_A", routing_key="A.#"),  # 路由键以“A.”开头的消息都进tasks_A队列
​        Queue("tasks_B", routing_key="B.#"),  # 路由键以“B.”开头的消息都进tasks_B队列
​    )
​    
​    CELERY_TASK_DEFAULT_QUEUE = "default"  # 设置默认队列名为 default
​    CELERY_TASK_DEFAULT_EXCHANGE = "tasks"
​    CELERY_TASK_DEFAULT_EXCHANGE_TYPE = "topic"
​    CELERY_TASK_DEFAULT_ROUTING_KEY = "task.default"
​    
​    CELERY_ROUTES = (
​        [
​            (
​                re.compile(r"myCeleryProj\.tasks\.(taskA|taskB)"),
​                {"queue": "tasks_A", "routing_key": "A.import"},
​            ),  # 将tasks模块中的taskA,taskB分配至队列 tasks_A ,支持正则表达式
​            (
​                "myCeleryProj.tasks.add",
​                {"queue": "default", "routing_key": "task.default"},
​            ),  # 将tasks模块中的add任务分配至队列 default
​        ],
​    )


​    
​    # CELERY_ROUTES = (
​    #    [
​    #        ("myCeleryProj.tasks.*", {"queue": "default"}), # 将tasks模块中的所有任务分配至队列 default
​    #    ],
​    # )
​    
​    # CELERY_ROUTES = (
​    #    [
​    #        ("myCeleryProj.tasks.add", {"queue": "default"}), # 将add任务分配至队列 default
​    #        ("myCeleryProj.tasks.taskA", {"queue": "tasks_A"}),# 将taskA任务分配至队列 tasks_A
​    #        ("myCeleryProj.tasks.taskB", {"queue": "tasks_B"}),# 将taskB任务分配至队列 tasks_B
​    #    ],
​    # )
​    
​    BROKER_URL = "redis://127.0.0.1:6379/0"  # 使用redis 作为消息代理
​    
​    CELERY_RESULT_BACKEND = "redis://127.0.0.1:6379/0"  # 任务结果存在Redis
​    
​    CELERY_RESULT_SERIALIZER = "json"  # 读取任务结果一般性能要求不高，所以使用了可读性更好的JSON
​    
​    CELERY_TASK_RESULT_EXPIRES = 60 * 60 * 24  # 任务过期时间，不建议直接写86400，应该让这样的magic数字表述更明显


​    
​    CELERYBEAT_SCHEDULE = {
​        "add": {
​            "task": "myCeleryProj.tasks.add",
​            "schedule": timedelta(seconds=10),
​            "args": (10, 16),
​        },
​        "taskA": {
​            "task": "myCeleryProj.tasks.taskA",
​            "schedule": crontab(hour=21, minute=10),
​        },
​        "taskB": {
​            "task": "myCeleryProj.tasks.taskB",
​            "schedule": crontab(hour=21, minute=12),
​        },
​    }


tasks.py

    import os
    from myCeleryProj.app import app
    import time
    import socket


​    
​    def get_host_ip():
​        """
​        查询本机ip地址
​        :return: ip
​        """
​        try:
​            s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
​            s.connect(("8.8.8.8", 80))
​            ip = s.getsockname()[0]
​        finally:
​            s.close()
​        return ip


​    
​    @app.task
​    def add(x, y):
​        s = x + y
​        time.sleep(3)  # 模拟耗时操作
​        print("主机IP {}: x + y = {}".format(get_host_ip(), s))
​        return s


​    
​    @app.task
​    def taskA():
​        print("taskA begin...")
​        print(f"主机IP {get_host_ip()}")
​        time.sleep(3)
​        print("taskA done.")


​    
​    @app.task
​    def taskB():
​        print("taskB begin...")
​        print(f"主机IP {get_host_ip()}")
​        time.sleep(3)
​        print("taskB done.")


readme.txt

    #启动 worker 
    #分别在三个终端窗口启动三个队列的worker，执行命令如下所示：
    celery -A myCeleryProj.app worker -Q default -l info
    celery -A myCeleryProj.app worker -Q tasks_A -l info
    celery -A myCeleryProj.app worker -Q tasks_B -l info
    #当然也可以一次启动多个队列，如下则表示一次启动两个队列tasks_A，tasks_B。
    celery -A myCeleryProj.app worker -Q tasks_A,tasks_B -l info
    #则表示一次启动两个队列tasks_A，tasks_B。
    #最后我们再开启一个窗口来调用task: 注意观察worker界面的输出
    >>> from myCeleryProj.tasks import *
    >>> add.delay(4,5);taskA.delay();taskB.delay() #同时发起三个任务
    <AsyncResult: 21408d7b-750d-4c88-9929-fee36b2f4474>
    <AsyncResult: 737b9502-77b7-47a6-8182-8e91defb46e6>
    <AsyncResult: 69b07d94-be8b-453d-9200-12b37a1ca5ab>
    #也可以使用下面的方法调用task
    >>> from myCeleryProj.app import app
    >>> app.send_task(myCeleryProj.tasks.add,args=(4,5)
    >>> app.send_task(myCeleryProj.tasks.taskA)
    >>> app.send_task(myCeleryProj.tasks.taskB)


（完）
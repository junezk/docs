# 用flask+mongo打造分布式服务器监控平台

> 阅读本文，代表你将和 奎因 一起，从 0 开始快速入门 flask 开发以及 mongodb 的使用，打造一个稳定、健壮的分布式多机服务器资源监控平台

在前面一篇文章《 Python 系统资源信息获取工具，你用过没？》中，我们学习了如何使用 Python 中的第三方库 psutil 来获取系统的资源信息，比如 cpu 占用率、内存使用情况、硬盘以及进程情况等。并且奎因在文章的末尾还大言不惭、信誓旦旦的说可以用 psutil 打造一个分布式服务器监控平台

我们翻翻旧帐，看看 **奎因** 上一篇文章末尾说的分布式服务器监控平台是什么：

![img](https://user-gold-cdn.xitu.io/2018/11/30/16764663839f8b28?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 项目开始之前的故事

现在 **奎因** 的手上有 N 台 Linux 服务器，上面部署了很多的爬虫以及爬虫依赖的服务，**德玛西亚阵营** 希望 **奎因** 能够尽量节省开销，毕竟上次跟 **诺克萨斯** 一战之后军费就很紧张。**奎因** 临危受命， 负责爬虫项目的优化。

但是在必须在优化之前，记录服务器上的资源峰值以及对应的爬虫进程数，以估算爬虫数量与资源消耗量的关联关系。市面上的服务器监控平台很多，而且功能强大无比，为什么 **奎因** 要自己打造，而不用现成的呢？

这个项目的需求并不大、也不复杂，而且定制化程度较高，如果使用成熟的服务器监控平台未免也太浪费了，而且需要频繁的记录服务器峰值情况，恐怕还得更改代码 -- **奎因** 考虑到这些问题，心中觉得不如重写来的实在，毕竟只有造轮子，才能够进步得更快。

## 造一个什么样的轮子

上面的项目图是在假设需求的情况下制作的，但是现在有了真正的需求，那么就应该将上面图的进行调整和细化，想到这里，**奎因**不禁皱了皱眉头：要是 **华洛** 在这里就好了，至少还有个可以商量的人，但是 **华洛** 被派出去执行任务，**奎因** 也只好自己操刀了：



![img](https://user-gold-cdn.xitu.io/2018/11/30/16764525b688b043?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



首先，这个模块完成的任务是获取服务器资源信息，包括当前 cpu、内存和磁盘的占用率以及当前 python （因为爬虫是 python 编写的，所以监控 python 进程信息）相关进程的列表，它的名字似乎不太合适，应该改为 系统资源信息获取模块。

然后在存储中心这里，redis 似乎不太适合，如果要用 nosql 的话，mongodb 可能更适合。而且还要考虑分布式的问题。 **奎因** 心中很清楚，分布式不过是使用一个中心存储，将不同端的数据共同存储在数据中心而已，那么这就衍生了三种情况：

- A).对等分布式 -- 每台服务器上的代码都一样，功能也都相同
- B).主从分布式之推模式 -- 多台从机将数据推给主机，由主机负责存储和其他操作。
- C).主从分布式之拉模式 -- 由主机主动向从机索要数据，形象的理解为拉。

**奎因** 脑海中闪过一个很形象的画面： 

![img](https://user-gold-cdn.xitu.io/2018/11/30/167645a038966141?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 推模式与拉模式的选择

推与拉，看似方向相同，但是却需要不同的操作。

首先要考虑的是主机所在的环境，如果主机不在公网当中，而从机都在公网中（比如阿里云服务器），这个情况下主机可以访问从机，但是从机却无法访问主机，推拉模式当中的 **推** 显然是不适用的，反之亦然。

**奎因** 咨询了峡谷先锋，得知目前 **德玛西亚阵营** 的服务器中主机不在公网，而从机都在公网，所以很适合 **拉** 模式。根据经验得出，**拉** 模式在代码中的表现通常是主机 request.get(从机)。

所以，存储中心只需要提供给主机即可，从机不需要访问到存储中心，那么图就可以改为：



![img](https://user-gold-cdn.xitu.io/2018/11/30/167646377df2acc8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 警报中心

**奎因** 是一名优秀的战士，甚至警报的重要性，这么大的项目怎么能没有警报功能呢？

警报中心需要即时又准确，那么警报中心就分为两部分：发现警情和通知。也就是阈值检测以及邮件发送功能。



![img](https://user-gold-cdn.xitu.io/2018/11/30/1676468493bf9544?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 数据处理与可视化

要让大家相信，必须拿出真凭实据，所以数据的处理与可视化是必不可少的，但是由于 **华洛** 外出执行任务，所以对于数据的呈现这部分 **奎因** 也没有什么头绪，那就暂且保留原样吧。

所以整个服务器信息监控平台的结构图如下：

![img](https://user-gold-cdn.xitu.io/2018/11/30/167646b02c470b7b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 临阵磨枪

既然已经规划好了，那么差不多就可以开始动工了。**奎因** 最擅长使用的就是工具，他拥有有整个联盟最强的弓弩，必定知道工具选择的重要性。

既然这是一个平台，那么就需要 WEB 服务作为支撑，纵观 Python 领域，最受欢迎的莫过于 Django 和 Flask，由于项目并不大，而且要求轻量、快速，所以 **奎因** 选择了他从未使用过的 Flask restful [点击前往文档](https://link.juejin.im?target=https%3A%2F%2Fflask-restful.readthedocs.io%2Fen%2Flatest%2F)。

刚才探讨到数据存储中心，那就选定 Mongodb，但是为了快速开发，还需要选定一个能够将 Flask 和 Mongodb 连接在一起的中间工具。目前有几种可以选择：

- A).Pymongo -- mongodb 的 Python 连接库
- B).Flask-pymonggo -- 在 A 的基础上封装了一层
- C).MongoEngine -- 一个成熟的 Mongodb ORM 框架

三种都是可以完成任务的，**华洛** 曾经跟 **奎因**  说过：从速度、代码结构完整性、易用性以及可维护性几个方面来考虑的话，ORM 当然是最合适的，所以这里选择 MongoEngine [点击前往文档](https://link.juejin.im?target=http%3A%2F%2Fdocs.mongoengine.org%2F)

虽然 Flask restful 和 MongoEngine 是 **奎因** 之前很少接触的工具，但是 **奎因** 相信 **华洛** 的推荐，也相信自己的快速学习能力

## 最重要的能力

**奎因**心里深知：你永远无法学会所有的知识，但是如果你学会了如何快速学习，那么你就会变得很强。

## 开始

经过上面的分析和规划 『项目似了然于胸  又飘渺于峡谷之中』

### 工具的安装

如果你真的从 0 开始，那么所有的学习都可以从官方文档开始。

根据 [Flask restful](https://link.juejin.im?target=https%3A%2F%2Fflask-restful.readthedocs.io%2Fen%2Flatest%2Finstallation.html) 文档

```
pip install flask-restful
```

 和 `MongoEngine`

```
pip install mongoengine
```

以及 **Mongodb 文档**

![img](用flask+mongo打造分布式服务器监控平台.assets/16764833a290fc48)

的安装介绍

[Mongodb 可视化工具 Robt 3T](https://link.juejin.im?target=https%3A%2F%2Frobomongo.org%2Fdownload)

![img](用flask+mongo打造分布式服务器监控平台.assets/167649a95b0e7c7e)

的安装介绍

**由于有官方文档，所以奎因很快就将他们安装好了**

### 运行 Flask

![img](用flask+mongo打造分布式服务器监控平台.assets/1676484eed1579e1)

Flask restful 文档中的 [快速开始](https://link.juejin.im?target=https%3A%2F%2Fflask-restful.readthedocs.io%2Fen%2Flatest%2Fquickstart.html) 部分，给出了一个简单的示例代码，我们可以通过这段代码来启动 Flask。当然 **奎因** 从来都不会这么做，因为直接复制粘贴文档是不可能快速学习的，必须立即将文档转为自己的知识：

```
from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)


class Home(Resource):
    
    def get(self):
        return {'status': 'success', 'message': 'Flask restful is running'}


api.add_resource(Home, '/')

if __name__ == '__main__':
    app.run(debug=True)
```

然后我们运行代码

![img](用flask+mongo打造分布式服务器监控平台.assets/167648892454c79a)

得到 Flask 已经启动的提示信息，接着我们可以点击链接，到浏览器上看一看输出了什么内容：

![img](用flask+mongo打造分布式服务器监控平台.assets/1676489528ced32c)



如代码所写，正确输出了刚才的内容，说明适当的改动是可以的，不怕不怕。

### Flask 与 MongoEngine

通过 MongoEngine 文档 [Connecting to MongoDB](https://link.juejin.im?target=http%3A%2F%2Fdocs.mongoengine.org%2Fguide%2Fconnecting.html) 部分的介绍：

![img](用flask+mongo打造分布式服务器监控平台.assets/167648c2252ac671)

 我们很快就学会了如何用代码连接 Mongodb 数据库(将刚才的代码改为)：

```
from flask import Flask
from flask_restful import Resource, Api
from mongoengine import *

connect(
    host='mongodb://localhost/server_resource'
)

app = Flask(__name__)
api = Api(app)


class Home(Resource):

    def get(self):
        return {'status': 'success', 'message': 'Flask restful is running'}


api.add_resource(Home, '/')

if __name__ == '__main__':
    app.run(debug=True)

```

### 数据模型

数据模型可以让我们很轻松的操作数据库，而且还能够保证代码的整洁性和美观程度 -- 就像士兵总喜欢将自己的武器擦亮一样 **奎因** 就经常这么做。

[文档](https://link.juejin.im?target=http%3A%2F%2Fdocs.mongoengine.org%2Fguide%2Fdefining-documents.html) 上面清楚的写着应该如何建立数据模型，并且用一些例子告诉我们字段的设置和定义应该如何选择。

我们现在想一想，需要定义哪一些数据模型呢？

首先，回到那个推拉的模式，我们已经确定使用 **拉** 模式（不是龙龟 **拉莫斯** ），所以意味着主机要从从机上拉取数据，主机如何在数据记录中确定数据到底是来自于哪一台从机呢？

所以，我们需要一个服务器队列，队列中记录了从机的名称、ip 地址以及备注信息等，那么数据模型就可以定义了。

在代码中新增一个数据模型类：

```
class ResourceModel(Document):
    """ 服务器队列模型 """
    name = StringField(max_length=255, required=True)
    address = StringField(max_length=160, required=True)
    description = StringField(max_length=255)
```

运行代码后，你就可以打开 Mongodb 可视化工具，查看新生成的 Collection 了。

今天我们将编写功能模块，并在代码开始之前对功能模块进行分析，并通过流程图和 UML 类图的绘制将模块功能细化，接着从搭建骨架开始，逐步完成一个模块的编写。

接下来我们应该对每个模块功能进行编写，并且为每个编写好的模块编写视图。

## 模块编写顺序

> 思考：在多个模块当中，**萌新** 应该先完成哪一个模块呢？

我们给需求进行排序，看看他们各自的权重和依赖关系，回顾一下我们需要编写的模块：

- 数据处理与可视化
- 信息监控
- 警报模块
- 数据存储
- API 视图

**奎因** 画了一张图，从这张图上我们就可以看出每个模块的依赖和作用：

![img](用flask+mongo打造分布式服务器监控平台.assets/1677334c7acd417d)

首先应该先读取每台服务器的资源，在读取资源的时候检查是否超过阈值，然后主机一次性获取所有服务器资源信息并存入数据库，接着出库计算并将数据处理成可视化图表。所以可以将上面的模块编写顺序做一下调整：

- 信息监控
- 数据存储
- 数据处理与可视化

至于剩下的警报模块和 API 视图，在前三个模块开发过程中用于调试，可以只保留逻辑占位（也就是 python 中常用的 Pass）待前三个模块开发完成后再回头补充警报模块和 API 视图的逻辑即可。

## 信息监控模块的实现

![img](用flask+mongo打造分布式服务器监控平台.assets/167732668e0fa3ae)



在 [《Python 系统资源信息获取工具，你用过没？》](https://juejin.im/post/5bfdc2675188250808258485) 中，我们已经学会了(非 windows)服务器资源信息的获取，现在我们需要将之前所学的知识转化成为功能模块，也就是在代码中通过类来实现资源信息获取，这样我们就可以通过类的实例化和方法调用的方式获取的所需的数据。

> 思考：在弄清楚需求、作用之后，我们就可以开始编写代码了吗？

如果你是一名经验丰富的开发者，想必你心中已经有了代码的一个大致结构。但是作为萌新，我们还没能够那么快在脑海中生成结构，所以我们还是需要画图。在代码开始前进行构思、画图可以让萌新们减少错误、同时也减少代码的改动次数，有一定的几率提升，既然现在是编写类，那么我们就来绘制一个 UML 类图吧！

首先，我们应该给类想一个名字，就叫 PresentMomentSystemResource 吧，所以一个空的 UML 图就可以画出来了：



![img](用flask+mongo打造分布式服务器监控平台.assets/167734b6c5ba2d96)



至于里面的方法，我们想一下：

- 应该有一个 **init** 方法，这样就可以在类实例化的时候指定一些类变量；
- 对于 cpu、内存硬盘以及进程的数据，应该有不同的方法进行获取；
- 考虑到这个类可能会被多次实例化，在多个地方被调用，那么就有可能需要使用单例模式；

所以 UML 图改动一下：



![img](用flask+mongo打造分布式服务器监控平台.assets/1677354e540db5f1)



**奎因** 给大家解释一下这个 UML 类图的含义：

- 魔术方法 new 来完成类单例模式
- 魔术方法 init 设定实例化时使用的一些类变量
- 接下来给每个硬件信息数据定义一个方法，并且的到的结果都以 dict 的数据类型返回

然后我们就可以开始代码的编写了。

对于这一次的项目，我们新建一个文件夹 Monitors，然后再在里面新建一个 python package 名字叫 monitors，接着在项目内新建一个名为 [core.py](https://link.juejin.im?target=http%3A%2F%2Fcore.py) 的文件，结构如下图所示：

![img](用flask+mongo打造分布式服务器监控平台.assets/16773af1c3bd6ef7)

并且根据 UML 类图编写类的基础结构：

```
class PresentMomentSystemResource:

    def __new__(cls, *args, **kwargs):
        # singleton
        pass

    def __init__(self):
        pass

    def memory_usage(self):
        """当前时刻内存用量信息
        """
       pass

    def cpu_usage(self):
        """ 当前时刻cpu用量信息 """
        pass

    def disks_usage(self):
        """ 当前时刻根目录磁盘用量信息 """
        pass

    def processes_id(self):
        """ 筛选当前时刻关键字相关的pid列表及数量 """
        pass
```

搭好类的骨架之后，我们就来为每个方法编写实际的代码。

首先，我们用魔术方法 new 将类变成单例模式，所以 new 方法部分的代码改为：

```
    def __new__(cls, *args, **kwargs):
        # singleton
        if not hasattr(cls, '_instance'):
            cls._instance = super(PresentMomentSystemResource, cls).__new__(cls, *args, **kwargs)
        return cls._instance
```

作为老司机，应该是可以在编写完代码开始下一个方法的编写。但是现在我们是 **召唤师峡谷的萌新** ，对不对？那 **萌新** 肯定是不知道自己写的代码是否正确，所以我们需要编写测试代码，只要确定类实例化后确实只有一个实例对象，那么就可以进行下一步了。

所以我们还需要新建一个测试文件 [testing.py](https://link.juejin.im?target=http%3A%2F%2Ftesting.py)，并在里面编写：

```
from monitors.core import PresentMomentSystemResource

if __name__ == "__main__":
    p1 = PresentMomentSystemResource()
    p2 = PresentMomentSystemResource()
    print(p1, p2)
```

运行 [testing.py](https://link.juejin.im?target=http%3A%2F%2Ftesting.py) 文件，我们看一看控制台的输出内容：

```
<monitors.core.PresentMomentSystemResource object at 0x7fb0862a7128>
<monitors.core.PresentMomentSystemResource object at 0x7fb0862a7128>

Process finished with exit code 0
```

由输出结果得知，p1 和 p2 是同一个实例对象，说明 new 方法实现单例模式奏效了。

接着我们来编写下一个方法。在之前的文章 [《Python 系统资源信息获取工具，你用过没？》](https://juejin.im/post/5bfdc2675188250808258485) 中提到过可以获取系统资源信息的 psutil ，并且知道获取 cpu 信息、内存信息以及磁盘信息所用的方法，所以我们可以在 init 方法中将这几个方法初始化：

```
import psutil

    def __init__(self):
        self.memory = psutil.virtual_memory()
        self.cpu = psutil.cpu_times()
        self.disk = psutil.disk_usage("/")
```

其中关于磁盘部分的信息，我们制定获取 "/" 盘符信息即可，其他挂载磁盘信息并没有那么重要。

获取到的数值单位为ｋ，但是通常情况下我们使用的单位是 M 或者 G ，这里还需要设定两个单位：

```
from math import pow

    def __init__(self):
        self.memory = psutil.virtual_memory()
        self.cpu = psutil.cpu_times()
        self.disk = psutil.disk_usage("/")
        self.mb = pow(1024, 2)
        self.gb = pow(1024, 3)
```

pow 方法是 python 内置库中用于计算幂的方法，可以计算某个数值 m 的 n 次幂，也就是可以将它理解为:

> pow(m, n) = m 的n 次方。

到了真正编写每个硬件资源信息代码的时候了，我们首先来看看内存。内存需要的信息为内存总量、已使用量、剩余量及剩余百分比。我们从之前的文章可以知道，通过上面定义的 self.memory 就可以直接取到部分内存的用量信息：

```
    def memory_usage(self):
        """当前时刻内存用量信息
        """
        total = self.memory.tota
        used = self.memory.used

```

psutil 并没有给我们提供直接获取余量和余量百分比，所以我们在将数值单位计算完毕后，可以用数学运算计算出余量和余量百分比，此处 memory_usage 代码改为：

```
    def memory_usage(self):
        """当前时刻内存用量信息
        """
        total = self.memory.total/self.gb
        used = self.memory.used/self.gb
        free = total - used
        percent = round(free/total, 2)
        return total, used, free, percent
```

然后到 [testing.py](https://link.juejin.im?target=http%3A%2F%2Ftesting.py) 中测试一下：

```
from monitors.core import PresentMomentSystemResource

if __name__ == "__main__":
    p1 = PresentMomentSystemResource()
    res = p1.memory_usage()
    print(res)
```

运行后得到的输出结果为：

```
(7.676643371582031, 1.7717132568359375, 5.904930114746094, 0.77)
```

跟系统自带的系统资源监控做个比对：

![img](用flask+mongo打造分布式服务器监控平台.assets/16773c34d28cbf50)



总体上是吻合的，说明这种方法取值和计算是没有问题的。

然后按照之前的文章和这样的方法，编写其他几个硬件的代码，最后整个 [core.py](https://link.juejin.im?target=http%3A%2F%2Fcore.py) 文件的代码为：

```python
import psutil
from math import pow
from functools import reduce


class PresentMomentSystemResource:

    def __new__(cls, *args, **kwargs):
        # singleton
        if not hasattr(cls, '_instance'):
            cls._instance = super(PresentMomentSystemResource, cls).__new__(cls, *args, **kwargs)
        return cls._instance

    def __init__(self):
        self.memory = psutil.virtual_memory()
        self.cpu = psutil.cpu_times()
        self.disk = psutil.disk_usage("/")
        self.mb = pow(1024, 2)
        self.gb = pow(1024, 3)

    @property
    def memory_usage(self):
        """当前时刻内存用量信息
        """
        total = self.memory.total/self.gb
        used = self.memory.used/self.gb
        free = total - used
        percent = round(free/total, 2)
        buffers = self.memory.buffers/self.gb
        cached = self.memory.cached/self.gb
        total, used, free, buffers, cached = map(lambda x: round(x, 2), [total, used, free, buffers, cached])
        return {"total": total, "used": used, "free": free, "free_percent": percent, "buffers": buffers, "cached": cached}

    @property
    def cpu_usage(self):
        """ 当前时刻cpu用量信息 """
        count = psutil.cpu_count()
        logical_count = psutil.cpu_count(logical=True)
        percent = psutil.cpu_percent(interval=1)
        return {"count": count, "logical_count": logical_count, "percent": percent}

    @property
    def disks_usage(self):
        """ 当前时刻根目录磁盘用量信息 """
        total, used, free = map(lambda x: round(x/self.gb), self.disk[:3])
        percent = self.disk.percent
        return {"total": total, "used": used, "free": free, "free_percent": percent}

    def processes_id(self, keywords=['python']):
        """ 筛选当前时刻关键字相关的pid列表及数量 """
        attrs = psutil.process_iter(attrs=['pid', 'name'])
        pid = [[p.info for p in attrs if keyword in p.info['name']] for keyword in keywords]
        pid_number = reduce(lambda x, y: x+y, [len(p) for p in pid])
        return {"pid": pid, "type_number": len(pid), "pid_number": pid_number}

```

**这里着重说明一下**：由于我们的监控是针对爬虫与服务器资源关系的监控，所以 processes_id 方法限定进程 id 的获取仅获取 Python 相关的进程。

## Flask 视图编写

有了信息获取，那么我们来试试，如何在 flask 中使用这个类。

首先，我们在 monitors 的 [init.py](https://link.juejin.im?target=http%3A%2F%2Finit.py) 文件中设置好 flask

```
from flask import Flask
from flask.ext.restful import Resource, Api

app = Flask(__name__)
api = Api(app)
resource = Resource
```

然后新建一个 [start.py](https://link.juejin.im?target=http%3A%2F%2Fstart.py) 文件，并像之前的文章一样将 flask 的骨架搭好

(在此之前请在电脑的 python 环境中安装 flask、flask-restful)：

```
# start.py
from monitors import app, api, resource

class PresentMomentSystemResourceView(resource):
    """ 当前时刻系统资源占用信息视图 """
    def __init__(self):
        pass

    def get(self):
        
        return {"status": "success", "message": "this is flask view"}

api.add_resource(PresentMomentSystemResourceView, '/')

if __name__ == '__main__':
    app.run(debug=False)
```

接着运行 [start.py](https://link.juejin.im?target=http%3A%2F%2Fstart.py) 文件，得到输出：

```
  from flask.ext.restful import Resource, Api
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

说明我们可以通过浏览器访问本机的 5000 端口：

![img](用flask+mongo打造分布式服务器监控平台.assets/16773d0ba189ba96)

这里返回的内容就是刚才编写的试图时 return 的内容，说明 flask 的视图骨架搭好了。下一步则是将系统资源信息获取类与视图类相关联，将 start .py 的代码改为：

```
# start.py
from monitors import app, api, resource
from monitors.core import PresentMomentSystemResource


class PresentMomentSystemResourceView(resource):
    """ 当前时刻系统资源占用信息视图 """
    def __init__(self):
        self.sr = PresentMomentSystemResource()

    def get(self):
        memory = self.sr.memory_usage
        disks = self.sr.disks_usage
        cpu = self.sr.cpu_usage
        pid = self.sr.processes_id()
        return {"cpu": cpu, "memory": memory, "disk": disks, "pid": pid}
```

在运行 [start.py](https://link.juejin.im?target=http%3A%2F%2Fstart.py) 文件后，我们刷新刚才浏览器的页面，得到一串数据（火狐浏览器自动格式化，其他浏览器的数据显示可能没有那么整齐）：

![img](用flask+mongo打造分布式服务器监控平台.assets/16773d7accc067c0)

这些数据就是我们在视图类中 return 的 cpu、内存、磁盘以及进程信息数据。

至此，我们 **德玛西亚阵营** 的服务器信息获取模块就编写完成，下一次我们将会编写数据存储以及其他的模块。


# 使用 django middleware 和 celery 隔离业务系统和积分系统的尝试

游戏类或是论坛类web应用，到后面很多都会有积分系统：像发成功一篇文章，回复一条博客得多少积分这种。积分系统和很多的业务场景都有关系，但是和具体的用户请求没有关系。应该将积分系统和具体的业务程序进行隔离，以降低耦合度。笔者要开发的积分系统`不需要具有实时性`，所以打算用异步的方式来实现积分计算程序。 **实现方案:**

> - 使用`celery`实现积分计算和验证程序
> - 在django middleware 中准备好参数，并异步调用积分系统

------

## django middleware 简介

Middleware is a framework of hooks into Django’s request/response processing. It’s a light, low-level “plugin” system for globally altering Django’s input or output. 以上摘自[django官网](https://docs.djangoproject.com/en/2.1/topics/http/middleware/) *个人觉得* django middleware 是一种面向方面编程(AOP)思想的实现: 对所有的views程序提供统一的无感的处理。实际场景中可用middleware来统一response输出格式，统一异常处理，还有统一在输入中加入登录验证信息等。 新版的自定义middleware方式很简单:

1. 新建python文件

```
class TestMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # 这后面写的代码对中间件进行初始化，只执行一次

    def __call__(self, request):
        # 这里写的代码在调用view的方法之前执行，可以对request进行处理

        response = self.get_response(request)

        # 这里写的代码在调用view方法之后执行，可以对response的返回值进行统一格式处理等
        
        return response
```

1. 写好middleware逻辑后，将这个类放到django的settings文件中

```
# 注意: django调用middle时按照这里的排列顺序来 装饰 的，越前面的装饰的越外层，即:
# 排在越前面的，在调用views方法前越早被执行，调用views方法后越晚被执行
# 排在越后面的，在调用views方法前越晚被执行，调用views方法后越早被执行
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'your middleware path',
]
```

写好这些后，这个middleware就完成了 [django middleware](https://docs.djangoproject.com/en/2.1/ref/middleware/)

## django 集成 celery

Celery - Distributed Task Queue Celery is a simple, flexible, and reliable distributed system to process vast amounts of messages, while providing operations with the tools required to maintain such a system. It’s a task queue with focus on real-time processing, while also supporting task scheduling. 以上信息摘自[celery官网](http://docs.celeryproject.org/en/master/)

celery是用python语言开发的分布式任务队列，可用于任务的同步处理(接收任务处理返回结果)，也可用于异步处理(不接收返回结果)。在django中处理异步任务(发邮件；消息推送等)时，celery是个很好的选择。 运行celery需要中间件，这里使用rabbitMQ，下面简单介绍如何集成。

### 安装rabbitMQ

1. 执行安装 brew install rabbitmq
2. 设置环境变量 `PATH=$PATH:/usr/local/sbin`
3. 启动 sudo rabbitmq-server -detached
4. 设置用户

> `$ sudo rabbitmqctl add_user myuser mypassword` `$ sudo rabbitmqctl add_vhost myvhost` `$ sudo rabbitmqctl set_user_tags myuser mytag` `$ sudo rabbitmqctl set_permissions -p myvhost myuser ".*" ".*" ".*"` 设置好后在celery中直接配置 `'amqp://myuser:mypassword@localhost:5672/myvhost'`就可以用了

1. 启停 启动: `sudo rabbitmq-server` 以后台运行方式启动: `$ sudo rabbitmq-server -detached` 停止: `$ sudo rabbitmqctl stop` Never use kill (kill(1)) to stop the RabbitMQ server（忌直接杀进程！！！） 详细操作见[rabbit官网](http://www.rabbitmq.com/download.html)

### 安装celery

celery 是用python开发的，可以直接用pip安装 `$ pip install -U Celery` 更多详细安装选项见[celery官网](http://docs.celeryproject.org/en/master/getting-started/introduction.html#installation)

### django集成celery

django集成celery是将celery的配置嵌入到django的settings文件中，更重要的，是celery执行任务的时候，能直接使用django项目的运行环境。

- 配置settings文件：上面的rabbitMQ安装设置好后，在settings中直接引入 `CELERY_BROKER_URL = 'amqp://myuser:mypassword@localhost:5672/myvhost'`
- 编写celery执行入口文件`celery.py`，***文件放在settings的同级目录下***

```
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery


# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'test_app.settings')

app = Celery('msl')

# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()

@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

- 编写自定义任务 在自定义的app的`一级目录`下面新建python文件，名称为 tasks.py用于构建自定义的任务。

```
# Create your tasks here
from __future__ import absolute_import, unicode_literals
from celery import shared_task


@shared_task
def add(x, y):
    return x + y


@shared_task
def mul(x, y):
    return x * y


@shared_task
def xsum(numbers):
    return sum(numbers)
```

- 启停celery 要后台运行方式启动的，可以使用`supervisord`等管理程序进行启停管理。非后台方式启动为:`celery -A proj worker -l info` ；proj 为celery所在目录的目录名，不做修改的话就是项目名字。 这样启动的使用`ctrl + c`停止就可以。 [django集成celery官方示例](https://github.com/celery/celery/tree/master/examples/django)

------

## 积分系统实现尝试

### 思路

- django middleware中，在调用业务处理获得response后，可以通过访问request和response中的数据确定：

> 1. 本次调用的具体是哪个业务(request的METHOD和path一起来确定)

1. 对应的用户的信息(需要事先在中间件中将用户信息加入request)
2. 调用是否成功(response的状态或是定义的返回码等)
3. 返回的数据(从response中获取)

- 制定任务规则，在celery task中将对接口的调用抽象成一次任务，符合要求的即为完成某一件任务，并对完成的任务做记录。

> 签到接口调用:调用完成后，在task中判断，如果此次签到调用成功，则记录该时间点上完成某个用户完成一次签到任务

- 制定针对任务的积分奖励规则，在规定时间内完成多少次任务就可以获得积分。并用刚完成的任务对应奖励规则，记录该项规则的任务完成进度，任务完成的记录积分获取记录。

> 将奖励规则制定为: 完成一次签到任务可以获得1点积分，连续完成十次签到任务可以额外获得2点积分。则此次签到任务完成后，会对应去判断这两条规则。第一条肯定直接满足，记录下该条规则下签到获取积分进度为完成，并记录积分获取记录。第二条如果不满足，记录下该规则下调到积分获取进度为未完成，未完成的就没有积分获取记录了。

### 库表设计(精简版)

```python
from django.db import models


class TaskConfig(models.Model):
    """任务配置表
    task_type 任务类型
    task_desc 任务描述
    method 方法(GET, POST, ...)
    interface_name 接口名
    id_group_name 业务主键分组名字(正则式中的分组名称)
    status 状态 (编辑中， 使用中， 废弃)
    created_at 创建时间
    deleted_at 删除时间
    updated_at 修改时间
    """
    

class UserTaskRecord(models.Model):
    """用户任务记录表
    user 用户
    task 任务类型(外键)
    business_id 业务主键(创建博客的就是哪条博客的ID)
    created_at 创建时间
    updated_at 修改时间
    delete_at 删除时间
    """
    
    
class ScoreRule(SoftDeleteModel):
    """积分规表
    task_type 任务类型
    score_type 积分规则类型
    score_desc 积分规则描述
    statistical_period 统计周期(每天，每周，每月，每年，无周期，...)
    required_number_of_times 所需次数
    number_limit_all 获取人数上限
    start_time 规则生效时间
    end_time 规则截止时间
    status 规则状态(编辑中和已生效)
    can_repeat 任务内容是否可重复(默认为可重复)
    created_at 创建时间
    deleted_at 删除时间
    updated_at 修改时间
    """


class UserScoreInfo(SoftDeleteModel):
    """用户积分信息表
    user 用户
    achievement_type 积分规则类型(外键)
    required_number 所需任务量
    achievement_amount_now 当任务就量
    achieved 达成标志
    created_at 创建时间
    deadline 计算截止时间
    updated_at 修改时间
    deleted_at 删除时间
    """
    

class ScoreRecord(SoftDeleteModel):
    """积分记录
    profile 账户
    user 用户
    serial_number 流水号
    trade_amount 积分
    amount_before_trade 计算前账户积分量
    amount_after_trade 计算后账户积分量
    business_type 业务类型(积分规则描述)
    score_info 积分信息(外键)
    location 地点
    country 国家
    lang_type 语言
    created_at 创建时间
    updated_at 修改时间
    deleted_at 删除时间
    """
```

到这里一个简陋的积分系统就基本完成了，不过这样制定的规则会有很多限制，比如分享博客可以获得积分这种，如果针对某个具体的博客分享后所获得积分量不同，这就需要更进一步的设计。
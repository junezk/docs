# 详解django-apscheduler的使用方法

如果你在使用Django框架开发web项目时，需要设置定时任务或让用户手动在页面上设置定时任务，那么这篇文章可能会帮助到你。

# 概述

在Django中实现定时任务功能大概有以下三种方法：

### Celery框架

定时任务是分布式任务的一种特殊类型的任务。Django的分布式主要由Celery框架实现，这是python开发的分布式任务队列。由于它本身不支持消息存储服务，所以需要第三方消息服务来传递任务，一般使用Redis。

优点：

1. Celery侧重于实时操作，可用于生产系统每天处理数以百万计的任务，可用于大型项目。
2. 可在分布的机器、进程、线程上执行任务调度。

缺点：

配置和使用较为复杂，需要Redis数据库和多个python第三方库。

### django-crontab

只需要下载一个 django-crontab 包就可以使用cron表达式在Django框架中设置定时任务。本人对这种方法了解不多，不过这种方法好像不支持windows系统，功能也相对简单。

### django-apscheduler

配置简单、功能齐全、使用灵活、支持windows和linux，适合中小型项目。

# 使用方法

django-apscheduler中相关的概念和python的定时任务框架apscheduler中的概念是一样的，有感兴趣的同学可以自行查阅。

(本文使用 django + mysql 架构)

## 安装模块

```
pip install django-apscheduler
```

## 配置

1. 先在settings.py中配置好数据库信息(略).
2. 在**INSTALLED_APPS**中加入**django-apscheduler**应用：

```
    INSTALLED_APPS = [
        ...
        'django_apscheduler',
        ...
    ]
```

## 执行迁移

```
python manage.py migrate
```

去数据库中看一看，生成了两个表格，大部分都顾名思义。

**1. django_apscheduler_djangojob**

用于存储任务的表格

![img](详解django-apscheduler的使用方法.assets/16bbd5c0a7fb0e76)

job_state: 我猜是将任务具体的执行代码和参数进行序列化后存在了这里

**2. django_apscheduler_djangojobexecution**

用于存储任务执行状态的表格

![img](详解django-apscheduler的使用方法.assets/16bbd61010d1091e)

 status: 执行状态
duration: 执行了多长时间
exception: 是否出现了什么异常 

# 使用

## 创建任务

大概有两种创建任务的方法：装饰器和add_job函数。

### 1. 装饰器

在任意**view.py**中实现代码（我习惯新开一个app专门实现定时任务）：

```
from apscheduler.schedulers.background import BackgroundScheduler
from django_apscheduler.jobstores import DjangoJobStore, register_events, register_job

# 实例化调度器
scheduler = BackgroundScheduler()
# 调度器使用默认的DjangoJobStore()
scheduler.add_jobstore(DjangoJobStore(), 'default')

# 每天8点半执行这个任务
@register_job(scheduler, 'cron', id='test', hour=8, minute=30，args=['test'])
def test(s):
    # 具体要执行的代码
    pass

# 注册定时任务并开始
register_events(scheduler)
scheduler.start()
```

启动服务 `python manage.py runserver` 这个任务就会被存储到**django_apscheduler_djangojob**表中，并按照设置定时的执行程序。

#### 参数

- scheduler: 指定调度器
- trigger: 任务执行的方式，共有三种： 'date'、'interval'、'cron'。
  - 'date' + 'run_date' 的参数组合, 能实现**单次任务**。
    例子： 2019-07-07 22:49:00 执行任务
    `@register_job(scheduler, 'date', id='test', run_date='2019-07-07 22:49:00')`
    注：在亲测时，执行完任务会报错，原因时执行完任务后会去mysql中删除djangojob表中的任务。但是djangojobexecution表记录着执行结果，有外键关联着djangojob表，所以删除时显示有外键约束错误。但是任务会正常执行，执行之后也会正常删除。
  - 'interval' + 'hours' + 'minutes' + ..... 的参数组合，能实现**间隔性任务**。
    例子：每隔3个半小时执行任务
    `@register_job(scheduler, 'interval', id='test', hours=3, minutes=30)`
    还有seconds,days参数可以选择
    注：如果任务需要执行10秒，而间隔设置为1秒，它是不会给你开10个线程同时去执行10个任务的。它会错过其他任务直到当前任务完成。
  - 'cron' + 'hour' + 'minute'+...的参数组合，能实现**cron类的任务**。
    例子：每天的8点半执行任务
    `@register_job(scheduler, 'cron', id='test', hour=8, minute=30)`
    还有day,second,month等参数可以选择。
- id: 任务的名字，不传的话会自动生成。不过为了之后对任务进行暂停、开启、删除等操作，建议给一个名字。并且是唯一的，如果多个任务取一个名字，之前的任务就会被覆盖。
- args: list类型。执行代码所需要的参数。
- next_run_time：datetime类型。开始执行时间。如果你现在创建一个定时任务，想3天后凌晨三点半自动给你女朋友发微信，那就需要这个参数了。

还有些其他的参数感兴趣的同学可以查看源代码来了解。

### 2. add_job函数

装饰器的方法适合于写代码的人自己创建任务，如果想让用户通过页面输入参数，并提交来手动创建定时任务，就需要使用add_job函数。
下面这个小例子，前端传递json数据给后端，触发test_add_task函数，来添加任务：

```
import json
from django.http import JsonResponse
from apscheduler.schedulers.background import BackgroundScheduler
from django_apscheduler.jobstores import DjangoJobStore, register_events, register_job


scheduler = BackgroundScheduler()
scheduler.add_jobstore(DjangoJobStore(), 'default')

# 与前端的接口
def test_add_task(request):
    if request.method == 'POST':
        content = json.loads(request.body.decode())  # 接收参数
        try:
            start_time = content['start_time']  # 用户输入的任务开始时间, '10:00:00'
            start_time = start_time.split(':')
            hour = int(start_time)[0]
            minute = int(start_time)[1]
            second = int(start_time)[2]
            s = content['s']  # 接收执行任务的各种参数
            # 创建任务
            scheduler.add_job(test, 'cron', hour=hour, minute=minute, second=second, args=[s])
            code = '200'
            message = 'success'
        except Exception as e:
            code = '400'
            message = e
            
        back = {
            'code': code,
            'message': message
        }
        return JsonResponse(json.dumps(data, ensure_ascii=False), safe=False)
        
# 具体要执行的代码
def test(s):
    pass
    

register_events(scheduler)
scheduler.start()
```

这样就可以由前端用户来手动设置定时任务了。

#### 参数

和装饰器的参数大同小异，只是第一个参数不同。
如果具体要执行的函数和调用它的函数在一个文件中，那么只需要传递这个函数名就可以了（如上面的例子）。
但是我习惯将具体的业务代码写到另外一个文件中，view.py中只写前后端交互的接口函数，这种情况下传递的参数为一个字符串，格式为： 'package.module:some.object',即 包名.模块:函数名

## 其他功能

django-apscheduler框架还提供了很多操作定时任务的函数。比如：

- 删除任务
  `scheduler.remove_job(job_name)`
- 暂停任务
  `scheduler.pause_job(job_name)`
- 开启任务
  `scheduler.resume_job(job_name)`
- 修改任务
  `scheduler.modify_job(job_name)`
  注：修改任务只能修改参数，如果要修改执行时间的话，就把任务删了重新创建。

可以在页面上做一个这样的表格，再加上简单的前后端交互就可以让用户自行管理定时任务：

![img](详解django-apscheduler的使用方法.assets/16bd1dd35db20e4c)

其他的还有一些辅助功能（包括显示所有任务，显示任务的执行时间等），同学们可以自行查看。
最后，感谢阅读。
# Django - Celery异步任务队列

## 背景

在开发中，我们常常会遇到一些耗时任务，举个例子：

上传并解析一个 1w 条数据的 Excel 文件，最后持久化至数据库。

在我的程序中，这个任务耗时大约 6s，对于用户来说，6s 的等待已经是个灾难了。

比较好的处理方式是：

1. 接收这个任务的请求
2. 将这个任务添加到队列中
3. 立即返回「操作成功，正在后台处理」的字样
4. 后台消费这个队列，执行这个任务

我们按照这个思路，借助 Celery 进行实现。

## 实现

> 本文所使用的环境如下：
>
> - Python 3.6.7
> - RabbitMQ 3.8
> - Celery 4.3

### 使用 Docker 安装 RabbitMQ

Celery 依赖一个消息后端，可选方案有 RabbitMQ, Redis 等，本文选用 RabbitMQ 。

同时为了安装方便，RabbitMQ 我直接使用 Docker 安装：

```
docker run -d --name anno-rabbit -p 5672:5672 rabbitmq:3
```

启动成功后，即可通过 `amqp://localhost` 访问该消息队列。

### 安装并配置 Celery

Celery 是 Python 实现的工具，安装可以直接通过 Pip 完成：

```
pip install celery
```

> 同时假设当前我的项目文件夹为 `proj` ，项目名为 `myproj` ，应用名为 `myapp`

安装完成后，在 `proj/myproj/` 路径下创建一个 `celery.py` 文件，用来初始化 Celery 实例：

*proj/myproj/celery.py*

```
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery, platforms

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproj.settings')

app = Celery('myproj',
             broker='amqp://localhost//',
             backend='amqp://localhost//')

# Using a string here means the worker don't have to serialize
# the configuration object to child processes.s
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()
```

然后在 `proj/myproj/__init__.py` 中添加对 Celery 对象的引用，确保 Django 启动后能够初始化 Celery：

*proj/myproj/__init__.py*

```
from __future__ import absolute_import, unicode_literals

# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app

__all__ = ('celery_app',)
```

无其他特殊配置的话，Celery 的基本配置就是这些。

### 编写一个耗时任务

为了模拟一个耗时任务，我们直接创建一个方法，使其「睡」10s ，并将其设置为 Celery 的任务：

*proj/myapp/tasks.py*

```
import time
from myproj.celery import app as celery_app

@celery_app.task
def waste_time():
    time.sleep(10)
    return "Run function 'waste_time' finished."
```

### 启动 Celery Worker

Celery 配置完成，并且任务创建成功后，我们以**异步任务的模式**启动 Celery ：

```
celery -A myproj worker -l info
```

注意到我强调了**异步模式**，是因为 Celery 除了支持异步任务，还支持定时任务，因此启动时候要指明。

同时要注意，Celery 一旦启动，对 Task(此处为 `waste_time`) 的修改必须重启 Celery 才会生效。

### 任务调用

在请求处理的逻辑代码中，调用上面创建好的任务：

*proj/myapp/views.py*

```
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from .tasks import waste_time

@require_http_methods(["POST"])
def upload_files(request):
    waste_time.delay()
    # Status code 202: Accepted, 表示异步任务已接受，可能还在处理中
    return JsonResponse({"results": "操作成功，正在上传，请稍候..."}, status=202)
```

调用 `waste_time.delay()` 方法后， `waste_time` 会被加入到任务队列中，等待空闲的 Celery Worker 调用。

### 效果

当我们发送请求时，这个接口会直接返回 `{"results": "操作成功，正在上传，请稍候..."}` 的响应内容而非卡住十秒，用户体验要好许多。

## 总结

用 Celery 处理这种异步任务是 Python 常用的方法，虽然实际执行成功耗时不变甚至有所增加（如 Worker 繁忙导致处理滞后），但是对于用户体验来说更容易接受，点击上传大文件后可以继续处理其他事务，而不需要在页面等待。

Celery 还有更多用法本文未介绍到，其文档已经非常详尽，有需要可直接参考。

## 参考

- [docs.celeryproject.org/en/latest/d…](http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html)
- [hub.docker.com/_/rabbitmq](https://hub.docker.com/_/rabbitmq)
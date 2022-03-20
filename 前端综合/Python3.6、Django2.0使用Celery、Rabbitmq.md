# Python3.6、Django2.0使用Celery、Rabbitmq

Django使用Celery中间存储使用Rabbitmq

项目使用环境:

```
Python==3.6.2
Django==2.0.6
Celery==4.2.1
django-celery==3.2.2
django-celery-result==1.0.4
```


第一步：创建django项目demo以及项目下的应用home

```
django-admin.py startproject demo
python manage.py startapp home
第二步：在项目的主配置下demo/demo/settings.py文件中加入

BROKER_URL = 'amqp://guest:guest@localhost:5672//'  # 连接rabbitmq
```

第三步：在项目的主配置目录下创建文件demo/demo/celery.py

```python
# -*- coding:utf-8 -*-

from __future__ import absolute_import  # 注意此项必须放在最上方, 为了向上版本兼容

import os
from celery import Celery
from django.conf import settings

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'demo.settings')
django.setup()

app = Celery('demo')

# 自动加载每个应用(app)目录下的tasks.py文件

app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)
```

第四步：在项目的应用中创建异步任务队列home/tasks.py

```python
# -*- coding:utf-8 -*-
from demo.celery import app

@app.task
def print_hello():
    print('Hello World,fvf')
```

第五步：在项目的应用的视图函数中调用异步任务队列home/views.py

```python
# -*- coding:utf-8 -*-

from django.shortcuts import render,render_to_response
from django.http import HttpResponse

from .tasks import print_hello

def index(request):
    print_hello.delay()
    return HttpResponse(u"欢迎光临 红浪漫!")
```

第六步：在项目的应用的路由设置文件中添加路由home/urls.py

```python
# -*- coding:utf-8 -*-

from django.conf.urls import include, url
from home import views
urlpatterns = [
    url(r'^$', views.index),
]
```

第七步：在项目的主配置文件的初始化文件中增加以下内容home/home/__init__.py

```python
# -*- coding:utf-8 -*-

from home.settings.celery import app as celery_app
__all__ = ["celery_app"]
```

第八步：在项目的主目录demo/manage.py文件目录运行celery队列

```
celery -A demo worker --loglevel=info    # 其中demo为celery.py所在目录
```


第九步：在项目的主目录demo/manage.py文件目录启动django项目

```
python3 manage.py runserver 0.0.0.0:8008
```


第十步：返回消息队列的信息home/views.py

```python
# -*- coding:utf-8 -*-

from django.shortcuts import render,render_to_response
from django.http import HttpResponse
from .tasks import print_hello

def index(request):
    result = print_hello.delay()  
    id = result.task_id  # 获取消息队列任务id
    print print_hello.AsyncResult(id).get()  # 获取对应id的返回值
    return HttpResponse(u"欢迎光临 红浪漫!")
```

第十一步：返回任务存入django使用的数据库，项目主配置目录下settings.py文件增加配置

```
INSTALLED_APPS = [
    'django_celery_results'
]
CELERY_RESULT_BACKEND = 'django-db'
```

第十二步：同步存储队列任务结果的表

```
python3 manage.py migrate django_celery_result
```


第十三步：Celery提供了一个工具flower，将各个任务的执行情况、各个worker的健康状态进行监控并以可视化的方式展现

```
pip3 install flower
```


第十四步：启动flower

```
python3 manage.py celery flower
```


第十五步：网页查看，默认端口是5555

http://localhost:5555

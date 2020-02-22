# python3.7.0+Django2.0.4使用django-celery遇到的那些坑

首先大家知道Django是一个同步框架。
为了加快用户的响应时间，提高用户体验。所以决定采用异步任务的方式在后台执行一些任务
而且celery除了异步任务，还可以开启定时任务，方便调度

需要安装的软件包
最好按顺序安装

```bash
pip install celery
pip install celery-with-redis
pip install django-celery
```

因为async这个单词在python3.7以上版本中已经作为系统关键字存在了
所以要把所有涉及到这个关键字的文件都要改掉

因为async这个单词在python3.7以上版本中已经作为系统关键字存在了
所以要把所有涉及到这个关键字的文件都要改掉

```
  /kombu/async
　/celery/utils/timer2.py
　/concurrency/asynpool.py
　/kombu/transport/redis.py
　/celery/worker/auto_scale.py,components,consumer,strategy
```


基本就这些。有一个文件中包含关键字比较多，我记得是27个，替换的时候切记小心

接下来设置settings.py文件

```python
INSTALLED_APPS = (
　　...,
　　...,
　　...,
　　...,
　　...,
　 'djcelery'
)
　　
# 末尾初始化
import djcelery
djcelery.setup_loader()
BROKER_URL = 'redis://127.0.0.1:6379/0'
CELERY_IMPORTS = ('应用名称.task')
```

新增task.py

```python
　　#导入异步任务
　　from celery.task import task
　　#导入定时任务库
　　from celery.decorators import periodic_task
　　
　　#利用参数来设置任务周期
　　@periodic_task(run_every=10)
　　def some_task():
    　　print('每10秒执行一次')
    　　time.sleep(5)
    　　print('执行完毕')
    　　return True

　　#通过装饰器来注册异步任务
　　# 这里的sendmail是从别的文件中导入的，所以你们可以自己实现一些功能
　　@task
　　def task_mail():
    　　#实例化一个对象
    　　sendmail = SendMail(
    　　'欢迎注册',
    　　'您的验证码是1234',
    　　['邮箱号'],
    　　DEFAULT_FROM_EMAIL
    　　)
    status = sendmail.do_send_mail()
    if status:
        print('发送邮件成功')
    else:
        print('发送邮件失败')
```

新增celery.py

```python
import os
import django
from celery import Celery
from django.conf import settings

os.environ.setdefault('DJANGO_SETTINGS_MODULE', '项目.settings')
django.setup()
app = Celery('项目')
app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)

@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```


启动服务

```python
#异步服务
celery -A 项目名 worker -l info  

#定时任务服务
celery -A myproject beat -l info
```

但是在执行异步任务的时候发现服务总是自动断掉
我觉得是因为python库里的redis版本太高了
所以通过pip卸载，然后指定安装低版本2.6.10

```
pip uninstall redis
pip install redis==2.6.10
```

基本流程就这样

# Python Django Celery 实现异步任务

## 先从需求说起**

* 接口实现之后，需要异步跑自动化任务，因为自动化执行是耗时耗资源的操作，不可能在请求发出去后一直等待全部结果返回，所以需要实现异步。

* 环境：

    > Python3.6 , PyCharm, W7

* Celery介绍：

    * 安装：

        > pip install celery
        > pip install djcelery

    * [Celery DOC](http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html)

    * Celery是一个简单、灵活可靠的，处理大量消息的分布式系统，它是一个专注于实时处理的任务队列， 同时也支持任务调度。

    * Celery中有两个比较关键的概念

        > Worker: worker 是一个独立的进程，它持续监视队列中是否有需要处理的任务；
        >
        > Broker: broker 也被称为中间人或者协调者，broker 负责协调客户端和 worker 的沟通。客户端向 队列添加消息，broker 负责把消息派发给 worker。

* 项目结构：
    ![这里写图片描述](https://img-blog.csdn.net/20171103195437320?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGllXzA3MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 实现：

    * 编写代码：

        1. 修改项目setting.py配置

            ```

            # celery 配置

            djcelery.setup_loader()
            BROKER_URL = 'django://'  # 使用django做broker
            CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'  # 定时任务.
            CELERY_RESULT_BACKEND = 'djcelery.backends.database:DatabaseBackend'  # 需要跟踪任务的状态时保存结果和状态
            CELERY_ENABLE_UTC = False  # 不用UTC.
            CELERY_TIMEZONE = 'Asia/Shanghai'  # 指定上海时区
            CELERY_ACCEPT_CONTENT = ['pickle', 'json', 'msgpack', 'yaml']  # 允许的格式
            CELERY_TASK_SERIALIZER = 'json'
            CELERY_RESULT_SERIALIZER = 'json'
            CELERY_IGNORE_RESULT = True

            INSTALLED_APPS = [
                'djcelery',# 新增
                'kombu.transport.django',  # 新增kombu.transport.django则是基于Django的broker
                ]

            ```

        2. 在项目下新建celery.py 如上图

            ```
            from __future__ import absolute_import, unicode_literals
            import os
            from celery import Celery

            os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'AutoApi.settings')

            app = Celery('AutoApi')

            app.config_from_object('django.conf:settings')

            app.autodiscover_tasks()

            ```

        3. 在应用下新建tasks.py 如上图

            ```
            @shared_task
            def running(env, project, cases):
            print('开始执行异步任务') 
            print(env,project,cases)

            # test_cases_runing(env, project, cases) 启动自动化测试

            return 1
            ```

        4. 创建Celery所需的数据表

            > python manage.py migrate

        5. 启动Django服务

            > python manage.py runserver 0.0.0.0:8090

        6. 启动celery worker 服务

            > celery -A AutoApi worker -l info

        7. celer worker 启动成功提示

            > 提示 **[2017-11-03 20:30:17,526: WARNING/MainProcess] celery@SHANB040312 ready.**
            > 代码celery worker 服务启成功了

        8. postman 请求如下图,可以看到响应的时间是毫秒
            ![这里写图片描述](https://img-blog.csdn.net/20171103204227254?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGllXzA3MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

        9. 看下console celery worker 的输出,可以看到成功执行了我们的异步任务

            > [2017-11-03 20:39:25,234: INFO/MainProcess] Received task: Api.tasks.running[8a994909-a139-4f2a-bcdd-58b6280a2bc3]
            > [2017-11-03 20:39:31,740: WARNING/Worker-1] **开始执行异步任务**
            > [2017-11-03 20:39:31,740: WARNING/Worker-1] **sit**
            > [2017-11-03 20:39:31,741: WARNING/Worker-1] **zhengxin**
            > [2017-11-03 20:39:31,741: WARNING/Worker-1] **test_taiyue.py**
            > [2017-11-03 20:39:31,742: INFO/MainProcess] Task Api.tasks.running[8a994909-a139-4f2a-bcdd-58b6280a2bc3] **succeeded** in 0s: 1

* * *

其中有很多细节可以优化

*   broker 可以使用RabbitMQ 和 Redis
*   worker 可以再多启几个进程, 可以使用
*   可以启用 Celery Flower 它是一个 celery 的监控工具，它提供了一个图形用户界面，可以极大的方便我们监控任务的执行过程， 执行细节及历史记录，还提供了统计功能。

原理(网络资源)：

celery的模块架构

![这里写图片描述](https://img-blog.csdn.net/20171103205326634?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGllXzA3MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

工作原理
![这里写图片描述](https://img-blog.csdn.net/20171103205400331?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGllXzA3MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

以上就是我们的异步任务的实现，后面有机会再总结下定时任务的实现和使用RabbitMQ 。
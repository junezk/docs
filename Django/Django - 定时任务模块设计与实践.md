# Django - 定时任务模块设计与实践

## 背景

在开发后台与任务相关的功能中，遇到一个需求：用户需要能够为任务配置定时策略，使任务定时执行某个操作。

## 需求分析

根据需求，我们可以拆解成如下几个步骤：

1. 「某个操作」的实现
2. 配置为定时任务
3. 定时策略可配置
4. 用户体验好

其中步骤 1 与本文无关不提；对于定时任务的实现，在上节 [Celery异步任务队列 ](https://juejin.im/post/5d043d28e51d4510bf1d666e)有简单提到 celery 也支持定时任务。

Celery 的定时任务策略配置于代码中，在启动 celery 时写入本地 [shelve](https://docs.python.org/dev/library/shelve.html#module-shelve) 文件，不利于管理。

因此在 celery 的文档中也提到一个扩展模块 `django-celery-beat`，该模块将定时任务的配置写入 Django 配置的数据库中，当程序启动后可以通过 `admin` 后台进行管理，并且可以直接通过 ORM 对定时任务配置进行修改，无需修改代码然后重启 celery，符合我们预期。

当然还有很多其他库也能实现，因为我们已经使用 celery 执行异步任务，所以本文还是用 `django-celery-beat` 解决问题。

Celery 的定时任务使用的是类似 `crontab` 的语法，因此在用户体验上，要考虑普通用户的学习成本，可以提供一些常用的配置，例如每周的工作日每天 1 点执行任务；也要考虑后期的扩展性，可以提供输入框方便配置。

## 设计与实现

### 基本用法

#### 定时策略（CrontabSchedule）

`CrontabSchedule` 支持类 crontab 语法，同样是 5 个配置域，分别为：

- 分
- 时
- 每周中的天
- 每月中的天
- 每年中的月

每个配置域使用空格隔开。

对每个配置域常用语法：

- `*`: 范围内的所有值
- `M-N`: M到N之间的值
- `M-N/X` 或 `*/X`: 每X分钟、每X天等等
- `A,B,...,Z`: 枚举的值

举个例子： 每个工作日1点执行：`0 1 1-5 * *`

创建定时策略代码如下：

```
from django_celery_beat.models import CrontabSchedule, PeriodicTask
>>> schedule, _ = CrontabSchedule.objects.get_or_create(
...     minute='30',
...     hour='*',
...     day_of_week='*',
...     day_of_month='*',
...     month_of_year='*',
... )
```

#### 定时任务

定时任务可以依赖不同的定时策略，例如 crontab, interval 等，创建时指定 `schedule` 即可。以 crontab 定时任务为例：

```
>>> import json
>>> from datetime import datetime, timedelta

>>> PeriodicTask.objects.create(
...     crontab=schedule,                  # we created this above.
...     name='Importing contacts',          # simply describes this periodic task.
...     task='proj.tasks.import_contacts',  # name of task.
...     args=json.dumps(['arg1', 'arg2']),
...     kwargs=json.dumps({
...        'be_careful': True,
...     }),
...     expires=datetime.utcnow() + timedelta(seconds=30)
... )
```

其中 `name` 为定时任务的名称，每个任务名必须唯一；`task` 为需要执行的 celery 任务。加上定时策略调度器，这三个是一个定时任务所必须的属性。

定时任务还有其他配置，如 `args`/`kwargs` 对应一个 celery 任务的入参；`expires` 设置了该定时任务的过期时间。

### Django配置

最基础的配置只需要在 `INSTALLED_APPS` 中添加引用，并设置定时任务调度器即可：

*settings.py*

```
INSTALLED_APPS = [
    ...
    'django_celery_beat'
]

# 配置 celery 定时任务使用的调度器
CELERY_BEAT_SCHEDULER = 'django_celery_beat.schedulers:DatabaseScheduler'
```

#### 时区问题

在使用 `django-celery-beat` 过程中遇到两个关于时区的问题：

1. 创建的定时任务，实际触发时间与配置的时间存在8小时时间差

**解决方案：**

8小时明显是因为时区不同导致，而 `django-celery-beat` 对时区的处理似乎总有问题（若不对请指教）。

修改 `settings.py` 中的时区配置：

*settings.py*

```
# 设置 Django 大部分应用通用的时区
TIME_ZONE = 'Asia/Shanghai'
# 关闭 UTC
USE_TZ = False
CELERY_ENABLE_UTC = False
# 设置 django-celery-beat 真正使用的时区
CELERY_TIMEZONE = TIME_ZONE
# 使用 timezone naive 模式
DJANGO_CELERY_BEAT_TZ_AWARE = False
```

> 关于 timezone naive 与 timezone aware 模式的区别可以参考文章：[Django时区详解](https://blog.csdn.net/laughing2333/article/details/53513414)
>
> 简单来说就是，naive 模式不存储时区信息，只存储经过时区转换后的时间；反之 aware 模式则存储了 UTC 时间和 UTC 时区信息。

根据[文档](https://django-celery-beat.readthedocs.io/en/latest/#important-warning-about-time-zones)，在修改了时区后，需要将已执行过的定时任务的 `last_run_at` 重置为 `None`：

```
python manage.py shell
>>> from django_celery_beat.models import PeriodicTask
>>> PeriodicTask.objects.all().update(last_run_at=None)
```

修改完成后，重启 `celery beat`。

> PS: 就算是经过这样配置，我也仍然遇到了任务不断执行的问题，并且在我多次重启 celery 后不再复现，因此本配置可能还有问题。

1. 数据库中，`CrontabSchedule` 的 `timezone` 配置始终是 `UTC`

**解决方案：**

查看 `CrontabSchedule` 模型的源码，找到数据库中 `timezone` 字段的属性：

```
class CrontabSchedule(models.Model):
    ...
    timezone = timezone_field.TimeZoneField(
        default='UTC',
        verbose_name=_('Cron Timezone'),
        help_text=_(
            'Timezone to Run the Cron Schedule on.  Default is UTC.'),
    )
```

由于我们在创建 `CrontabSchedule` 实例时并未指定 `timezone`，因此在创建任务时，添加该字段的配置即可：

```
from django_celery_beat.models import CrontabSchedule
>>> schedule, _ = CrontabSchedule.objects.get_or_create(
...     minute='30',
...     hour='*',
...     day_of_week='*',
...     day_of_month='*',
...     month_of_year='*',
...     timezone='Asia/Shanghai'
... )
```

### *业务前后端设计

> 本节内容仅供参考，不一定适用其他场景。

#### 前端

设计前端定时任务配置项，包含一个开关，一个三选一单选组件，以及一个输入框：

 ![img](Django - 定时任务模块设计与实践.assets/16c1d6d53c87ef6e) 

为了方便非技术人员设置定时任务，优化用户体验，定时任务除了「自定义」的输入模式，还有一个「每天」与「每周」的选项：

- 每天：`0 1 1-5 * *`
- 每周：`0 1 1 * *`

单选框与字符串双向绑定，在后端返回上面两个字符串之一时选中每天或每周，否则选中自定义选项。

#### 后端

假设对于我的业务来说，前端需要的任务数据字段为：

```
{
    "task_id": 1,
    "is_periodic_task": true,
    "periodic_task_id": 1,
    "crontab": "* * * * *"
}
```

ER 模型如图：

 ![img](Django - 定时任务模块设计与实践.assets/16c1d6630990ee14) 

返回给前端的数据中，若 `periodic_task` 不为空，则 `is_periodic_task` 为 `True`，并通过 `periodic_task.crontab_id` 获取到 `CrontabSchedule` 实例，转化为字符串返回。

要注意，`CrontabSchedule` 的 `__str__` 方法除了返回 `crontab` 配置，还会返回时区等信息，而这些信息前端展示时并不需要。

因此可以新建一个方法：

```
def get_crontab_str(contab) -> str:
    """
    获取前端配置需要的 5 项值
    :param contab: CrontabSchedule对象
    :return:
    """
    return '{0} {1} {2} {3} {4}'.format(
        cronexp(contab.minute), cronexp(contab.hour),
        cronexp(contab.day_of_week), cronexp(contab.day_of_month),
        cronexp(contab.month_of_year)
    )
```

序列化时调用该方法返回给前端即可。

##### 修改任务

修改任务包括以下三种情况

1. 从定时任务改为非定时任务
2. 从非定时任务改为定时任务
3. 在定时任务基础上修改定时策略

对应流程图如下：

1:

 ![img](Django - 定时任务模块设计与实践.assets/16c1d87f48378763) 

2, 3:

 ![img](Django - 定时任务模块设计与实践.assets/16c1d88a7b797bea) 

> 图中「修改配置中的」指前端传来的修改请求中的新配置信息

具体代码就不赘述，只提一下暂停定时任务的方法：

修改 `PeriodicTask.objects.enabled` 为 `False/0` 即可

```
>>> periodic_task.enabled = False
>>> periodic_task.save()
```

## 总结

Django 实现网页端配置定时任务的功能实现大抵如此，可惜 Django 及其插件对时区的配置比较复杂，花了很多时间踩了很多坑都还是有些问题没能搞清楚。继续探索吧！

## 版本说明

| 框架/服务/组件     | 版本  | 说明 |
| ------------------ | ----- | ---- |
| Python             | 3.6.7 |      |
| Django             | 2.2   |      |
| RabbitMQ           | 3     |      |
| Celery             | 4.3   |      |
| django-celery-beat | 1.5.0 |      |

## 参考

- [docs.celeryproject.org/en/latest/u…](http://docs.celeryproject.org/en/latest/userguide/periodic-tasks.html)
- [django-celery-beat.readthedocs.io/en/latest/](https://django-celery-beat.readthedocs.io/en/latest/)
- [docs.djangoproject.com/en/2.2/topi…](https://docs.djangoproject.com/en/2.2/topics/i18n/timezones/)
- [blog.csdn.net/laughing233…](https://blog.csdn.net/laughing2333/article/details/53513414)
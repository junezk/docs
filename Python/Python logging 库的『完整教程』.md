# Python logging 库的『完整教程』

## 前言

本文的标题是『完整』。所谓『完整』，大意是想表达：提炼出一组最小的经验组合，并且能够快速应用于工程中，能 work，甚至能完美地 work。这篇文章就是想要做到『如何能完美地work』。

## 初衷

最原始的初衷就是：『现在的，是不完美的，不系统的』。

一是：Python 官方对 logging 这个库的使用，介绍得不够“ 透彻 ”。我们能在官方文档中找到关于 logging 库的有价值的东西，大概是如下几样：

1. 库文档，主要是介绍一些class，以及logging 库是如何组成的，中间穿插了一些零碎的使用方法，但是仍然没有系统介绍如何使用。它们分别是：
   - 16.6. logging — Logging facility for Python
   - 16.7. logging.config — Logging configuration
   - 16.8. logging.handlers — Logging handlers
2. 两篇 HOWTOs。事无巨细地介绍logging 这个库的组成和使用，优点很明显：事无巨细，基本覆盖到了。缺点也很明显：事无巨细，各个部分平均发力，让人找不到重点，很容易迷失在其中，看了和没看一样。它们分别是：
   - Logging HOWTO
   - Logging Cookbook

二是：新手大概需要多年的经验磨合，才会知道，日志才是调试最好的手段。日常开发中，单步调试 VS 日志调试的比例大概是 1 ：9（我个人的是 0：10 ）。新手一般喜欢使用单步调试，或是基于 print 的调试，这两者都是效率比较低下的，下面分别介绍：

- print调试。不想介绍了，缺点比较多，我不说，大家也懂。适用于临时性使用。
- 单步调试。优点明显：能单步，能看清每一步的状况。缺点也很明显：效率低，多线程情况下比较无解。适用于小范围使用。

## 现在的情况

现在的情况，如同上面所说，大家没有充分重视日志调试的作用，官方的logging库亦是缺乏比较系统的『最小可用教程』。

## HOW TO DO

先从需求出发，即：从调用端推导接口设计。

调用端大概率喜欢这么使用（作为调用端，一般都希望接口越简单越小越好）：

```
log_factory.SOME_LOGGER.info('MY LOG MSG')
```

这样，我们可以把『log_factory』弄成一个 package（module也可以，不过我很喜欢 Go 语言那种看似麻烦实则规范的『基于package组织项目』的原则），『SOME_LOGGER』我们可以使用单例，不过Python有全局变量这种东西，我们可以使用全局变量。

另外，一个比较直觉的想法是：日志应该要有对应的配置文件，不过Python是脚本语言，**脚本语言的源码文件，天生就是配置文件（因为脚本语言一般不需要编译，改改源码就能快速上线验证，而且还可以热更新）**。

这样，我们的目录可以如此规划：

```
common_libs/
    __init__.py
    log_factory/
        __init__.py       
        代码可以直接写在这里，或者拆分成多个 py 文件，反正对外也就提供一个『log_factory』的命名空间
```

主要内容如下：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#
# author:      he.zhiming
#

from __future__ import unicode_literals, absolute_import

import logging
import logging.config
import logging.handlers
from datetime import datetime
import os


class _InfoFilter(logging.Filter):
    def filter(self, record):
        """only use INFO

        筛选, 只需要 INFO 级别的log

        :param record:
        :return:
        """
        if logging.INFO <= record.levelno < logging.ERROR:
            # 已经是INFO级别了
            # 然后利用父类, 返回 1
            return super().filter(record)
        else:
            return 0


def _get_filename(*, basename='app.log', log_level='info'):
    date_str = datetime.today().strftime('%Y%m%d')
    pidstr = str(os.getpid())
    return ''.join((
        date_str, '-', pidstr, '-', log_level, '-', basename,))


class _LogFactory:
    # 每个日志文件，使用 2GB
    _SINGLE_FILE_MAX_BYTES = 2 * 1024 * 1024 * 1024
    # 轮转数量是 10 个
    _BACKUP_COUNT = 10

    # 基于 dictConfig，做再次封装
    _LOG_CONFIG_DICT = {
        'version': 1,

        'disable_existing_loggers': False,

        'formatters': {
            # 开发环境下的配置
            'dev': {
                'class': 'logging.Formatter',
                'format': ('%(levelname)s %(asctime)s %(created)f %(name)s %(module)s [%(processName)s %(threadName)s] '
                           '[%(filename)s %(lineno)s %(funcName)s] %(message)s')
            },
            # 生产环境下的格式(越详细越好)
            'prod': {
                'class': 'logging.Formatter',
                'format': ('%(levelname)s %(asctime)s %(created)f %(name)s %(module)s %(process)d %(thread)d '
                           '%(filename)s %(lineno)s %(funcName)s %(message)s')
            }

            # ? 使用UTC时间!!!

        },

        # 针对 LogRecord 的筛选器
        'filters': {
            'info_filter': {
                '()': _InfoFilter,

            }
        },

        # 处理器(被loggers使用)
        'handlers': {
            'console': {  # 按理来说, console只收集ERROR级别的较好
                'class': 'logging.StreamHandler',
                'level': 'ERROR',
                'formatter': 'dev'
            },

            'file': {
                'level': 'INFO',
                'class': 'logging.handlers.RotatingFileHandler',
                'filename': _get_filename(log_level='info'),
                'maxBytes': _SINGLE_FILE_MAX_BYTES,  # 2GB
                'encoding': 'UTF-8',
                'backupCount': _BACKUP_COUNT,
                'formatter': 'dev',
                'delay': True,    
                'filters': ['info_filter', ]  # only INFO, no ERROR            
            },
            'file_error': {
                'level': 'ERROR',
                'class': 'logging.handlers.RotatingFileHandler',
                'filename': _get_filename(log_level='error'),
                'maxBytes': _SINGLE_FILE_MAX_BYTES,  # 2GB
                'encoding': 'UTF-8',
                'backupCount': _BACKUP_COUNT,
                'formatter': 'dev',
                'delay': True,                
            },

        },

        # 真正的logger(by name), 可以有丰富的配置
        'loggers': {
            'SAMPLE_LOGGER': {
                 # 输送到3个handler，它们的作用分别如下
                 #   1. console：控制台输出，方便我们直接查看，只记录ERROR以上的日志就好
                 #   2. file： 输送到文件，记录INFO以上的日志，方便日后回溯分析
                 #   3. file_error：输送到文件（与上面相同），但是只记录ERROR级别以上的日志，方便研发人员排错
                'handlers': ['console', file', 'file_error'],
                'level': 'INFO'
            },
        },
    }

    logging.config.dictConfig(_LOG_CONFIG_DICT)

    @classmethod
    def get_logger(cls, logger_name):
        return logging.getLogger(logger_name)

# 一个示例
SAMPLE_LOGGER = _LogFactory.get_logger('SAMPLE_LOGGER')
# 示例——debugger，需要先配置好（如同SAMPLE_LOGGER一样）
DEBUGGER = _LogFactory.get_logger('CONSOLE')
# 软件项目一般是分层的，所以可以每一层放置一个logger，各司其职，这里是一个示例
SOME_BASE_LIB_LOGGER = _LogFactory.get_logger('SOME_BASE_LIB_LOGGER')
```

## 几个最佳实践点

### 同一个logger配置多个handler

有个handler适合临时排错，有的handler适合永久记录，有的handler记录得事无巨细，有的handler仅仅记录关心的内容（如ERROR)

### 内容格式的设计

有如下几点诉求：

1. 能追踪到进程、线程（必须要能追踪到线程，多线程必备）
2. 能追踪到出错行数
3. 格式很规范统一

所以我们设计如下格式：

```
INFO 2018-05-18 16:42:56,637 1526632976.637384 DEBUGGER __main__ 73580 52688
__main__.py 29 test_func GOT RESULT. ['HELLO-WORLD FROM logginglib_project.business_layer.core.CoreUtils#get_hellowolrd']

分别对应
level date_time timestamp logger_name python_module process_id thread_id filename line_number function_name log_message
```

### 日志文件名设置

比如 `20180518-73580-info-debug_INFO.log`，不用使用大脑思考，就知道是个什么文件

## 光说不练假把式

光说不练假把式，自己写的代码，要『eat dog food』，演示如下：

![img](Python logging 库的『完整教程』.assets/16372669e5cb86c8)

![img](Python logging 库的『完整教程』.assets/163726f2002124e5)

## 更加高级的需求

### INFO级别的handler，只使用『INFO <= && < ERROR』的日志

利用logging库提供的Filter概念，可以轻松实现：

```python
先实现 Filter
class _InfoFilter(logging.Filter):
    def filter(self, record):
        """only use INFO

        筛选, 只需要 INFO 级别的log

        :param record:
        :return:
        """
        if logging.INFO <= record.levelno < logging.ERROR:
            # 已经是INFO级别了
            # 然后利用父类, 返回 1
            return super().filter(record)
        else:
            return 0

然后适配到Handler上面
'file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': _get_filename(log_level='info'),
            'maxBytes': _SINGLE_FILE_MAX_BYTES,  # 2GB
            'encoding': 'UTF-8',
            'backupCount': _BACKUP_COUNT,
            'formatter': 'dev',
            'delay': True,    
            'filters': ['info_filter', ]  # only INFO, no ERROR            
        },
```

## 其他的注意事项

### 并不不一定适合框架（提供了完整日志规范的），如Django

Django 提供的完整的日志规范，一般的Django项目，按照框架的规范来即可（如Django，需要在settings文件中配置 LOGGING)。

### 应该配置多少logger

一般按照自己的需求来，我有一个特别好的方式：按照软件项目的分层结构来（软件是分层的，应该是常识），每一层配置一个logger，这样就不会混乱。

### logging库是进程安全，或者线程安全的吗

是线程安全的，但不是进程安全的。但是可以很轻易地解决这点，即：每个文件名带一个pid即可，让每一个进程始终对应只属于自己的文件（见_get_filename的使用）。

## 固化我们的成果

变成 GitHub 上面的仓库，欢迎 star。

链接：https://github.com/hezhiming/py_logging_usage/tree/master
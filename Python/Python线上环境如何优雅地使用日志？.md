# Python线上环境如何优雅地使用日志？

## 瞎比比

这篇文章其实早在一个月之前就写好了。奈何，加班猛如虎，真的怕了。直至今天才幸运地有了个双休，赶紧排版一下文章发布了。以下为正文。 在初学 Python 的时候，我们使用

```python
print("hello world")
```

输出了我们的第一行代码。在之后的日子里，便一直使用 print 进行调试（当然，还有 IDE 的 debug 模式）。但是，当你在线上运行 Python 脚本的时候，你并不可能一直守着你的运行终端。可是如果不守着的话，每当出现 bug ，错误又无从查起。这个时候，你需要对你的调试工具进行更新换代了，这里我推荐一个优雅的调试工具 logging。

## 与 print 相比 logging 有什么优势？

那既然我推荐这个工具，它凭什么要被推荐呢？且来看看它有什么优势：

- 可以输出到多处，例如：在输出到控制台的同时，可以保存日志到日志文件里面，或者保存到其他远程服务器
- 可以设置日志等级，DEBUG、INFO、ERROR等，在不同的环境（调试环境、线上环境）使用不同的等级来过滤日志，使用起来很方便
- 配置灵活，可保存到配置文件，格式化输出

## 基础用法

#### 下面涉及到的代码我都省略了导包部分，详见源码(后台回复 logging 获取源码)

#### base_usage.py

```python
logging.basicConfig(level=log_level, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

logger.info("Log level info")
logger.debug("Log level debug")
logger.warning("Log level warning")

# 捕获异常，并打印出出错行数
try:
    raise Exception("my exception")
except (SystemExit, KeyboardInterrupt):
    raise
except Exception:
    logger.error("there is an error =>", exc_info=True)
```

**level** 为日志等级，分为：

```
FATAL：致命错误
CRITICAL：特别糟糕的事情，如内存耗尽、磁盘空间为空，一般很少使用
ERROR：发生错误时，如IO操作失败或者连接问题
WARNING：发生很重要的事件，但是并不是错误时，如用户登录密码错误
INFO：处理请求或者状态变化等日常事务
DEBUG：调试过程中使用DEBUG等级，如算法中每个循环的中间状态
```

**foamat** 可以格式化输出，其参数有如下：

```
%(levelno)s：打印日志级别的数值
%(levelname)s：打印日志级别的名称
%(pathname)s：打印当前执行程序的路径，其实就是sys.argv[0]
%(filename)s：打印当前执行程序名
%(funcName)s：打印日志的当前函数
%(lineno)d：打印日志的当前行号
%(asctime)s：打印日志的时间
%(thread)d：打印线程ID
%(threadName)s：打印线程名称
%(process)d：打印进程ID
%(message)s：打印日志信息
```

**捕获异常**,以下两行代码都具有相同的作用

```python
logger.exception(msg,_args)
logger.error(msg,exc_info = True,_args)
```

## 保存到文件，并输出到命令行

这个用法直接 copy 使用就行

```python
import logging
# 写入文件
import logging

logger = logging.getLogger(__name__)
logger.setLevel(level=logging.INFO)
handler = logging.FileHandler("info.log")
handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)

logger.info("Log level info")
logger.debug("Log level debug")
logger.warning("Log level warning")


# 写入文件，同时输出到屏幕

import logging
logger = logging.getLogger(__name__)
logger.setLevel(level = logging.INFO)
handler = logging.FileHandler("info.log")
handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

console = logging.StreamHandler()
console.setLevel(logging.INFO)

logger.addHandler(handler)
logger.addHandler(console)

logger.info("Log level info")
logger.debug("Log level debug")
logger.warning("Log level warning")
```

## 多模块使用 logging

被调用者的日志格式会与调用者的日志格式一样 main.py

```python
# -*- coding: utf-8 -*-
__auth__ = 'zone'
__date__ = '2019/6/17 下午11:46'
'''
公众号：zone7
小程序：编程面试题库
'''
import os
import logging

from python.logging_model.code import sub_of_main

logger = logging.getLogger("zone7Model")
logger.setLevel(level=logging.INFO)
handler = logging.FileHandler("log.txt")
handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(formatter)

logger.addHandler(handler)
logger.addHandler(console)

sub = sub_of_main.SubOfMain()
logger.info("main module log")
sub.print_some_log()
```

sub_of_main.py

```python
# -*- coding: utf-8 -*-
__auth__ = 'zone'
__date__ = '2019/6/17 下午11:47'
'''
公众号：zone7
小程序：编程面试题库
'''
import logging
module_logger = logging.getLogger("zone7Model.sub.module")
class SubOfMain(object):
    def __init__(self):
        self.logger = logging.getLogger("zone7Model.sub.module")
        self.logger.info("init sub class")

    def print_some_log(self):
        self.logger.info("sub class log is printed")


def som_function():
    module_logger.info("call function some_function")
```

## 使用配置文件配置 logging

这里分别给出了两种配置文件的使用案例，都分别使用了三种输出，输出到命令行、输出到文件、将错误信息独立输出到一个文件

#### log_cfg.json

```json
{
    "version":1,
    "disable_existing_loggers":false,
    "formatters":{
        "simple":{
            "format":"%(asctime)s - %(name)s - %(levelname)s - %(message)s"
        }
    },
    "handlers":{
        "console":{
            "class":"logging.StreamHandler",
            "level":"DEBUG",
            "formatter":"simple",
            "stream":"ext://sys.stdout"
        },
        "info_file_handler":{
            "class":"logging.handlers.RotatingFileHandler",
            "level":"INFO",
            "formatter":"simple",
            "filename":"info.log",
            "maxBytes":10485760,
            "backupCount":20,
            "encoding":"utf8"
        },
        "error_file_handler":{
            "class":"logging.handlers.RotatingFileHandler",
            "level":"ERROR",
            "formatter":"simple",
            "filename":"errors.log",
            "maxBytes":10485760,
            "backupCount":20,
            "encoding":"utf8"
        }
    },
    "loggers":{
        "my_module":{
            "level":"ERROR",
            "handlers":["info_file_handler2"],
            "propagate":"no"
        }
    },
    "root":{
        "level":"INFO",
        "handlers":["console","info_file_handler","error_file_handler"]
    }
}
```

通过 json 文件读取配置：

```python
import json
import logging.config
import os


def set_log_cfg(default_path="log_cfg.json", default_level=logging.INFO, env_key="LOG_CFG"):
    path = default_path
    value = os.getenv(env_key, None)
    if value:
        path = value
    if os.path.exists(path):
        with open(path, "r") as f:
            config = json.load(f)
            logging.config.dictConfig(config)
    else:
        logging.basicConfig(level=default_level)


def record_some_thing():
    logging.info("Log level info")
    logging.debug("Log level debug")
    logging.warning("Log level warning")


if __name__ == "__main__":
    set_log_cfg(default_path="log_cfg.json")
    record_some_thing()
```

#### log_cfg.yaml

```yaml
version: 1
disable_existing_loggers: False
formatters:
        simple:
            format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
handlers:
    console:
            class: logging.StreamHandler
            level: DEBUG
            formatter: simple
            stream: ext://sys.stdout
    info_file_handler:
            class: logging.handlers.RotatingFileHandler
            level: INFO
            formatter: simple
            filename: info.log
            maxBytes: 10485760
            backupCount: 20
            encoding: utf8
    error_file_handler:
            class: logging.handlers.RotatingFileHandler
            level: ERROR
            formatter: simple
            filename: errors.log
            maxBytes: 10485760
            backupCount: 20
            encoding: utf8
loggers:
    my_module:
            level: ERROR
            handlers: [info_file_handler]
            propagate: no
root:
    level: INFO
    handlers: [console,info_file_handler,error_file_handler]
```

通过 yaml 文件读取配置：

```python
import yaml
import logging.config
import os

def set_log_cfg(default_path="log_cfg.yaml", default_level=logging.INFO, env_key="LOG_CFG"):
    path = default_path
    value = os.getenv(env_key, None)
    if value:
        path = value
    if os.path.exists(path):
        with open(path, "r") as f:
            config = yaml.load(f)
            logging.config.dictConfig(config)
    else:
        logging.basicConfig(level=default_level)


def record_some_thing():
    logging.info("Log level info")
    logging.debug("Log level debug")
    logging.warning("Log level warning")


if __name__ == "__main__":
    set_log_cfg(default_path="log_cfg.yaml")
    record_some_thing()
```
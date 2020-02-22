# Python日志模块-多进程日志记录

## 一、问题描述 

项目中，使用`RotatingFileHandler`根据日志文件大小来切分日志。设置文件的`MaxBytes`为`1GB`， `backupCount`大小为5。

经查看，发现日志文件的大小均小于`10MB`，且每个回滚日志文件的写入时间也都比较接近。

## 二、分析 

日志文件过小，猜测是代码有问题，或者是文件内容有丢失；日志写入时间接近猜测是同时写入的问题。

经检查，代码没有问题，排除此原因。考虑当前使用`gunicorn`的多进程启动程序，多半是多个进程同时写入当个文件造成日志文件丢失。

`logging`模块是线程安全的，但并不是进程安全的。

如何解决此问题呢？首先先过一遍`Python`的`logging`模块在处理日志回滚的具体实现方法。

### 2.1 logging模块实现日志回滚

`logging`中`RotatingFileHandler`类和`TimedRotatingFileHandler`类分别实现按照日志文件大小和日志文件时间来切分文件，均继承自`BaseRotatingHandler`类。

`BaseRotatingHandler`类中实现了文件切分的触发和执行，具体过程如下：

```python
def emit(self, record):
    """
        Emit a record.
        Output the record to the file, catering for rollover as described
        in doRollover().
        """
    try:
        if self.shouldRollover(record):
            self.doRollover()
        logging.FileHandler.emit(self, record)
    except Exception:
        self.handleError(record)
```

具体的执行过程`shouldRollover(record)`和`doRollover()`函数则在`RotatingFileHandler`类和`TimedRotatingFileHandler`类中实现。

以`RotatingFileHandler`类为例，`doRollover()`函数流程如下：

```python
def doRollover(self):
    if self.stream:
        self.stream.close()
        self.stream = None
    if self.backupCount > 0:
        for i in range(self.backupCount - 1, 0, -1): # 从backupCount，依次到1
            sfn = self.rotation_filename("%s.%d" % (self.baseFilename, i))
            dfn = self.rotation_filename("%s.%d" % (self.baseFilename,
                                                        i + 1))
            if os.path.exists(sfn):
                if os.path.exists(dfn):
                    os.remove(dfn)
                os.rename(sfn, dfn) # 实现将xx.log.i->xx.log.i+1
        dfn = self.rotation_filename(self.baseFilename + ".1")
        # ---------start-----------
        if os.path.exists(dfn): # 判断如果xx.log.1存在，则删除xx.log.1
            os.remove(dfn)
        self.rotate(self.baseFilename, dfn) # 将xx.log->xx.log.1
        # ----------end------------
    if not self.delay:
        self.stream = self._open() # 执行新的xx.log
```

分析如上过程，整个步骤是：

1. 当前正在处理的日志文件名为`self.baseFilename`，该值`self.baseFilename = os.path.abspath(filename)`是设置的日志文件的绝对路径，假设`baseFilename`为`error.log`。
2. 当进行文件回滚的时候，会依次将`error.log.i`重命名为`error.log.i+1`。
3. 判断`error.log.1`是否存在，若存在，则删除，将当前日志文件`error.log`重命名为`error.log.1`。
4. `self.stream`重新指向新建`error.log`文件。

当程序启动多进程时，每个进程都会执行`doRollover`过程，若有多个进程进入临界区，则会导致`dfn`被删除多次等多种混乱操作。

### 2.2 多进程日志安全输出到同一文件方案

相应的解决方法：

1. 将日志发送到同一个进程中，由该进程负责输出到文件中（使用`Queue`和`QueueHandler`将所有日志事件发送至一个进程中）
2. 对日志输出加锁，每个进程在执行日志输出时先获得锁（用多处理模块中的`Lock`类来序列化对进程的文件访问）
3. 让所有进程都将日志记录至一个`SocketHandler`，然后用一个实现了套接字服务器的单独进程一边从套接字中读取一边将日志记录至文件（`Python`手册中提供）

## 三、解决方案 

### 3.1 使用ConcurrentRotatingFileHandler包

> 该方法就属于加锁方案。

`ConcurrentLogHandler` 可以在多进程环境下安全的将日志写入到同一个文件，并且可以在日志文件达到特定大小时，分割日志文件（***支持按文件大小分割\***）。但`ConcurrentLogHandler` 不支持按时间分割日志文件的方式。

`ConcurrentLogHandler` 模块使用文件锁定，以便多个进程同时记录到单个文件，而不会破坏日志事件。该模块提供与`RotatingFileHandler`类似的文件循环方案。

该模块尝试不惜一切代价保存记录，这意味着日志文件将大于指定的最大大小（如果磁盘空间不足，则坚持使用`RotatingFileHandler`，因为它是严格遵守最大文件大小），如果有多个脚本的实例同时运行并写入同一个日志文件，那么所有脚本都应该使用`ConcurrentLogHandler`，不应该混合和匹配这这个类。

并发访问通过使用**文件锁**来处理，该文件锁应确保日志消息不会被丢弃或破坏。这意味着将为写入磁盘的每个日志消息获取并释放文件锁。（在Windows上，您可能还会遇到临时情况，必须为每个日志消息打开和关闭日志文件。）这可能会影响性能。在我的测试中，性能绰绰有余，但是如果您需要大容量或低延迟的解决方案，建议您将其放在其他地方。

`ConcurrentRotatingFileLogHandler`类是python标准日志处理程序`RotatingFileHandler`的直接替代。

这个包捆绑了`portalocker`来处理文件锁定。由于使用了`portalocker`模块，该模块当前仅支持`“nt”`和`“posix”`平台。

**安装：**

```
pip install ConcurrentLogHandler
```

> 该模块支持`Python2.6`及以后版本。当前最新版本是`0.9.1`
>
> `ConcurrentLogHandler`的使用方法与其他`handler`类一致，如与`RotatingFileHandler`的使用方法一样。

**初始化函数及参数：**

```python
class ConcurrentRotatingFileHandler(BaseRotatingHandler):
    """
    Handler for logging to a set of files, which switches from one file to the
    next when the current file reaches a certain size. Multiple processes can
    write to the log file concurrently, but this may mean that the file will
    exceed the given size.
    """
    def __init__(self, filename, mode='a', maxBytes=0, backupCount=0,
                 encoding=None, debug=True, delay=0):
```

参数含义同`Python`内置`RotatingFileHandler`类相同，具体可参考上一篇博文。同样继承自`BaseRotatingHandler`类。

**简单的示例：**

```python
import logging
from cloghandler import ConcurrentRotatingFileHandler

logger = logging.getLogger()
rotateHandler = ConcurrentRotatingFileHandler('./logs/my_logfile.log', "a", 1024*1024, 5)
logger.addHandler(rotateHandler)
logger.setLevel(logging.DEBUG)

logger.info('This is a info message.')
```

为了适应没有`ConcurrentRotatingFileHandler`包的情况，增加回退使用`RotatingFileHandler`的代码：

```python
try:
    from cloghandler import ConcurrentRotatingFileHandler as RFHandler
except ImportError:
    from warning import warn
    warn('ConcurrentRotatingFileHandler package not installed, Using builtin log handler')
    from logging.handlers import RotatingFileHandler as RFHandler
```

运行后可以发现，会自动创建一个`.lock`文件，通过锁的方式来安全的写日志文件。

> 备注： 该库自2013年以后就没有再更新，若有问题，可使用`3.2`小节中的`concurrent-log-handler`软件包。

------

在非单独使用`python`脚本的时候，注意使用方式：

```python
# 不建议使用方式
from cloghandler import ConcurrentRotatingFileHandler

.......
'handlers':{
        "error_file": {
            "class": "ConcurrentRotatingFileHandler",
            "maxBytes": 100*1024*1024,
            "backupCount": 3,
# 建议写完整
import cloghandler
'handlers':{
        "error_file": {
            "class": "cloghandler.ConcurrentRotatingFileHandler",
            "maxBytes": 100*1024*1024,
            "backupCount": 3,
```

否则，会出现如下错误：

```python
Error: Unable to configure handler 'access_file': Cannot resolve 'ConcurrentRotatingFileHandler': No module named 'ConcurrentRotatingFileHandler'
```

### 3.2 concurrent-log-handler包

该模块同样也为python的标准日志记录软件提供了额外的日志处理程序。即回将日志事件写入日志文件，当文件达到一定大小时，该日志文件将轮流轮转，多个进程可以安全地写入同一日志文件，还可以将其进行压缩（开启）。`Windows`和`POSIX`系统均受支持。

它可以看做是旧版本`cloghandler`的直接替代品，主需要将`cloghandler`更改为`concurrent_log_handler`。

其特征及说明与`cloghandler`一致，具体可见`3.1`小节。

**安装**：

```
pip install concurrent-log-handler
```

若是从源码安装，则执行如下命令：

```python
python setup.py install
```

**使用示例**：

```python
import logging
from concurrent_log_handler import ConcurrentRotatingFileHandler

logger = logging.getLogger()
rotateHandler = ConcurrentRotatingFileHandler('./logs/mylogfile.log', 'a', 512*1024, 5)
logger.addHandler(rotateHandler)
logger.setLevel(logging.DEBUG)

logger.info('This is a info message.')
```

同样的，若要分发代码，不确定是否都已安装`concurrent_log_handler`软件包时，使`Python`可以轻松的回退到内置的`RotatingFileHandler`。下面是示例：

```python
import logging
try:
    from concurrent_log_handler import ConcurrentRotatingFileHandler as RFHandler
except ImportError:
    # 下面两行可选
    from warnings import warn
    warn('concurrent_log_handler package not installed. Using builtin log handler')
    from logging.handlers import RotatingFileHandler as RFHandler

logger = logging.getLogger()
rotateHandler = RFHandler('./logs/mylogfile.log', 'a', 1024*1024, 5)
logger.addHandler(rotateHandler)
```

> 同样的，建议直接导入`concurrent_log_handler`，使用`concurrent_log_handler.ConcurrentRotatingFileHandler`方式。

### 3.3 对日志输出加锁

`TimedRotatingFileHandler`类`doRollover`函数的主要部分如下：

```python
def doRollover(self):
    ....
    dfn = self.rotation_filename(self.baseFilename + "." +
                                     time.strftime(self.suffix, timeTuple))
    # -------begin-------
    if os.path.exists(dfn): # 判断如果存在dfn，则删除
            os.remove(dfn)
    self.rotate(self.baseFilename, dfn) # 将当前日志文件重命名为dfn
    # --------end--------
    if self.backupCount > 0:
        for s in self.getFilesToDelete():
            os.remove(s)
    if not self.delay:
        self.stream = self._open()
    ....
```

**修改思路：**

判断`dfn`文件是否已经存在，如果存在，表示已经被`rename`过了；如果不存在，则只允许一个进程去`rename`，其他进程需等待。

新建一个类继承自`TimeRotatingFileHandler`，修改`doRollover`函数，只需处理上面代码的注释部分即可。如下：

```python
class MPTimeRotatingFileHandler(TimeRotatingFileHandler):
    def doRollover(self):
        ....
        dfn = self.rotation_filename(self.baseFilename + "." +
                                     time.strftime(self.suffix, timeTuple))
        # ----modify start----
        if not os.path.exists(dfn):
            f = open(self.baseFilename, 'a')
            fcntl.lockf(f.fileno(), fcntl.LOCK_EX)
            if os.path.exists(self.baseFilename): # 判断baseFilename是否存在
                self.rotate(self.baseFilename, dfn)
        # ----modify end-----
        if self.backupCount > 0:
        for s in self.getFilesToDelete():
            os.remove(s)
        ....
```

### 3.4 重写FileHandler类

`logging.handlers.py`中各类的继承关系如下图所示：

![image-20200106151225654](Python日志模块-多进程日志记录.assets/16fa344b43219601)

`TimeRotatingFileHandler`类就是继承自该类，在`FileHandler`类中增加一些处理。

具体可参考以下博文：

1. [python logging日志模块以及多进程日志 | doudou0o blog](https://www.doudou0o.com/archives/fe118cd8.html)
2. [python多进程解决日志错乱问题_qq_20690231的博客-CSDN博客](https://blog.csdn.net/qq_20690231/article/details/84644939)

------

> 在`Python`官方手册中，提供了多进程中日志记录至单个文件的方法。

`logging`是线程安全的，将单个进程中的多个线程日志记录至单个文件也是支持的。但将多个进程中的日志记录至单个文件中则不支持，因为在`Python`中并没有在多个进程中实现对单个文件访问的序列化的标准方案。

将多个进程中日志记录至单个文件中，有以下几个方案：

1. 让所有进程都将日志记录至一个 `SocketHandler`，然后用一个实现了套接字服务器的单独进程一边从套接字中读取一边将日志记录至文件。
2. 使用 `Queue` 和 [QueueHandler](https://docs.python.org/zh-cn/3/library/logging.handlers.html#logging.handlers.QueueHandler) 将所有的日志事件发送至你的多进程应用的一个进程中。

### 3.5 单独进程负责日志事件

一个单独监听进程负责监听其他进程的日志事件，并根据自己的配置记录。

**示例：**

```python
import logging
import logging.handlers
import multiprocessing

from random import choice, random
import time

def listener_configurer():
    root = logging.getLogger()
    h = logging.handlers.RotatingFileHandler('test.log', 'a', 300,10) # rotate file设置的很小，以便于查看结果
    f = logging.Formatter('%(asctime)s %(processName)-10s %(name)s %(levelname)-8s %(message)s')
    h.setFormatter(f)
    root.addHandler(h)
   
def listenser_process(queue, configurer):
    configurer()
    while True:
        try:
            record = queue.get()
            if record is None:
                break
            logger = logging.getLogger(record.name)
            logger.handle(record)
        except Exception:
            import sys, traceback
            print('Whoops! Problem:', file=sys.stderr)
            trackback.print_exc(file=sys.stderr)

LEVELS = [logging.DEBUG, logging.INFO, logging.WARNING,
          logging.ERROR, logging.CRITICAL]

LOGGERS = ['a.b.c', 'd.e.f']

MESSAGES = [
    'Random message #1',
    'Random message #2',
    'Random message #3',
]

def worker_configurer(queue):
    h = logging.handlers.QueueHandler(queue)
    root = logging.getLogger()
    root.addHandler(h)
    root.setLevel(logging.DEBUG)
    
# 该循环仅记录10个事件，这些事件具有随机的介入延迟，然后终止
def worker_process(queue, configurer):
    configurer(queue)
    name = multiprocessing.current_process().name
    print('Worker started:%s'%name)
    for i in range(10):
        time.sleep(random())
        logger = logging.getLogger(choice(LOGGERS))
        level = choice(LEVELS)
        message = choice(MESSAGES)
        logger.log(level, message)
# 创建队列，创建并启动监听器，创建十个工作进程并启动它们，等待它们完成，然后将None发送到队列以通知监听器完成
def main():
    queue = multiprocessing.Queue(-1)
    listener = multiprocessing.Process(target=listener_process,
                                      args=(queue, listener_configurer))
    listener.start()
    workers = []
    for i in range(10):
        worker = multiprocessing.Process(target=worker_process,
                                        args=(queue, listener_configurer))
        workers.append(worker)
        worker.start()
    for w in workers:
        w.join()
    queue.put_nowait(None)
    listener.join()
    
if __name__ == '__main__':
    main()
```

**使用主进程中一个单独的线程记录日志**

下面这段代码展示了如何使用特定的日志记录配置，例如`foo`记录器使用了特殊的处理程序，将`foo`子系统中所有的事件记录至一个文件`mplog-foo.log`。在主进程（即使是在工作进程中产生的日志事件）的日志记录机制中将直接使用恰当的配置。

```python
import logging
import logging.config
import logging.handlers
from multiprocessing import Process, Queue
import random
import threading
import time

def logger_thread(q):
    while True:
        record = q.get()
        if record is None:
            break
        logger = logging.getLogger(record.name)
        logger.handle(record)
        
def worker_process(q):
    qh = logging.handlers.QueueHandler(q)
    root = logging.getLogger()
    root.setLevel(logging.DEBUG)
    root.addHandler(qh)
    levels = [logging.DEBUG, logging.INFO, logging.WARNING, logging.ERROR,
              logging.CRITICAL]
    loggers = ['foo', 'foo.bar', 'foo.bar.baz', 'spam', 'spam.ham', 'spam.ham.eggs']
    
    for i in range(100):
        lv1=l = random.choice(levles)
        logger = logging.getLogger(random.choice(loggers))
        logger.log(lvl, 'Message no. %d', i)

for __name__ == '__main__':
    q = Queue()
    d = {
        'version': 1,
        'formatters': {
            'detailed': {
                'class': 'logging.Formatter',
                'format': '%(asctime)s %(name)-15s %(levelname)-8s %(processName)-10s %(message)s'
            }
        },
        'handlers': {
            'console': {
                'class': 'logging.StreamHandler',
                'level': 'INFO',
            },
            'file': {
                'class': 'logging.FileHandler',
                'filename': 'mplog.log',
                'mode': 'w',
                'formatter': 'detailed',
            },
            'foofile': {
                'class': 'logging.FileHandler',
                'filename': 'mplog-foo.log',
                'mode': 'w',
                'formatter': 'detailed',
            },
            'errors': {
                'class': 'logging.FileHandler',
                'filename': 'mplog-errors.log',
                'mode': 'w',
                'level': 'ERROR',
                'formatter': 'detailed',
            },
        },
        'loggers': {
            'foo': {
                'handlers': ['foofile']
            }
        },
        'root': {
            'level': 'DEBUG',
            'handlers': ['console', 'file', 'errors']
        },
    }
    workers = []
    for i in range(5):
        wp = Process(target=worker_process, name='worker %d'%(i+1), args=(q,))
        workers.append(wp)
        wp.start()
    logging.config.dictConfig(d)
    lp = threading.Thread(target=logger_thread, args=(q,))
    lp.start()
    
    for wp in workers:
        wp.join()
    q.put(None)
    lp.join()
```

### 3.6 logging.SocketHandler的方案

该方案具体的使用方法，下方博文给出了说明，具体实现参考如下博客：

[Python中logging在多进程环境下打印日志 - VictoKu - 博客园](https://www.cnblogs.com/kuliuheng/p/11190485.html)

## 四、参考文献 

1. [用 Python 写一个多进程兼容的 TimedRotatingFileHandler - piperck - 博客园](https://www.cnblogs.com/piperck/p/9837637.html)
# 使用loguru记录日志

前面我们学习了Python的日志模块，发现想要使用不是开箱即用，而是需要进行一系列的配置。那么，有咩有直接开箱即用的日志库呢？答案当然是有的，今天我们学习下`loguru`模块。

## 安装

```python
pip install loguru
```

## 基本使用

那么这个库怎么来用呢？我们先用一个实例感受下：

```python
In [1]: from loguru import logger
   ...:
   ...: logger.debug('this is a debug message')
```

看到了吧，不需要配置什么东西，直接引入一个 `logger`，然后调用其 `debug`方法即可。

在 `loguru` 里面有且仅有一个主要对象，那就是 `logger`，`loguru` 里面有且仅有一个 `logger`，而且它已经被提前配置了一些基础信息，比如比较友好的格式化、文本颜色信息等等。

上面的代码运行结果如下：

```python
2019-11-06 22:45:31.653 | DEBUG    | __main__:<module>:3 - this is a debug message
```

可以看到其默认的输出格式是上面的内容，有时间、级别、模块名、行号以及日志信息，不需要手动创建  `logger`，直接使用即可，另外其输出还是彩色的，看起来会更加友好。

以上的日志信息是直接输出到控制台的，并没有输出到其他的地方，如果想要输出到其他的位置，比如存为文件，我们只需要使用一行代码声明即可。

例如将结果同时输出到一个 runtime.log 文件里面，可以这么写：

```python
In [2]: from loguru import logger
   ...:
   ...: logger.add('runtime.log')
   ...: logger.debug('this is a debug')
```

很简单吧，我们也不需要再声明一个 `FileHandler` 了，就一行 `add` 语句搞定，运行之后会发现目录下 `runtime.log` 里面同样出现了刚刚控制台输出的 `DEBUG` 信息。

```shell
cat runtime.log
2019-11-06 22:46:59.690 | DEBUG    | __main__:<module>:4 - this is a debug
```

上面就是一些基本的使用，但这还远远不够，下面我们来详细了解下它的一些功能模块。·

## 详细使用

既然是日志，那么最常见的就是输出到文件了。`loguru` 对输出到文件的配置有非常强大的支持，比如支持输出到多个文件，分级别分别输出，过大创建新文件，过久自动删除等等。

下面我们分别看看这些怎样来实现，这里基本上就是 `add` 方法的使用介绍。因为这个 `add` 方法就相当于给 `logger` 添加了一个 `Handler`，它给我们暴露了许多参数来实现 `Handler` 的配置，下面我们来详细介绍下。

首先看看它的方法定义吧：

```python
def add(
    self,
    sink,
    *,
    level=_defaults.LOGURU_LEVEL,
    format=_defaults.LOGURU_FORMAT,
    filter=_defaults.LOGURU_FILTER,
    colorize=_defaults.LOGURU_COLORIZE,
    serialize=_defaults.LOGURU_SERIALIZE,
    backtrace=_defaults.LOGURU_BACKTRACE,
    diagnose=_defaults.LOGURU_DIAGNOSE,
    enqueue=_defaults.LOGURU_ENQUEUE,
    catch=_defaults.LOGURU_CATCH,
    **kwargs
):
    r"""Add a handler sending log messages to a sink adequately configured.
```

看看它的源代码，它支持这么多的参数，如 `level`、`format`、`filter`、`color` 等等，另外我们还注意到它有个非常重要的参数 `sink`，我们看看官方文档：[sink](https://links.jianshu.com/go?to=https%3A%2F%2Floguru.readthedocs.io%2Fen%2Fstable%2Fapi%2Flogger.html%23sink)，可以了解到通过 `sink` 我们可以传入多种不同的数据结构，汇总如下：

-  `sink` 可以传入一个 `file` 对象，例如 `sys.stderr` 或者 `open('file.log', 'w')` 都可以。
-  `sink` 可以直接传入一个 `str` 字符串或者 `pathlib.Path` 对象，其实就是代表文件路径的，如果识别到是这种类型，它会自动创建对应路径的日志文件并将日志输出进去。
-  `sink` 可以是一个方法，可以自行定义输出实现。
-  `sink` 可以是一个 `logging` 模块的 `Handler`，比如 `FileHandler`、`StreamHandler` 等等，或者上文中我们提到的 `CMRESHandler` 照样也是可以的，这样就可以实现自定义 `Handler` 的配置。
-  `sink` 还可以是一个自定义的类，具体的实现规范可以参见官方文档。

所以说，刚才我们所演示的输出到文件，仅仅给它传了一个 `str` 字符串路径，他就给我们创建了一个日志文件，就是这个原理。

### **基本参数**

下面我们再了解下它的其他参数，例如 `format`、`filter`、`level` 等等。

其实它们的概念和格式和 `logging` 模块都是基本一样的了，例如这里使用`format`、`filter`、`level`来规定输出的格式：

```python
logger.add('runtime.log', format="{time} {level} {message}", filter="my_module", level="INFO")
```

### **删除 sink**

另外添加 `sink` 之后我们也可以对其进行删除，相当于重新刷新并写入新的内容。

删除的时候根据刚刚 `add` 方法返回的 `id` 进行删除即可，看下面的例子：

```python
from loguru import logger
 
trace = logger.add('runtime.log')
logger.debug('this is a debug message')
logger.remove(trace)
logger.debug('this is another debug message')
```

看这里，我们首先 `add` 了一个 `sink`，然后获取它的返回值，赋值为 `trace`。随后输出了一条日志，然后将 `trace` 变量传给`remove` 方法，再次输出一条日志，看看结果是怎样的。

控制台输出如下：

```python
2019-11-06 23:03:24.368 | DEBUG    | __main__:<module>:4 - this is a debug message
2019-11-06 23:03:24.369 | DEBUG    | __main__:<module>:6 - this is another debug message
```

日志文件 `runtime.log` 内容如下：

```powershell
cat runtime.log
2019-11-06 23:03:24.368 | DEBUG    | __main__:<module>:4 - this is a debug message
```

可以发现，在调用 `remove` 方法之后，确实将历史 `log` 删除了。

这样我们就可以实现日志的刷新重新写入操作。

### **rotation 配置**

用了 `loguru` 我们还可以非常方便地使用`rotation` 配置，比如我们想一天输出一个日志文件，或者文件太大了自动分隔日志文件，我们可以直接使用 `add` 方法的 `rotation` 参数进行配置。

我们看看下面的例子：

```python
logger.add('runtime_{time}.log', rotation="500 MB")
```

通过这样的配置我们就可以实现每 500MB 存储一个文件，每个 log 文件过大就会新创建一个 log 文件。我们在配置 log 名字时加上了一个 `time` 占位符，这样在生成时可以自动将时间替换进去，生成一个文件名包含时间的 log 文件。

另外我们也可以使用 `rotation` 参数实现定时创建 log 文件，例如：

```python
logger.add('runtime_{time}.log', rotation='00:00')
```

这样就可以实现每天 0 点新创建一个 log 文件输出了。

另外我们也可以配置 log 文件的循环时间，比如每隔一周创建一个 log 文件，写法如下：

```python
logger.add('runtime_{time}.log', rotation='1 week')
```

这样我们就可以实现一周创建一个 log 文件了。

```python
- an |int| which corresponds to the maximum file size in bytes before that the current
  logged file is closed and a new one started over.
- a |timedelta| which indicates the frequency of each new rotation.
- a |time| which specifies the hour when the daily rotation should occur.
- a |str| for human-friendly parametrization of one of the previously enumerated types.
  Examples: ``"100 MB"``, ``"0.5 GB"``, ``"1 month 2 weeks"``, ``"4 days"``, ``"10h"``,
  ``"monthly"``, ``"18:00"``, ``"sunday"``, ``"w0"``, ``"monday at 12:00"``, ...
- a |function|_ which will be called before logging. It should accept two
  arguments: the logged message and the file object, and it should return ``True`` if
  the rotation should happen now, ``False`` otherwise.
```

### **retention 配置**

很多情况下，一些非常久远的 log 对我们来说并没有什么用处了，它白白占据了一些存储空间，不清除掉就会非常浪费。`retention` 这个参数可以配置日志的最长保留时间。

比如我们想要设置日志文件最长保留 10 天，可以这么来配置：

```python
logger.add('runtime.log', retention='10 days')
```

这样 log 文件里面就会保留最新 10 天的 log，妈妈再也不用担心 log 沉积的问题啦。

我们看下源码看下这个参数可以设置为哪些值:

```python
- an |int| which indicates the number of log files to keep, while older files are removed.
- a |timedelta| which specifies the maximum age of files to keep.
- a |str| for human-friendly parametrization of the maximum age of files to keep.
  Examples: ``"1 week, 3 days"``, ``"2 months"``, ...
- a |function|_ which will be called before the retention process. It should accept the list
  of log files as argument and process to whatever it wants (moving files, removing them,
  etc.).
```

### **compression 配置**

`loguru` 还可以配置文件的压缩格式，比如使用 `zip` 文件格式保存，示例如下：

```python
logger.add('runtime.log', compression='zip')
```

这样可以更加节省存储空间。

我们看下源码看下这个参数可以设置为哪些值:

```python
- a |str| which corresponds to the compressed or archived file extension. This can be one
  of: ``"gz"``, ``"bz2"``, ``"xz"``, ``"lzma"``, ``"tar"``, ``"tar.gz"``, ``"tar.bz2"``,
  ``"tar.xz"``, ``"zip"``.
- a |function|_ which will be called before file termination. It should accept the path
  of the log file as argument and process to whatever it wants (custom compression,
  network sending, removing it, etc.).
```

### **enqueue配置**

`loguru`可以配置在多进程同时往日志文件写日志的时候使用队列达到异步功效。

```python
logger.add("somefile.log", enqueue=True)  # 异步写入
```

看下源码的解释:

```python
enqueue : |bool|, optional
    Whether the messages to be logged should first pass through a multiprocess-safe queue
    before reaching the sink. This is useful while logging to a file through multiple
    processes.
```

### **字符串格式化**

`loguru` 在输出 log 的时候还提供了非常友好的字符串格式化功能，像这样：

```python
logger.info('If you are using Python {}, prefer {feature} of course!', 3.6, feature='f-strings')
```

这样在添加参数就非常方便了。

### **Traceback 记录**

在很多情况下，如果遇到运行错误，而我们在打印输出 log 的时候万一不小心没有配置好 Traceback 的输出，很有可能我们就没法追踪错误所在了。

但用了 loguru 之后，我们用它提供的装饰器就可以直接进行 Traceback 的记录，类似这样的配置即可：

```python
@logger.catch
def my_function(x, y, z):
    # An error? It's caught anyway!
    return 1 / (x + y + z)
```

我们做个测试，我们在调用时三个参数都传入 0，直接引发除以 0 的错误，看看会出现什么情况：

```python
my_function(0, 0, 0)
```

运行完毕之后，可以发现 log 里面就出现了 Traceback 信息，而且给我们输出了当时的变量值，真的是不能再赞了！结果如下：

```python
> File "run.py", line 15, in <module>
    my_function(0, 0, 0)
    └ <function my_function at 0x1171dd510>
 
  File "/private/var/py/logurutest/demo5.py", line 13, in my_function
    return 1 / (x + y + z)
                │   │   └ 0
                │   └ 0
                └ 0
 
ZeroDivisionError: division by zero
```

因此，用 loguru 可以非常方便地实现日志追踪，debug 效率可能要高上十倍了？

> Want to intercept standard logging messages toward your Loguru sinks?

```python
class InterceptHandler(logging.Handler):
    def emit(self, record):
        # Retrieve context where the logging call occurred, this happens to be in the 6th frame upward
        logger_opt = logger.opt(depth=6, exception=record.exc_info)
        logger_opt.log(record.levelno, record.getMessage())

logging.basicConfig(handlers=[InterceptHandler()], level=0)
```

参考文章：

- [官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Floguru.readthedocs.io%2Fen%2Fstable%2Findex.html)
- [loguru模块](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.cdwanze.work%2Farticles%2Floguru-module.html)
- [Python 中更优雅的日志记录方案 loguru](https://links.jianshu.com/go?to=https%3A%2F%2Fcuiqingcai.com%2F7776.html)
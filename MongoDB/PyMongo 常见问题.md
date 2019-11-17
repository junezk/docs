# PyMongo 常见问题

这是一篇翻译文章,原链接在[这里](http://api.mongodb.com/python/current/faq.html#writes-and-ids)。翻译**可能不准确**，欢迎指出文章中存在的问题。

### PyMongo是线程安全的吗

PyMongo是线程安全的,并且为多线程应用提供了内置的连接池

### PyMongo是进程安全的吗

PyMongo不是进程安全的,如果你在fork()中使用MongoClient实例，必须小心。具体来说，MongoClient实例不能从父进程复制到子进程,父进程和每个子进程必须创建属于自己的MongoClient实例。由于本身的不兼容性，在子进程中使用从父进程复制的MonogoClient实例很有可能发生死锁。PyMongo会在有可能引起死锁的情况下发出警告。

MongoClient产生多个线程来运行后台任务，如监视连接服务器。这些线程共享受Lock实例(进程不安全)保护的状态，所以，MongoClient受到与其他使用锁(互斥)的多线程程序一样的限制，其中一个限制是在使用fork()后锁失效。在fork过程中，所有锁都会被复制到子进程中，并且与父进程保持相同的状态：如果父进程中是锁定的，子进程复制的锁也是锁定的。由fork()创建的子进程只有一个线程，所以在这个子线程中任何从父进程中任何子线程中取出的锁都不会被释放，当这个子线程尝试获取其中一个锁时，会发生死锁。

有关在fork()使用多线程上下文中的Python锁的问题，请参阅[bugs.python.org/issue6721](http://bugs.python.org/issue6721)。

### 连接池在PyMongo中是如何工作的

每个MongoClient实例在每个MongoDb服务器都有一个内置的连接池，这些连接池会立即打开socket，用来支持多线程应用所需的并发操作MongoDB数量。这些socket没有线程相关性。

每个连接池的大小被限制在`maxPoolSize`，默认值为100.如果存在`maxPoolSize`个到服务器的连接并且这些连接全部在使用中，那么到该服务器的下一个请求会一直等待，直到其中一个连接可用。

客户端实例在MongoDB集群中的每个服务器上额外打开一个socket来监视服务器的状态。

例如：一个连接到3个节点主从复制服务器的客户端将打开3个监视socket。它还可以根据需要打开更多个socket(最多`maxPoolSize`)来支持每个服务器上多线程应用的并发操作。在`maxPoolSize`为100的情况下，如果应用只使用主连接，则只有主连接池的连接数增加(最多103)。如果应用使用ReadPreference查询辅助数据库，则它的连接池的连接数也会增加，总连接数可以达到303.

可以通过使用`minPoolSize`(默认0)参数来设置每个服务器的最小并发连接数。连接池将初始化`minPoolZise`个socket。如果由于网络原因导致socket关闭，导致socket的连接数(使用中和空闲中)下降到最小值以下，则会打开新的socket，直到socket的数量达到最小值。

可以使用`maxIdleTime`参数来设置一个连接在连接池中保持空闲的最大毫秒数，之后它将被删除或者替换，默认值为None(没有限制)。

MongoClient的默认配置适用于大多数应用：

```
client = MongoClient(host, port)
```

为每个进程创建一次客户端，并将其重用于所有操作。为每个请求创建一个新的客户端是一个常见的错误，因为这样非常低效。

要在一个进程中支持极高数量的并发MongoDB操作，需增加`maxPoolSize`:

```
client = MongoClient(host, port, maxPoolSize=200)
```

或者使其没有限制：

```
client = MongoClient(host, port, maxPoolSize=None)
```

默认情况下，允许任意数量的线程等待socket可用，并且可以等待任意长的时间。可以设置`waitQueueMultiple`参数来限制等待线程的数量。例如：限制等待数量不大于500：

```
client = MongoClient(host, port, maxPoolSize=50, waitQueueMultiple=10)
```

当已经由500个线程正在等待socket时，第501个需要socket的线程将抛出ExceededMaxWaiters。使用`waitQueueMultiple`可以现在加载峰值期间应用中排队的数量，但是会引起额外的异常。

一旦连接池达到最大值，另外的线程可以无限等待socket可用，除非你设置了`waitQueueTimeoutMS`:

```
client = MongoClient(host, port, waitQueueTimeoutMS=100)
```

在这个例子中，一个线程如果等待socket的时间超过100ms，它将抛出ConnectionFailure错误。`waitQueueTimeoutMS`适用于在加载峰值期间限制操作的持续时间比完成每个操作更重要的情景。

当任何线程调用close()时， 所有闲置的socket都会被关闭，所有正在使用的socket将在它返回连接池时被关闭。

### PyMongo支持Python3 吗？

PyMongo支持CPython3.4+和PyPy3。详情请参阅[Python3 FAQ](http://api.mongodb.com/python/current/python3.html)。

### PyMongo是否支持Gevent,asyncio,Tornado或Twisted等异步框架？

PyMongo完全支持[Gevent](http://api.mongodb.com/python/current/examples/gevent.html)。

要将MongoDB与[asyncio](https://docs.python.org/3/library/asyncio.html)或[Tornado](http://www.tornadoweb.org/en/stable/)一起使用，请参阅[Motor](https://github.com/mongodb/motor)项目。

对于[Twisted](http://twistedmatrix.com/trac/),请参阅[TxMongo](https://github.com/twisted/txmongo)

### 为什么PyMongo将一个_id字段添加到我所有的文档中？

当使用`insert_one()`,`insert_many()`或者`bulk_write()`向MongoDB中插入一个文档时，如果文档没有`_id`字段，PyMongo将自动加上`_id`字段，其值为[ObjectId](http://api.mongodb.com/python/current/api/bson/objectid.html#bson.objectid.ObjectId)的一个实例。例如：

```
>>> my_doc = {'x': 1}
>>> collection.insert_one(my_doc)
<pymongo.results.InsertOneResult object at 0x7f3fc25bd640>
>>> my_doc
{'x': 1, '_id': ObjectId('560db337fba522189f171720')}
```

当调用`insert_many()`向单个文档插入一个引用列表时，经常会引起`BulkWriteError`错误。这是几个Python习惯引起的：

```
>>> doc = {}
>>> collection.insert_many(doc for _ in range(10))
Traceback (most recent call last):
...
pymongo.errors.BulkWriteError: batch op errors occurred
>>> doc
{'_id': ObjectId('560f171cfba52279f0b0da0c')}

>>> docs = [{}]
>>> collection.insert_many(docs * 10)
Traceback (most recent call last):
...
pymongo.errors.BulkWriteError: batch op errors occurred
>>> docs
[{'_id': ObjectId('560f1933fba52279f0b0da0e')}]
```

PyMongo以这种方式添加`_id`字段有以下几个原因：

- 所有的MongoDB文档都必须由一个`_id`字段。
- 如果PyMongo插入补个不带有`_id`字段的文档，MongoDB会自己添加，并且不会返回`_id`字段给PyMongo。
- 在添加`_id`字段之前复制要插入的文档对于大多数高写入的应用而言代价是极其昂贵的。

如果你不希望PyMongo向文档中添加`_id`字段，则只能插入已有`_id`字段的文档。

### 副本中的键顺序-为什么查询在shell是有序的，在PyMongo中无序？

BSON文档中的键值对可以是任何顺序(除了`_id`始终是第一个)。在读写数据是，mongo shell按键保持顺序。下面的例子中请注意在插入是'b'在'a'前面，查询时也一样：

```
> // mongo shell.
> db.collection.insert( { "_id" : 1, "subdocument" : { "b" : 1, "a" : 1 } } )
WriteResult({ "nInserted" : 1 })
> db.collection.find()
{ "_id" : 1, "subdocument" : { "b" : 1, "a" : 1 } }
```

PyMongo在默认情况下将BSON文档表示为Python字典，并且没有字典中键的顺序。也就是说，声明Python字典时，'a'在前面或者'b'在前面是一样的。

```
>>> print({'a': 1.0, 'b': 1.0})
{'a': 1.0, 'b': 1.0}
>>> print({'b': 1.0, 'a': 1.0})
{'a': 1.0, 'b': 1.0}
```

因此，Python的字典不能保证按照他们在BSON中的顺序显示键值对。下面的例子中，'a'显示在'b'前面:

```
>>> print(collection.find_one())
{u'_id': 1.0, u'subdocument': {u'a': 1.0, u'b': 1.0}}
```

使用SON类可以在读取BSON时保持顺序，它是一个记住了键顺序的字典。首先，获取集合的句柄，通过配置使用SON代替字典：

```
>>> from bson import CodecOptions, SON
>>> opts = CodecOptions(document_class=SON)
>>> opts
CodecOptions(document_class=<class 'bson.son.SON'>,
             tz_aware=False,
             uuid_representation=PYTHON_LEGACY,
             unicode_decode_error_handler='strict',
             tzinfo=None)
>>> collection_son = collection.with_options(codec_options=opts)
```

现在，查询结果中的文档和副本都用SON对象表示:

```
>>> print(collection_son.find_one())
SON([(u'_id', 1.0), (u'subdocument', SON([(u'b', 1.0), (u'a', 1.0)]))])
```

副本中键顺序与实际存储的一致：'b'在'a'前面。

由于字典中的键顺序没有定义，所以你无法预测它如何序列化到BSON。但MongoDB认为副本只有在他们的键具有相同的顺序时才是相同的。所以，使用字典查询副本可能没有结果:

```
>>> collection.find_one({'subdocument': {'a': 1.0, 'b': 1.0}}) is None
True
```

在查询中交换键顺序没有任何区别：

```
>>> collection.find_one({'subdocument': {'b': 1.0, 'a': 1.0}}) is None
True
```

正如我们上面看到的，Python认为这两个字典是相同的。

由两个解决方法。第一个方法是按字段匹配副本：

```
>>> collection.find_one({'subdocument.a': 1.0,
...                      'subdocument.b': 1.0})
{u'_id': 1.0, u'subdocument': {u'a': 1.0, u'b': 1.0}}
```

上面的查询匹配任何'a'为1.0和'b'为1.0的副本，无论你在Python中指定它们的顺序如何或它们存储在BSON中的顺序如何。 此外，此查询现在可以将副本中与'a'和'b'之外的其他键相匹配，而之前的查询需要完全匹配。

第二个方法是使用SON来指定键的顺序:

```
>>> query = {'subdocument': SON([('b', 1.0), ('a', 1.0)])}
>>> collection.find_one(query)
{u'_id': 1.0, u'subdocument': {u'a': 1.0, u'b': 1.0}}
```

查询时，在创建SON时使用的键顺序在被序列化为BSON时会被保留。因此，您可以创建一个完全匹配集合中的副本的副本。

更多信息，请参阅[ MongoDB Manual entry on subdocument matching](https://docs.mongodb.com/manual/tutorial/query-documents/)

### CursorNotFound 游标id无效在服务端是什么意思？

如果MongoDB中的游标已经打开了很长时间而没有对它们执行任何操作，他们会在服务器上超时。这可能会导致在迭代游标时引发CursorNotFound异常。

### 如何更改游标的超时时间？

MongoDB不支持游标自定义超时时间，但可以完全关闭。在`find()`时传入`no_cursor_timeout=True`。

### 如何存储`decimal.Decimal`实例？

PyMongo >= 3.4 支持引入Decimal128 BSON类型。详情请参阅[Decimal12](http://api.mongodb.com/python/current/api/bson/decimal128.html#module-bson.decimal128)。

MongoDB <= 3.2 仅支持IEEE 754 浮点数-与Python浮点类型相同。PyMongo可以在这些版本的MongoDB中存储Decimal实例的唯一方法是将他们转换成这个标准，所以不管如何你只可以存储浮点数。我们强迫用户明确的做这个转换来告知它们转换正在发生。

### 我保存了9.99，但是查询是确变成了9.9900000000000002，这是怎么回事？

数据库将9.99表示为IEEE浮点数(这是MongoDB和Python以及大多数其他现代语言通用的)。问题是9.99不能用双精度浮点数来表示，在Python的某些版本中也是如此：

```
>>> 9.99
9.9900000000000002
```

使用PyMongo保存9.99时得到的结果与使用JavaScript shell或任何其他语言保存的结果完全相同（以及将9.99输入到 Python程序的一样）。

### 你们能添加对文档属性方式的取值吗？

> 通过`.`获取文档的值，而不仅仅是现在只能用Python字典的方式获取

这个请求已经出现了很多次，但我们决定不实现任何这样的方式。相关的[jria case](https://jira.mongodb.org/browse/PYTHON-35?_ga=2.237174980.570925531.1513670372-1759230200.1510018761&_gac=1.209004070.1510018761.EAIaIQobChMIqufM-qmr1wIVDKlpCh27iQHIEAAYASAAEgLZbPD_BwE)有关于这个决定的一些信息，这里是一个简短的总结：

1. 这将污染文档的属性名称空间，因此当使用与字典方法相同的名称的键时可能导致细微的bugs/混淆错误。
2. 我们使用SON对象而不是常规字典的唯一原因是维护键排序，因为服务器需要这些准确的操作。因此我们在是否需要复杂的SON的必要性上犹豫不决(某种程度上我们希望恢复单独使用字典，而不会破坏每个人的向后兼容性)
3. 因为文档的表现像字典，所以新用户很容易(和Pythonic)的处理文档。如果我们开始改变这点将会为新受增加一个障碍-另外的学习成本。

### PyMongo中处理时区的正确方法是什么？

有关如何正确处理datetime对象的示例，请参阅[Datetime and Timezones](http://api.mongodb.com/python/current/examples/datetimes.html)。

### 如何保存一个datetime.date实例？

PyMongo不支持保存datetime.date实例，因为BSON没有类型来保存(没有时间的日期(yyyy-MM-dd))。PyMongo并没有强制将`datetime.date`转换为`datetime.datetime`的约定，所以你需要在代码中执行转换。

### 在web程序中使用ObjectId查询文档时，没有得到任何结果？

在Web应用程序中，通常在URL中会对文档的ObjectId进行编码，如：

```
"/posts/50b3bda58a02fb9a84d8991e"
```

web框架将ObjectId作为url字符串的一部分传递给后台，因此在她们呢传递给`find_one()`之前，必须转换为ObjectId。忘记这个转换是一个常见的错误。下面的例子是在Flask中正确的执行操作(其他Web框架类似):

```
rom pymongo import MongoClient
from bson.objectid import ObjectId

from flask import Flask, render_template

client = MongoClient()
app = Flask(__name__)

@app.route("/posts/<_id>")
def show_post(_id):
   # NOTE!: converting _id from string to ObjectId before passing to find_one
   post = client.db.posts.find_one({'_id': ObjectId(_id)})
   return render_template('post.html', post=post)

if __name__ == "__main__":
    app.run()
```

更多内容，请参阅[Querying By ObjectId](http://api.mongodb.com/python/current/tutorial.html#querying-by-objectid)

### 如何在Django中使用PyMongo？

Django是一个流行的Python Web框架。 Django包含一个ORM，django.db。 目前，Django没有官方的MongoDB库。

[django-mongodb-engine](https://django-mongodb-engine.readthedocs.io/en/latest/)是一个非官方的MongoDB库，支持Django聚合，（原子）更新，嵌入对象，Map / Reduce和GridFS。它允许您使用Django的大部分内置功能，包括ORM，admin，authentication，site 和会话框架以及缓存。

但是，在Django中不使用Django库也很容易使用MongoDB和PyMongo。除了某些需要django.db（管理，认证和会话）的Django的功能不能使用MongoDB外，Django提供的大部分功能仍然可以使用。

有一个让Django和MongoDB容易使用的项目是[mango](https://github.com/vpulim/mango)，mango是为Django会话和认证开发的一系列MongoDB库（完全绕过django.db）。

### PyMongo是否可以使用mod_wsgi？

可以。详情请参阅[PyMongo and mod_wsgi](http://api.mongodb.com/python/current/examples/mod_wsgi.html#pymongo-and-mod-wsgi)

### 如何使用Python的json模块来将我的文档编码为JSON？

[json_util](http://api.mongodb.com/python/current/api/bson/json_util.html#module-bson.json_util)是PyMongo内置的工具库，可以灵活与Python的json模块与BSON文档和 MongoDB Extended JSON一起使用。由于`json`模块不支持一些PyMongo的特殊类型（比如ObjectId和DBRef），所以不能支持所有documents。

[python-bsonjs](https://pypi.python.org/pypi/python-bsonjs)是建立在libbson之上的将BSON快速转换为 MongoDB Extended JSON的转换器。python-bsonjs不依赖于PyMongo，可以提供比json_util更好的性能。python-bsonjs在使用RawBSONDocument时与PyMongo最适合。

### 在解码另一种语言存储的日期时为什么会出现OverflowError？

PyMongo将BSON日期时间值解码为Python的datetime.datetime实例。datetime.datetime的实例被限制在datetime.MINYEAR（通常为1）和datetime.MAXYEAR（通常为9999）之间。某些MongoDB驱动程序（例如PHP驱动程序）可以存储远远超出datetime.datetime支持的年份值的BSON日期时间。

有几种方法可以解决此问题。 一种选择是过滤掉datetime.datetime支持的范围以外的值的文档：

```
>>> from datetime import datetime
>>> coll = client.test.dates
>>> cur = coll.find({'dt': {'$gte': datetime.min, '$lte': datetime.max}})
```

另一个方法是在你不需要日期时间字段时过滤掉这个字段：

```
>>> cur = coll.find({}, projection={'dt': False})
```

### 在多进程中使用PyMongo

在Unix系统上，多进程模块使用`fork()`生成进程。在`fork()`中使用MongoClient实例时必须小心：MongoClient的实例不能从父进程复制到子进程，父进程和每个子进程必须创建他们自己的MongoClient实例。例如：

```
# Each process creates its own instance of MongoClient.
def func():
    db = pymongo.MongoClient().mydb
    # Do something with db.

proc = multiprocessing.Process(target=func)
proc.start()
```

永远不要这样做：

```
client = pymongo.MongoClient()

# Each child process attempts to copy a global MongoClient
# created in the parent process. Never do this.
def func():
  db = client.mydb
  # Do something with db.

proc = multiprocessing.Process(target=func)
proc.start()
```

由于`fork()`、线程和锁之间固有的不兼容性，从父进程复制的MongoClient实例在子进程中死锁的可能性很高。
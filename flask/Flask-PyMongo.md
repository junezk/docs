# Flask-PyMongo

[MongoDB](http://www.mongodb.org/) 是一个开源的数据库，它存储着灵活的类-JSON 的“文档”。与关系数据库中的数据行相反，它能够存储任何的数字，名称，或者复杂的层级结构。Python 开发者可以考虑把 MongoDB 作为一个持久化，可搜索的 Python 字典的“仓库”(实际上，这是如何用 [PyMongo](http://api.mongodb.org/python/current/) 来表示 MongoDB 中的“文档”)。

Flask-PyMongo 架起来 Flask 和 PyMongo 之间的桥梁，因此你能够使用 Flask 正常的机制去配置和连接 MongoDB。

## 快速入门

首先，安装 Flask-PyMongo:

```
$ pip install Flask-PyMongo
```

Flask-PyMongo 的各种依赖，比如，最新版本的 Flask（0.8或者以上）以及 PyMongo（2.4或者以上） ，也会为你安装的。Flask-PyMongo 是兼容 Python 2.6, 2.7, 和 3.3 版本并且通过测试。

接着，在你的代码中添加一个 [`PyMongo`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.PyMongo):

```
from flask import Flask
from flask.ext.pymongo import PyMongo

app = Flask(__name__)
mongo = PyMongo(app)
```

[`PyMongo`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.PyMongo) 连接运行在本机上且端口为 27017 的 MongoDB 服务器，并且假设默认的数据库名为 `app.name` (换而言之，你可以使用传入到 `Flask` 中的任何数据库名)。这个数据库能够作为 [`db`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.PyMongo.db) 属性被导入。

你可以在视图中直接使用 [`db`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.PyMongo.db) :

```
@app.route('/')
def home_page():
    online_users = mongo.db.users.find({'online': True})
    return render_template('index.html',
        online_users=online_users)
```

## Helpers

Flask-PyMongo 提供一些通用任务的现成方法:

- `Collection.``find_one_or_404`(**args*, ***kwargs*)

  Find and return a single document, or raise a 404 Not Found exception if no document matches the query spec. See [`find_one()`](http://api.mongodb.org/python/current/api/pymongo/collection.html#pymongo.collection.Collection.find_one) for details.`@app.route('/user/<username>') def user_profile(username):     user = mongo.db.users.find_one_or_404({'_id': username})     return render_template('user.html',         user=user) `

- `PyMongo.``send_file`(*filename*, *base='fs'*, *version=-1*, *cache_for=31536000*)

  Return an instance of the `response_class` containing the named file, and implement conditional GET semantics (using `make_conditional()`).`@app.route('/uploads/<path:filename>') def get_upload(filename):     return mongo.send_file(filename) `Parameters:**filename** ([*str*](http://docs.python.org/library/functions.html#str)) – the filename of the file to return**base** ([*str*](http://docs.python.org/library/functions.html#str)) – the base name of the GridFS collections to use**version** ([*bool*](http://docs.python.org/library/functions.html#bool)) – if positive, return the Nth revision of the file identified by filename; if negative, return the Nth most recent revision. If no such version exists, return with HTTP status 404.**cache_for** ([*int*](http://docs.python.org/library/functions.html#int)) – number of seconds that browsers should be instructed to cache responses

- `PyMongo.``save_file`(*filename*, *fileobj*, *base='fs'*, *content_type=None*)

  Save the file-like object to GridFS using the given filename. Returns `None`.`@app.route('/uploads/<path:filename>', methods=['POST']) def save_upload(filename):     mongo.save_file(filename, request.files['file'])     return redirect(url_for('get_upload', filename=filename)) `Parameters:**filename** ([*str*](http://docs.python.org/library/functions.html#str)) – the filename of the file to return**fileobj** ([*file*](http://docs.python.org/library/functions.html#file)) – the file-like object to save**base** ([*str*](http://docs.python.org/library/functions.html#str)) – base the base name of the GridFS collections to use**content_type** ([*str*](http://docs.python.org/library/functions.html#str)) – the MIME content-type of the file. If`None`, the content-type is guessed from the filename using[`guess_type()`](http://docs.python.org/library/mimetypes.html#mimetypes.guess_type)

- *class* `flask_pymongo.``BSONObjectIdConverter`(*map*)

  A simple converter for the RESTful URL routing system of Flask.`@app.route('/<ObjectId:task_id>') def show_task(task_id):     task = mongo.db.tasks.find_one_or_404(task_id)     return render_template('task.html', task=task) `Valid object ID strings are converted into [`ObjectId`](http://api.mongodb.org/python/current/api/bson/objectid.html#bson.objectid.ObjectId) objects; invalid strings result in a 404 error. The converter is automatically registered by the initialization of [`PyMongo`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.PyMongo) with keyword `ObjectId`.

## Configuration

[`PyMongo`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.PyMongo) 直接支持如下的配置项:

| `MONGO_URI`                | 一个 [MongoDB 网址](http://www.mongodb.org/display/DOCS/Connections#Connections-StandardConnectionStringFormat) 用于其他配置项。 |
| -------------------------- | ------------------------------------------------------------ |
| `MONGO_HOST`               | 你的 MongoDB 服务器的主机名或者 IP 地址。 默认：”localhost”。 |
| `MONGO_PORT`               | 你的 MongoDB 服务器的端口。默认：27017。                     |
| `MONGO_AUTO_START_REQUEST` | 为了禁用 PyMongo 2.2 的 “auto start request” 行为， 设置成 `False`。 (请见 [`MongoClient`](http://api.mongodb.org/python/current/api/pymongo/mongo_client.html#pymongo.mongo_client.MongoClient))。 默认：`True`。 |
| `MONGO_MAX_POOL_SIZE`      | (可选): PyMongo 连接池中保持空闲连接的最大数量。 默认：PyMongo 默认值。 |
| `MONGO_SOCKET_TIMEOUT_MS`  | (可选): (整型) 在超时前套接字允许一个发送或者接收的耗时(毫秒)。 默认: PyMongo 默认值。 |
| `MONGO_CONNECT_TIMEOUT_MS` | (可选): (整型) 在超时前允许一个连接的耗时(毫秒)。 默认: PyMongo 默认值。 |
| `MONGO_DBNAME`             | 可用于作为 `db` 属性的数据库名。默认： `app.name`。          |
| `MONGO_USERNAME`           | 用于认证的用户名。默认：`None`。                             |
| `MONGO_PASSWORD`           | 用于认证的密码。默认：`None`。                               |
| `MONGO_REPLICA_SET`        | 设置成连接的备份集的名称； 这必须匹配到备份集的内部名，由 <http://www.mongodb.org/display/DOCS/Replica+Set+Commands#ReplicaSetCommands-isMaster> 命令)决定的。默认：`None`。 |
| `MONGO_READ_PREFERENCE`    | 决定如何读取路由到备份集的成员。 必须是定义在 [`pymongo.read_preferences.ReadPreference`](http://api.mongodb.org/python/current/api/pymongo/read_preferences.html#pymongo.read_preferences.ReadPreference) 中的一个常量 或者一个字符串名称。 |
| `MONGO_DOCUMENT_CLASS`     | 告诉 pymongo 返回定制的对象而不是默认的字典， 比如 `bson.son.SON`。 默认： `dict`。 |

当 [`PyMongo`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.PyMongo) 或者 [`init_app()`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.PyMongo.init_app) 仅仅只有一个参数调用的时候 (the Flask 实例)，会假设配置值的前缀是 `MONGO`；能够用 config_prefix 来覆盖这个前缀。

这个技术能够用于连接多个数据库或者数据服务器:

```python
app = Flask(__name__)

# connect to MongoDB with the defaults
mongo1 = PyMongo(app)

# connect to another MongoDB database on the same host
app.config['MONGO2_DBNAME'] = 'dbname_two'
mongo2 = PyMongo(app, config_prefix='MONGO2')

# connect to another MongoDB server altogether
app.config['MONGO3_HOST'] = 'another.host.example.com'
app.config['MONGO3_PORT'] = 27017
app.config['MONGO3_DBNAME'] = 'dbname_three'
mongo3 = PyMongo(app, config_prefix='MONGO3')
```

你应该需要注意一些自动配置的设置:

- `tz_aware`:

  Flask-PyMongo 一直使用通用时区的 [`datetime`](http://docs.python.org/library/datetime.html#datetime.datetime) 对象。这是因为当建立连接的时候它设置 `tz_aware` 参数为 `True`。从 MongoDB 返回的 [`datetime`](http://docs.python.org/library/datetime.html#datetime.datetime) 对象一直是 UTC。

- `safe`:

  Flask-PyMongo 默认地设置成 “safe” 模式，这会导致 [`save()`](http://api.mongodb.org/python/current/api/pymongo/collection.html#pymongo.collection.Collection.save), [`insert()`](http://api.mongodb.org/python/current/api/pymongo/collection.html#pymongo.collection.Collection.insert),[`update()`](http://api.mongodb.org/python/current/api/pymongo/collection.html#pymongo.collection.Collection.update), 和 [`remove()`](http://api.mongodb.org/python/current/api/pymongo/collection.html#pymongo.collection.Collection.remove) 在返回前一直等待着服务器的应答。你可以在调用的时候通过传入 `safe=False` 参数到任何一个受影响的方法中来覆盖它。

# API

## 常量

- `flask_pymongo.``ASCENDING` *= 1*

  Ascending sort order.

- `flask_pymongo.``DESCENDING` *= -1*

  Descending sort order.

## 类

- *class* `flask_pymongo.``PyMongo`(*app=None*, *config_prefix='MONGO'*)

  Automatically connects to MongoDB using parameters defined in Flask configuration.`cx`The automatically created `Connection` or `ReplicaSetConnection` object.`db`The automatically created [`Database`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.wrappers.Database) object corresponding to the provided `MONGO_DBNAME` configuration parameter.`init_app`(*app*, *config_prefix='MONGO'*)Initialize the app for use with this `PyMongo`. This is called automatically if appis passed to `__init__()`.The app is configured according to the configuration variables `PREFIX_HOST`, `PREFIX_PORT`, `PREFIX_DBNAME`, `PREFIX_AUTO_START_REQUEST`,`PREFIX_REPLICA_SET`, `PREFIX_READ_PREFERENCE`, `PREFIX_USERNAME`, `PREFIX_PASSWORD`, and `PREFIX_URI` where “PREFIX” defaults to “MONGO”. If `PREFIX_URL` is set, it is assumed to have all appropriate configurations, and the other keys are overwritten using their values as present in the URI.Parameters:**app** (*flask.Flask*) – the application to configure for use with this `PyMongo`**config_prefix** ([*str*](http://docs.python.org/library/functions.html#str)) – determines the set of configuration variables used to configure this `PyMongo``save_file`(*filename*, *fileobj*, *base='fs'*, *content_type=None*)Save the file-like object to GridFS using the given filename. Returns `None`.`@app.route('/uploads/<path:filename>', methods=['POST']) def save_upload(filename):     mongo.save_file(filename, request.files['file'])     return redirect(url_for('get_upload', filename=filename)) `Parameters:**filename** ([*str*](http://docs.python.org/library/functions.html#str)) – the filename of the file to return**fileobj** ([*file*](http://docs.python.org/library/functions.html#file)) – the file-like object to save**base** ([*str*](http://docs.python.org/library/functions.html#str)) – base the base name of the GridFS collections to use**content_type** ([*str*](http://docs.python.org/library/functions.html#str)) – the MIME content-type of the file. If `None`, the content-type is guessed from the filename using [`guess_type()`](http://docs.python.org/library/mimetypes.html#mimetypes.guess_type)`send_file`(*filename*, *base='fs'*, *version=-1*, *cache_for=31536000*)Return an instance of the `response_class` containing the named file, and implement conditional GET semantics (using `make_conditional()`).`@app.route('/uploads/<path:filename>') def get_upload(filename):     return mongo.send_file(filename) `Parameters:**filename** ([*str*](http://docs.python.org/library/functions.html#str)) – the filename of the file to return**base** ([*str*](http://docs.python.org/library/functions.html#str)) – the base name of the GridFS collections to use**version** ([*bool*](http://docs.python.org/library/functions.html#bool)) – if positive, return the Nth revision of the file identified by filename; if negative, return the Nth most recent revision. If no such version exists, return with HTTP status 404.**cache_for** ([*int*](http://docs.python.org/library/functions.html#int)) – number of seconds that browsers should be instructed to cache responses

- *class* `flask_pymongo.wrappers.``Collection`(*database*, *name*, *create=False*, ***kwargs*)

  Custom sub-class of [`pymongo.collection.Collection`](http://api.mongodb.org/python/current/api/pymongo/collection.html#pymongo.collection.Collection) which adds Flask-specific helper methods.`find_one_or_404`(**args*, ***kwargs*)Find and return a single document, or raise a 404 Not Found exception if no document matches the query spec. See [`find_one()`](http://api.mongodb.org/python/current/api/pymongo/collection.html#pymongo.collection.Collection.find_one) for details.`@app.route('/user/<username>') def user_profile(username):     user = mongo.db.users.find_one_or_404({'_id': username})     return render_template('user.html',         user=user) `

## 封装

These classes exist solely in order to make expressions such as `mongo.db.foo.bar`evaluate to a [`Collection`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.wrappers.Collection) instance instead of a [`pymongo.collection.Collection`](http://api.mongodb.org/python/current/api/pymongo/collection.html#pymongo.collection.Collection)instance. They are documented here solely for completeness.

- *class* `flask_pymongo.wrappers.``MongoClient`(*host=None*, *port=None*, *max_pool_size=100*, *document_class=<type 'dict'>*, *tz_aware=False*, *_connect=True*, ***kwargs*)

  Returns instances of [`flask_pymongo.wrappers.Database`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.wrappers.Database) instead of [`pymongo.database.Database`](http://api.mongodb.org/python/current/api/pymongo/database.html#pymongo.database.Database) when accessed with dot notation.

- *class* `flask_pymongo.wrappers.``MongoReplicaSetClient`(*hosts_or_uri=None*, *max_pool_size=100*, *document_class=<type 'dict'>*, *tz_aware=False*, *_connect=True*, ***kwargs*)

  Returns instances of [`flask_pymongo.wrappers.Database`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.wrappers.Database) instead of [`pymongo.database.Database`](http://api.mongodb.org/python/current/api/pymongo/database.html#pymongo.database.Database) when accessed with dot notation.

- *class* `flask_pymongo.wrappers.``Database`(*connection*, *name*)

  Returns instances of [`flask_pymongo.wrappers.Collection`](http://www.pythondoc.com/flask-pymongo/index.html#flask_pymongo.wrappers.Collection) instead of [`pymongo.collection.Collection`](http://api.mongodb.org/python/current/api/pymongo/collection.html#pymongo.collection.Collection) when accessed with dot notation.
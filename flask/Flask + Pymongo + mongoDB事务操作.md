# Flask + Pymongo + mongoDB事务操作

## 本文概要

**主要对flask使用mongoDB 4.0事务操作进行记录**
**本文涉及flask 与 flask_pymongo 的组合使用，以及mongoDB 的事务操作**

### 本文目录

1. [flask_pymongo 的使用](https://blog.csdn.net/weixin_42290927/article/details/89672458#p1)
2. [mongoDB 4.0 事务操作](https://blog.csdn.net/weixin_42290927/article/details/89672458#p2)
3. [代码具体实现](https://blog.csdn.net/weixin_42290927/article/details/89672458#p3)

## flask_pymongo的使用

### 环境准备

1. mongoDB 4.0
2. flask 1.0.2
3. python 3.6.7
4. pymongo 3.8
5. flask_pymongo 2.2

### 操作解析

1. flask_pymongo 是对pymongo的一个简单封装
2. 首先需要在设置中建立与mongoDB 的数据库的连接
3. 其次在flask 中初始化一个对象用于数据库操作

```python
from flask import Flask
from flask_pymongo import Pymongo
from config import Config

app = Flask(__name__)
# 第一种初始化 config, 在config 中设置mongoDB的连接
app.config.from_object(Config)
# 第二种初始化 config
Config.init_app(app)
# 初始化mongoDB对象
mongo = Pymongo(app)
1234567891011
```

### config 文件

```python
class Config:
	USE_RELOADER = True
	MONGO_URI = "mongodb://{username}:{password}@{server}:{port}/{database}".\
			format(username=USERNAME, password=PASSWORD, server=SERVER,\
			 port=PORT, database=DATABASE)
			 
	@classmethod
	def init_app(cls, app):
		for key in dir(cls):
			if key.isupper():
				app.config[key] = getattr(cls, key)
1234567891011
```

### 基础操作

```python
@app.route("/hello")
def hello():
	mongo.db.hello.insert({"hello":"world"})
	data = mongo.db.hello.find({},{"_id":0})
	不能直接将data返回，且需要去除 _id 字段
	return [data]
123456
```

## mongoDB 事务

1. mongoDB 4.0 以上支持事务操作，[查看官方文档](https://blog.csdn.net/weixin_42290927/article/details/“https://docs.mongodb.com/manual/core/transactions/”)
2. 这种事务是跨集合和跨数据库的，相比以前一条文档的事务要强大很多
3. 对于事务的性能影响，暂时无法得知

## 代码具体实现

1. 使用pymongo中自带的session完成事务的操作
2. 官网中只有mongo Shell 的操作，没有python 版本的操作
3. 在pymongo 中也难以找到该说明

```python
	# cx 是 mongoClient 对象
	session = mongo.cx.start_session()
	hello_collection = mongo.cx.get_database("helloworld").hello
	# helloworld 是库，world 是 集合
	world_collection = mongo.cx.get_database("helloworld").world
	# 事务开始
	session.start_transaction()
	try:
		hello_collection.insert({"world":1})
		world_collection.insert({"hello":1})
	except:
		# 操作异常，中断事务
		session.abort_transaction()
	else:
		session.commit_transaction()
	finally:
		session.end_session()
```
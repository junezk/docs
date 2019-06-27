# Python的MongoDB模块PyMongo操作方法集锦

> 这篇文章主要介绍了Python的MongoDB模块PyMongo操作方法集锦,包括数据的增删查改以及索引等相关的基本操作,需要的朋友可以参考下

开始之前当然要导入模块啦：


```python
>>> import pymongo
```

下一步，必须本地mongodb服务器的安装和启动已经完成，才能继续下去。

### 建立于MongoClient 的连接

```python
client = MongoClient('localhost', 27017)
# 或者
client = MongoClient('mongodb://localhost:27017/')
```

### 得到数据库

```python
db = client.test_database
# 或者
db = client['test-database']
```

### 得到一个数据集合

```python
collection = db.test_collection
# 或者
collection = db['test-collection']
```

### bjson文档

MongoDB中的数据使用的是类似Json风格的文档：

```python
>>> import datetime
>>> post = {"author": "Mike",
...     "text": "My first blog post!",
...     "tags": ["mongodb", "python", "pymongo"],
...     "date": datetime.datetime.utcnow()}
```

### 插入一个文档

```python
>>> posts = db.posts
>>> post_id = posts.insert_one(post).inserted_id
>>> post_id
ObjectId('...')
```

### 找一条数据

```python
>>> posts.find_one()
{u'date': datetime.datetime(...), u'text': u'My first blog post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'mongodb', u'python', u'pymongo']}
 
>>> posts.find_one({"author": "Mike"})
{u'date': datetime.datetime(...), u'text': u'My first blog post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'mongodb', u'python', u'pymongo']}
 
>>> posts.find_one({"author": "Eliot"})
>>>
```

### 通过ObjectId来查找

```python
>>> post_id
ObjectId(...)
>>> posts.find_one({"_id": post_id})
{u'date': datetime.datetime(...), u'text': u'My first blog post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'mongodb', u'python', u'pymongo']}
`>>> post_id``ObjectId(...)``>>> posts.find_one({``"_id"``: post_id})``{u``'date'``: datetime.datetime(...), u``'text'``: u``'My first blog post!'``, u``'_id'``: ObjectId(``'...'``), u``'author'``: u``'Mike'``, u``'tags'``: [u``'mongodb'``, u``'python'``, u``'pymongo'``]}`
```

### 不要转化ObjectId的类型为String

```python
>>> post_id_as_str = str(post_id)
>>> posts.find_one({"_id": post_id_as_str}) # No result
>>>
```

### 如果你有一个post_id字符串，怎么办呢？

```python
from bson.objectid import ObjectId
 
# The web framework gets post_id from the URL and passes it as a string
def get(post_id):
  # Convert from string to ObjectId:
  document = client.db.collection.find_one({'_id': ObjectId(post_id)})
```

### 多条插入

```python
>>> new_posts = [{"author": "Mike",
...        "text": "Another post!",
...        "tags": ["bulk", "insert"],
...        "date": datetime.datetime(2009, 11, 12, 11, 14)},
...       {"author": "Eliot",
...        "title": "MongoDB is fun",
...        "text": "and pretty easy too!",
...        "date": datetime.datetime(2009, 11, 10, 10, 45)}]
>>> result = posts.insert_many(new_posts)
>>> result.inserted_ids
[ObjectId('...'), ObjectId('...')]

```

### 查找多条数据

```python
>>> for post in posts.find():
...  post
...
{u'date': datetime.datetime(...), u'text': u'My first blog post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'mongodb', u'python', u'pymongo']}
{u'date': datetime.datetime(2009, 11, 12, 11, 14), u'text': u'Another post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'bulk', u'insert']}
{u'date': datetime.datetime(2009, 11, 10, 10, 45), u'text': u'and pretty easy too!', u'_id': ObjectId('...'), u'author': u'Eliot', u'title': u'MongoDB is fun'}
```

### 当然也可以约束查找条件

```python
>>> for post in posts.find({"author": "Mike"}):
...  post
...
{u'date': datetime.datetime(...), u'text': u'My first blog post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'mongodb', u'python', u'pymongo']}
{u'date': datetime.datetime(2009, 11, 12, 11, 14), u'text': u'Another post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'bulk', u'insert']}
```

### 获取集合的数据条数：

```python
posts.count()
```

### 或者说满足某种查找条件的数据条数：

```python
posts.find({"author": "Mike"}).count()
```

### 范围查找，比如说时间范围

```python
>>> d = datetime.datetime(2009, 11, 12, 12)
>>> for post in posts.find({"date": {"$lt": d}}).sort("author"):
...  print post
...
{u'date': datetime.datetime(2009, 11, 10, 10, 45), u'text': u'and pretty easy too!', u'_id': ObjectId('...'), u'author': u'Eliot', u'title': u'MongoDB is fun'}
{u'date': datetime.datetime(2009, 11, 12, 11, 14), u'text': u'Another post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'bulk', u'insert']}
```

$lt是小于的意思。

如何建立索引呢？比如说下面这个查找：

```python
>>> posts.find({"date": {"$lt": d}}).sort("author").explain()["cursor"]
u'BasicCursor'
>>> posts.find({"date": {"$lt": d}}).sort("author").explain()["nscanned"]
```

### 建立索引

```python
>>> from pymongo import ASCENDING, DESCENDING
>>> posts.create_index([("date", DESCENDING), ("author", ASCENDING)])
u'date_-1_author_1'
>>> posts.find({"date": {"$lt": d}}).sort("author").explain()["cursor"]
u'BtreeCursor date_-1_author_1'
>>> posts.find({"date": {"$lt": d}}).sort("author").explain()["nscanned"]
```

### 连接聚集

```python
>>> account = db.Account
#或 
>>> account = db["Account"]
```

### 查看全部聚集名称

```python
db.collection_names()
```

### 查看聚集的一条记录

```python
>>> db.Account.find_one()
>>> db.Account.find_one({"UserName":"keyword"})
```

### 查看聚集的字段

```python
>>> db.Account.find_one({},{"UserName":1,"Email":1})
{u'UserName': u'libing', u'_id': ObjectId('4ded95c3b7780a774a099b7c'), u'Email': u'libing@35.cn'}
  
 
>>> db.Account.find_one({},{"UserName":1,"Email":1,"_id":0})
{u'UserName': u'libing', u'Email': u'libing@35.cn'}
```

### 查看聚集的多条记录

```python
>>> for item in db.Account.find():
    item
   
>>> for item in db.Account.find({"UserName":"libing"}):
    item["UserName"]
```

### 查看聚集的记录统计

```python
>>> db.Account.find().count()

>>> db.Account.find({"UserName":"keyword"}).count()
```

### 聚集查询结果排序

```python
>>> db.Account.find().sort("UserName") #默认为升序
>>> db.Account.find().sort("UserName",pymongo.ASCENDING)  #升序
>>> db.Account.find().sort("UserName",pymongo.DESCENDING) #降序
```

### 聚集查询结果多列排序

```python
>>> db.Account.find().sort([("UserName",pymongo.ASCENDING),("Email",pymongo.DESCENDING)])
```

###  添加记录

```python
>>> db.Account.insert({"AccountID":21,"UserName":"libing"})
```

###  修改记录

```python
>>> db.Account.update({"UserName":"libing"},{"$set":{"Email":"libing@126.com","Password":"123"}})
```

###  删除记录

```python
>>> db.Account.remove()  -- 全部删除
 
>>> db.Test.remove({"UserName":"keyword"})
```
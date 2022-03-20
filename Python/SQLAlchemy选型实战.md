# SQLAlchemy选型实战

## peewee 与sqlalchemy选型对比

peewee 在github5000星
sqlalchemy在github2000星，但比较活跃，都commit10000+次。

这次选用sqlalchemy，主要是为了连接池，而opentack上是用sqlalchemy+pymysql这组合的案例，性级比较可靠。

这里就是用了sqlalchemy

# 参考1

## 介绍

SQLAlchemy是一个基于Python实现的ORM框架。该框架建立在 DB API之上，使用关系对象映射进行数据库操作，简言之便是：将类和对象转换成SQL，然后使用数据API执行SQL并获取执行结果。

```
pip3 install sqlalchemy
```

组成部分：

- Engine：框架的引擎
- Connection Pooling：数据库连接池
- Dialect：选择连接数据库的DB API种类
- Schema/Types：架构和类型
- SQL Expression Language：SQL表达语言

SQLAlchemy本身无法操作数据库，其必须以来pymsql等第三方插件，Dialect用于和数据API进行交流，根据配置文件的不同调用不同的数据库API，从而实现对数据库的操作，如：

```
MySQL-Python
    mysql+mysqldb://<user>:<password>@<host>[:<port>]/<dbname>

pymysql
    mysql+pymysql://<username>:<password>@<host>/<dbname>[?<options>]

MySQL-Connector
    mysql+mysqlconnector://<user>:<password>@<host>[:<port>]/<dbname>

cx_Oracle
    oracle+cx_oracle://user:pass@host:port/dbname[?key=value&key=value...]

更多：http://docs.sqlalchemy.org/en/latest/dialects/index.html
```

Sqlserver 可以使用DSN、Hostname、Pyodbc String等方式进行连接：

DSN：

```
mssql+pyodbc://<user>:<password>@<dsnname>
```

Hostname：

```
mssql+pyodbc://<user>:<password>@<host>[:<port>]/<dbname>
```

Pyodbc String：

```
driver=SQL+Server+Native+Client+13.0
```

## 使用

### 执行原生SQL

```python
from sqlalchemy import create_engine
# 首先创建引擎
engine = create_engine(
    "mysql+pymysql://root:123@127.0.0.1:3306/flask_demo?charset=utf8",
    max_overflow=0,  # 超过连接池大小外最多创建的连接
    pool_size=5,  # 连接池大小
    pool_timeout=30,  # 池中没有线程最多等待的时间，否则报错
    pool_recycle=-1  # 多久之后对线程池中的线程进行一次连接的回收（重置）
)
```

**方式一**

```python
conn = engine.raw_connection()
cursor = conn.cursor()
cursor.execute('select * from USER WHERE id > %s', (10,))
data = cursor.fetchall()
cursor.close()
conn.close()
```

**方式二**

```python
conn = engine.contextual_connect()
with conn:
    cursor = conn.execute('select * from USER WHERE id > %s', (10,))
    data = cursor.fetchall()
```

**方式三**

```python
cursor = engine.execute('select * from USER WHERE id > %s', (10,))
data = cursor.fetchall()
cursor.close()
```

注意: 查看连接 `show status like 'Threads%';`

### ORM

a.创建数据表

```python
# 创建单表
import datetime
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Text, ForeignKey, DateTime, UniqueConstraint, Index

Base = declarative_base()  # 先创建基类

class Users(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String(32), index=True, nullable=False)
    email = Column(String(32), unique=True)
    ctime = Column(DateTime, default=datetime.datetime.now)
    extra = Column(Text, nullable=True)

    __table_args__ = (
        UniqueConstraint('id', 'name', name='uix_id_name'),
        Index('ix_id_name', 'name', 'email'),
    )

engine = create_engine("mysql+pymysql://root@127.0.0.1:3306/flask_demo?charset=utf8")
Base.metadata.create_all(engine)  # 根据类创建数据库表
Base.metadata.drop_all(engine)  # 根据类删除数据库表
# 创建多个表并包含Fk、M2M关系
import datetime
from sqlalchemy import create_engine
from sqlalchemy import Column, Integer, String, Text, ForeignKey, DateTime, UniqueConstraint, Index
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()
engine = create_engine("mysql+pymysql://root@127.0.0.1:3306/flask_demo?charset=utf8")

# ##################### 单表示例 #########################
class Users(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String(32), index=True)
    age = Column(Integer, default=18)
    email = Column(String(32), unique=True)
    ctime = Column(DateTime, default=datetime.datetime.now)
    extra = Column(Text, nullable=True)

    __table_args__ = (
        UniqueConstraint('id', 'name', name='uix_id_name'),
        Index('ix_id_name', 'name', 'extra'),
    )

class Hosts(Base):
    __tablename__ = 'hosts'

    id = Column(Integer, primary_key=True)
    name = Column(String(32), index=True)
    ctime = Column(DateTime, default=datetime.datetime.now)

# ##################### 一对多示例 #########################
class Hobby(Base):
    __tablename__ = 'hobby'
    id = Column(Integer, primary_key=True)
    caption = Column(String(50), default='篮球')

class Person(Base):
    __tablename__ = 'person'
    nid = Column(Integer, primary_key=True)
    name = Column(String(32), index=True, nullable=True)
    hobby_id = Column(Integer, ForeignKey("hobby.id"))  # 外键写表名

    # 与生成表结构无关，仅用于查询方便
    hobby = relationship("Hobby", backref='pers')

# ##################### 多对多示例 #########################

class Server2Group(Base):
    __tablename__ = 'server2group'
    id = Column(Integer, primary_key=True, autoincrement=True)
    server_id = Column(Integer, ForeignKey('server.id'))
    group_id = Column(Integer, ForeignKey('group.id'))

class Group(Base):
    __tablename__ = 'group'
    id = Column(Integer, primary_key=True)
    name = Column(String(64), unique=True, nullable=False)

    # 与生成表结构无关，仅用于查询方便
    servers = relationship('Server', secondary='server2group', backref='groups')

class Server(Base):
    __tablename__ = 'server'

    id = Column(Integer, primary_key=True, autoincrement=True)
    hostname = Column(String(64), unique=True, nullable=False)

if __name__ == '__main__':
    Base.metadata.create_all(engine)
    # Base.metadata.drop_all(engine)
```

b.操作数据表

models.py

```python
import datetime
from sqlalchemy import Column, Integer, String, Text, ForeignKey, DateTime, UniqueConstraint, Index
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class Users(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String(32), index=True)
    age = Column(Integer, default=18)
    email = Column(String(32), unique=True)
    ctime = Column(DateTime, default=datetime.datetime.now)
    extra = Column(Text, nullable=True)

    __table_args__ = (
        UniqueConstraint('id', 'name', name='uix_id_name'),
        Index('ix_id_name', 'name', 'extra'),
    )

class Hosts(Base):
    __tablename__ = 'hosts'

    id = Column(Integer, primary_key=True)
    name = Column(String(32), index=True)
    ctime = Column(DateTime, default=datetime.datetime.now)

# ##################### 一对多示例 #########################
class Hobby(Base):
    __tablename__ = 'hobby'
    id = Column(Integer, primary_key=True)
    caption = Column(String(50), default='篮球')

class Person(Base):
    __tablename__ = 'person'
    nid = Column(Integer, primary_key=True)
    name = Column(String(32), index=True, nullable=True)
    hobby_id = Column(Integer, ForeignKey("hobby.id"))  # 外键写表名

    # 与生成表结构无关，仅用于查询方便
    hobby = relationship("Hobby", backref='pers')

# ##################### 多对多示例 #########################

class Server2Group(Base):
    __tablename__ = 'server2group'
    id = Column(Integer, primary_key=True, autoincrement=True)
    server_id = Column(Integer, ForeignKey('server.id'))
    group_id = Column(Integer, ForeignKey('group.id'))

class Group(Base):
    __tablename__ = 'group'
    id = Column(Integer, primary_key=True)
    name = Column(String(64), unique=True, nullable=False)

    # 与生成表结构无关，仅用于查询方便
    servers = relationship('Server', secondary='server2group', backref='groups')

class Server(Base):
    __tablename__ = 'server'

    id = Column(Integer, primary_key=True, autoincrement=True)
    hostname = Column(String(64), unique=True, nullable=False)
```

单表操作：

```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
from models import Users

engine = create_engine("mysql+pymysql://root@127.0.0.1:3306/flask_demo?charset=utf8")
Session = sessionmaker(bind=engine)

# 每次执行数据库操作时，都需要创建一个session
session = Session()

# ############# 执行ORM操作 #############
obj1 = Users(name="alex1")
session.add(obj1)

# 提交事务
session.commit()
# 关闭session
session.close()
```

多线程执行示例：

```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
from models import Users
import threading

engine = create_engine("mysql+pymysql://root@127.0.0.1:3306/flask_demo?charset=utf8")
Session = sessionmaker(bind=engine)

# 每次执行数据库操作时，都需要创建一个session
session = Session()

def task(arg):
    session = Session()
    obj = Users(name="mxt-"+threading.current_thread().name)
    session.add(obj)
    session.commit()
    session.close()

for i in range(10):
    t = threading.Thread(target=task, args=(i,))
    t.start()
```

基本增删改查示例：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import time
import threading

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey, UniqueConstraint, Index
from sqlalchemy.orm import sessionmaker, relationship
from sqlalchemy import create_engine
from sqlalchemy.sql import text

from models import Users, Hosts

engine = create_engine("mysql+pymysql://root@127.0.0.1:3306/flask_demo?charset=utf8")
Session = sessionmaker(bind=engine)

session = Session()

# ################ 添加 ################
"""
obj1 = Users(name="mxt")
session.add(obj1)

session.add_all([
    Users(name="wupeiqi"),
    Users(name="alex"),
    Hosts(name="c1.com"),
])
session.commit()
"""

# ################ 删除 ################
"""
session.query(Users).filter(Users.id > 2).delete()
session.commit()
"""
# ################ 修改 ################
"""
session.query(Users).filter(Users.id > 0).update({"name" : "099"})
session.query(Users).filter(Users.id > 0).update({Users.name: Users.name + "099"}, synchronize_session=False)
session.query(Users).filter(Users.id > 0).update({"age": Users.age + 1}, synchronize_session="evaluate")
session.commit()
"""

# ################ 查询 ################
"""
r1 = session.query(Users).all()
r2 = session.query(Users.name.label('xx'), Users.age).all()
r3 = session.query(Users).filter(Users.name == "alex").all()
r4 = session.query(Users).filter_by(name='alex').all()
r5 = session.query(Users).filter_by(name='alex').first()
r6 = session.query(Users).filter(text("id<:value and name=:name")).params(value=224, name='fred').order_by(Users.id).all()
r7 = session.query(Users).from_statement(text("SELECT * FROM users where name=:name")).params(name='ed').all()
"""

session.close()
```

常用操作：

```python
#　条件
ret = session.query(Users).filter_by(name='alex').all()
ret = session.query(Users).filter(Users.id > 1, Users.name == 'eric').all()
ret = session.query(Users).filter(Users.id.between(1, 3), Users.name == 'eric').all()
ret = session.query(Users).filter(Users.id.in_([1,3,4])).all()
ret = session.query(Users).filter(~Users.id.in_([1,3,4])).all()
ret = session.query(Users).filter(Users.id.in_(session.query(Users.id).filter_by(name='eric'))).all()
from sqlalchemy import and_, or_
ret = session.query(Users).filter(and_(Users.id > 3, Users.name == 'eric')).all()
ret = session.query(Users).filter(or_(Users.id < 2, Users.name == 'eric')).all()
ret = session.query(Users).filter(
    or_(
        Users.id < 2,
        and_(Users.name == 'eric', Users.id > 3),
        Users.extra != ""
    )).all()

# 通配符
ret = session.query(Users).filter(Users.name.like('e%')).all()
ret = session.query(Users).filter(~Users.name.like('e%')).all()

# 限制
ret = session.query(Users)[1:2]

# 排序
ret = session.query(Users).order_by(Users.name.desc()).all()
ret = session.query(Users).order_by(Users.name.desc(), Users.id.asc()).all()

# 分组
from sqlalchemy.sql import func

ret = session.query(Users).group_by(Users.extra).all()
ret = session.query(
    func.max(Users.id),
    func.sum(Users.id),
    func.min(Users.id)).group_by(Users.name).all()

ret = session.query(
    func.max(Users.id),
    func.sum(Users.id),
    func.min(Users.id)).group_by(Users.name).having(func.min(Users.id) >2).all()

# 连表

ret = session.query(Users, Favor).filter(Users.id == Favor.nid).all()

ret = session.query(Person).join(Favor).all()

ret = session.query(Person).join(Favor, isouter=True).all()

# 组合
q1 = session.query(Users.name).filter(Users.id > 2)
q2 = session.query(Favor.caption).filter(Favor.nid < 2)
ret = q1.union(q2).all()

q1 = session.query(Users.name).filter(Users.id > 2)
q2 = session.query(Favor.caption).filter(Favor.nid < 2)
ret = q1.union_all(q2).all()
```

原生sql语句：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import time
import threading

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey, UniqueConstraint, Index
from sqlalchemy.orm import sessionmaker, relationship
from sqlalchemy import create_engine
from sqlalchemy.sql import text
from sqlalchemy.engine.result import ResultProxy
from db import Users, Hosts

engine = create_engine("mysql+pymysql://root:123@127.0.0.1:3306/s6", max_overflow=0, pool_size=5)
Session = sessionmaker(bind=engine)

session = Session()

# 查询
# cursor = session.execute('select * from users')
# result = cursor.fetchall()

# 添加
cursor = session.execute('insert into users(name) values(:value)',params={"value":'wupeiqi'})
session.commit()
print(cursor.lastrowid)

session.close()
```

基于relationship操作ForeignKey

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import time
import threading

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey, UniqueConstraint, Index
from sqlalchemy.orm import sessionmaker, relationship
from sqlalchemy import create_engine
from sqlalchemy.sql import text
from sqlalchemy.engine.result import ResultProxy
from db import Users, Hosts, Hobby, Person

engine = create_engine("mysql+pymysql://root:123@127.0.0.1:3306/s6?charset=utf8", max_overflow=0, pool_size=5)
Session = sessionmaker(bind=engine)
session = Session()
# 添加
"""
session.add_all([
    Hobby(caption='乒乓球'),
    Hobby(caption='羽毛球'),
    Person(name='张三', hobby_id=3),
    Person(name='李四', hobby_id=4),
])

person = Person(name='张九', hobby=Hobby(caption='姑娘'))
session.add(person)

hb = Hobby(caption='人妖')
hb.pers = [Person(name='文飞'), Person(name='博雅')]
session.add(hb)

session.commit()
"""

# 使用relationship正向查询
"""
v = session.query(Person).first()
print(v.name)
print(v.hobby.caption)
"""

# 使用relationship反向查询
"""
v = session.query(Hobby).first()
print(v.caption)
print(v.pers)
"""

session.close()
```

基于relationship操作m2m

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import time
import threading

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey, UniqueConstraint, Index
from sqlalchemy.orm import sessionmaker, relationship
from sqlalchemy import create_engine
from sqlalchemy.sql import text
from sqlalchemy.engine.result import ResultProxy
from db import Users, Hosts, Hobby, Person, Group, Server, Server2Group

engine = create_engine("mysql+pymysql://root:123@127.0.0.1:3306/s6?charset=utf8", max_overflow=0, pool_size=5)
Session = sessionmaker(bind=engine)
session = Session()
# 添加
"""
session.add_all([
    Server(hostname='c1.com'),
    Server(hostname='c2.com'),
    Group(name='A组'),
    Group(name='B组'),
])
session.commit()

s2g = Server2Group(server_id=1, group_id=1)
session.add(s2g)
session.commit()

gp = Group(name='C组')
```

参考：
[http://blog.bombox.org/2016-09-11/sqlalchemy-start/](https://link.jianshu.com/?t=http%3A%2F%2Fblog.bombox.org%2F2016-09-11%2Fsqlalchemy-start%2F)
https://www.jianshu.com/p/fd32db13a0db
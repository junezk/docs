# Flask-SQLAlchemy

Flask-SQLAlchemy 是一个为您的 [Flask](http://flask.pocoo.org/) 应用增加 [SQLAlchemy](http://www.sqlalchemy.org/) 支持的扩展。它需要 SQLAlchemy 0.6 或者更高的版本。它致力于简化在 Flask 中 SQLAlchemy 的使用，提供了有用的默认值和额外的助手来更简单地完成常见任务。

## 用户指南

这部分文档将会展示如何开始在 Flask 中使用 Flask-SQLAlchemy。

- 快速入门
  - [一个最小应用](http://www.pythondoc.com/flask-sqlalchemy/quickstart.html#id2)
  - [简单的关系](http://www.pythondoc.com/flask-sqlalchemy/quickstart.html#id3)
  - [启蒙之路](http://www.pythondoc.com/flask-sqlalchemy/quickstart.html#id4)
- [引入上下文](http://www.pythondoc.com/flask-sqlalchemy/contexts.html)
- 配置
  - [配置键](http://www.pythondoc.com/flask-sqlalchemy/config.html#id2)
  - [连接 URI 格式](http://www.pythondoc.com/flask-sqlalchemy/config.html#uri)
  - [使用自定义的元数据和命名约定](http://www.pythondoc.com/flask-sqlalchemy/config.html#id4)
- 声明模型
  - [简单示例](http://www.pythondoc.com/flask-sqlalchemy/models.html#id2)
  - [一对多(one-to-many)关系](http://www.pythondoc.com/flask-sqlalchemy/models.html#one-to-many)
  - [多对多(many-to-many)关系](http://www.pythondoc.com/flask-sqlalchemy/models.html#many-to-many)
- 选择(Select),插入(Insert), 删除(Delete)
  - [插入记录](http://www.pythondoc.com/flask-sqlalchemy/queries.html#id1)
  - [删除记录](http://www.pythondoc.com/flask-sqlalchemy/queries.html#id2)
  - [查询记录](http://www.pythondoc.com/flask-sqlalchemy/queries.html#id3)
  - [在视图中查询](http://www.pythondoc.com/flask-sqlalchemy/queries.html#id4)
- 绑定多个数据库
  - [示例配置](http://www.pythondoc.com/flask-sqlalchemy/binds.html#id2)
  - [创建和删除表](http://www.pythondoc.com/flask-sqlalchemy/binds.html#id3)
  - [引用绑定(Binds)](http://www.pythondoc.com/flask-sqlalchemy/binds.html#id4)
- [信号支持](http://www.pythondoc.com/flask-sqlalchemy/signals.html)

## API 参考

如果您在寻找一个特定函数、类或者方法的信息，那么这部分文档就是为您准备的。

- API
  - [配置](http://www.pythondoc.com/flask-sqlalchemy/api.html#id1)
  - [模型](http://www.pythondoc.com/flask-sqlalchemy/api.html#id2)
  - [会话](http://www.pythondoc.com/flask-sqlalchemy/api.html#id3)
  - [实用工具](http://www.pythondoc.com/flask-sqlalchemy/api.html#id4)
# MySQL用户与权限管理

在这篇文章中，我们来聊聊怎么管理`MySQL`的用户及如何为不同用户分配不同的管理权限，其实，在日常使用`MySQL`的过程中，这部分的工作是由`DBA`(数据管理员)来完成的。

而我们作为一般的开发人员，只要拿到分配好的账号和密码连接到`MySQL`服务器访问特定的数据库，并不涉及到用户管理的内容。

不过，今天我们不妨切换到一个`DBA`视角，一起来详细了解`MySQL`有关用户及权限管理的相关知识。

## 权限数据表

`MySQL`中与系统用户及用户权限管理有关的信息都存储在系统数据库`mysql`中,这些数据表需要经过数据初始化，每个数据表有特定的数据结构。

### 数据表初始化

在安装好`MySQL`之后，需要初始化`MySQL`系统数据库，比如我们一般安装好`MySQL`之后，`MySQL`会帮我们创建默认的超级管理员`root`，我们也可以使用下面的语句在初始化时创建其他账号：

> MySQL支持很多种不同的安装方式，一般情况下会帮我们初始化mysql系统数据库。

```
bin/mysqld --initialize --user=mysql
bin/mysqld --initialize-insecure --user=mysql
```

### 权限表简介

在`MySQL`系统数据库`mysql`中与用户及权限相关的数据表一共有6张，下面表格这几张数据表的说明。

> 系统数据库mysql中除了与权限管理有关的数据外，还有其他的数据表。

| 数据表       | 说明                             |
| ------------ | -------------------------------- |
| user         | 用户帐户，全局特权和其他非特权列 |
| db           | 数据库级别的管理权限             |
| tables_priv  | 数据表级别的管理权限             |
| columns_priv | 数据列级别的管理权限             |
| procs_priv   | 存储过程及函数的使用权限         |
| proxies_priv | 代理用户权限                     |

### 权限表的结构

下面我们来看这些数据表的更详细表结构，我们看到下面的数据表中的字段可以大体分为四类，分别为`scope columes`,`Privilege columns`,`Security columns`,`Resource control columns`。

`scope columes`用于限定是某个用户，某个数据库或者某个数据表，`Privilege columns`表示权限，在`user`表中表示全局权限，在`db`表中表示对某个数据库的操作权限，而`Security columns`与`Resource control columns`的字段则为`user`表所独有。

#### user与db数据表

`user`数据表存储的是用户的账号信息及全局权限信息，而`db`数据表存储的是用户对于具体数据库的操作权限。

| 表名                         | user                   | db                    |
| ---------------------------- | ---------------------- | --------------------- |
| **Scope columns**            | Host                   | Host                  |
|                              | User                   | Db                    |
|                              |                        | User                  |
| **Privilege columns**        | Select_priv            | Select_priv           |
|                              | Insert_priv            | Insert_priv           |
|                              | Update_priv            | Update_priv           |
|                              | Delete_priv            | Delete_priv           |
|                              | Index_priv             | Index_priv            |
|                              | Alter_priv             | Alter_priv            |
|                              | Create_priv            | Create_priv           |
|                              | Drop_priv              | Drop_priv             |
|                              | Grant_priv             | Grant_priv            |
|                              | Create_view_priv       | Create_view_priv      |
|                              | Show_view_priv         | Show_view_priv        |
|                              | Create_routine_priv    | Create_routine_priv   |
|                              | Alter_routine_priv     | Alter_routine_priv    |
|                              | Execute_priv           | Execute_priv          |
|                              | Ttrigger_priv          | Ttrigger_priv         |
|                              | Event_priv             | Event_priv            |
|                              | Create_tmp_table_priv  | Create_tmp_table_priv |
|                              | Lock_tables_priv       | Lock_tables_priv      |
|                              | References_priv        | References_priv       |
|                              | Reload_priv            |                       |
|                              | Shutdown_priv          |                       |
|                              | Process_priv           |                       |
|                              | File_priv              |                       |
|                              | Show_db_priv           |                       |
|                              | Super_priv             |                       |
|                              | Repl_slave_priv        |                       |
|                              | Repl_client_priv       |                       |
|                              | Create_user_priv       |                       |
|                              | Create_tablespace_priv |                       |
| **Security columns**         | ssl_type               |                       |
|                              | ssl_cipher             |                       |
|                              | x509_issuer            |                       |
|                              | x509_subject           |                       |
|                              | plugin                 |                       |
|                              | authentication_string  |                       |
|                              | password_expired       |                       |
|                              | password_last_changed  |                       |
|                              | password_lifetime      |                       |
|                              | account_locked         |                       |
| **Resource control columns** | max_questions          |                       |
|                              | max_updates            |                       |
|                              | max_connections        |                       |
|                              | max_user_connections   |                       |

上面是`db`与`user`数据表的对比，在上面的对比中，我们看到`db`数据表与`user`有相同的字段，这些字段在`db`数据表，表示某个用户对某个数据表有哪些权限，而在`user`数据表中，则表示用户的全局权限。

#### tables_priv与columns_priv数据表

tables数据用于存储用户对数据表的权限，而columns_priv数据用于存储用户对数据列的权限。

| 表名                  | tables_priv | 与columns_priv |
| --------------------- | ----------- | -------------- |
| **Scope columns**     | Host        | Host           |
|                       | Db          | Db             |
|                       | User        | User           |
|                       | Table_name  | Table_name     |
|                       |             | Column_name    |
| **Privilege columns** | Table_priv  | Column_priv    |
|                       | Column_priv |                |
| **Other columns**     | Timestamp   | Timestamp      |
|                       | Grantor     |                |

#### procs_priv数据表

`procs_priv`数据表用于存储用户对自定义函数以及存储过程的权限，其表结构比较简单，如下所示：

| 表名              | procs_priv   |
| ----------------- | ------------ |
| Scope columns     | Host         |
|                   | Db           |
|                   | User         |
|                   | Routine_name |
|                   | Routine_type |
| Privilege columns | Proc_priv    |
| Other columns     | Timestamp    |
|                   | Grantor      |

在上面的字段中，`Routine_type`为`ENUM`类型，其取值为`FUNCTION`或`PROCEDURE`，而`Timestamp`和`Grantor`两个字段暂时没有什么作用。

#### proxies_priv数据表

`proxies_priv`存储的是代理用户的关系数据，什么是代理用户？可以简单地理解为一个用户将自己的权限授予给被另一个用户使用，比较A用户通过代理将权限授予B用户使用，那么B用户则称为代理用户。

`proxies_priv`数据表包含`Host`,`User`,`Proxied_host`,`Proixed_user`,`Grantor`,`Timestamp`,`With_grant`等字段。

## 用户管理

介绍了与权限管理相关的数据表之后，我们也知道`MySQL`将用户存储在哪个数据表中，以及`MySQL`如何组织用户的权限信息，接来在用户管理这块内容中，我们来了解的是`MySQL`数据库系统中，如何识别一个用户，如何创建、修改、删除用户以及如何为用户设置密码。

### 用户

`MySQL`的用户由`用户名(User)`和`主机名(Host)`组成，主机名表示允许该用户从哪台主机连接到`MySQL`服务器，所以在`MySQL`中，表示一个用户的格式如下所示：

```
# username表示用户名，host_name表示主机名
'user_name'@'host_name'
```

比如我使用`root`账号在`MySQL`本地服务器登录，那么此时的`root`账号的用户为：

```
'root'@'localhost'
```

另外，在`MySQL`中，我们可以使用`USER()`和`CURRENT_USER()`函数或者`CURRENT_USER`查询当前的用户，如：

```
mysql> SELECT USER();
mysql> SELECT CURRENT_USER();
mysql> SELECT CURRENT_USER;
```

上面两个语句的查询结果为：

```
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)

+----------------+
| current_user() |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)

+----------------+
| current_user   |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
```

用户名和主机名可以唯一地确定一个用户，所以即使用户名相同，主机名不同时，也表示两个不同的用户，比如下面的例子中用户名虽然都是`test`，由于主机名不同所以是两个用户。

```
'test'@'192.168.0.1'
'test'@'192.168.0.2'
```

当我们在创建账号没有指定主机时，则主机名的值为`'%'`，这表示允许账号从任意主机连接到`MySQL`服务器，比如：

```
'test'@'%'
```

在上面的示例中，用户名和主机都是使用单引号括起来，不过，如果没有特殊的字符，可以不使用单引号，所以上面的用户可以这样表示：

```
test@localhost
```

如果带有横杠(-),点(.),百分号(%),空格等特殊字符，则必须使用单引号，如：

```
# 正确示例
test@'192.168.0.1'
root@'%'

# 错误示例
test@192.168.0.1
test@%
```

从上面的示例中我们也看到，用户名和主机名是分别使用单引号括起来的，如果使用一个单引号把用户名和主机号括号，则MySQL会认为是整个当作用户名，而主机名则用默认的'%'，如下所示：

```
'root@localhost'

#上面的写法，MySQL会解析为
'root@localhost'@'%'
```

另外，为了提高安全性，也可以为用户设置登录密码，不过密码并不是必填的，不过为了安全，最好还是设置密码。

`MySQL`用户名的最大长度为`32`个字符，这个字符长度在`MySQL`程序中是硬编码的，所以即使我们修改了存储账号的`user`数据表用户名长度，也是没有效果的。

### 查询用户权限

查询用户权限分为两种，一种是查询自己当前登录用户的权限，一种是查询其他用户的权限。

```
SHOW GRANTS [FOR user]
```

#### 查询当前用户权限

不使用`FOR`子句的话，可以当查询当前登录用户的权限，比如我当前登录了root用户，使用下面的语句进行查询：

```
mysql> SHOW GRANTS;
```

结果如下：

```
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
+---------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

#### 查询其他用户权限

如果你的账号有查询其他用户信息的权限，那么可以使用`FOR`子句后面加上想查询用户的账号信息，不过要注意的是，需要有查询别人权限的权限，才能使用下面的语句。

```
mysql> SHOW GRANTS FOR 'test'@'localhost';
```

结果如下：

```
+------------------------------------------+
| Grants for 'test'@'localhost'                |
+------------------------------------------+
| GRANT USAGE ON *.* TO 'test'@'localhost' |
+------------------------------------------+
1 row in set (0.00 sec)
```

### 创建用户

`MySQ`L创建用户使用`CREATE USER`语句，该语句比较复杂，其语句结构如下所示：

```
CREATE USER [IF NOT EXISTS]
    user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH resource_option [resource_option] ...]
    [password_option | lock_option] ...

#参考我们上面的介绍
user:
    (see Section 6.2.4, “Specifying Account Names”)

auth_option: {
    IDENTIFIED BY 'auth_string'
  | IDENTIFIED WITH auth_plugin
  | IDENTIFIED WITH auth_plugin BY 'auth_string'
  | IDENTIFIED WITH auth_plugin AS 'auth_string'
  | IDENTIFIED BY PASSWORD 'auth_string'
}

tls_option: {
   SSL
 | X509
 | CIPHER 'cipher'
 | ISSUER 'issuer'
 | SUBJECT 'subject'
}

resource_option: {
    MAX_QUERIES_PER_HOUR count
  | MAX_UPDATES_PER_HOUR count
  | MAX_CONNECTIONS_PER_HOUR count
  | MAX_USER_CONNECTIONS count
}

password_option: {
    PASSWORD EXPIRE
  | PASSWORD EXPIRE DEFAULT
  | PASSWORD EXPIRE NEVER
  | PASSWORD EXPIRE INTERVAL N DAY
}

lock_option: {
    ACCOUNT LOCK
  | ACCOUNT UNLOCK
}
```

上面的语法看着好像很复杂，其实很多选项是可以选择的，比如最简单的，我们可以使用下面的语句创建一个用户：

```
mysql> CREATE USER 'test'@'localhost' IDENTIFIED BY '123456';
```

### 修改用户

`MySQL`修改用户的信息使用`ALTER USER`语句，其用法如下所示：

```
ALTER USER [IF EXISTS]
    user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH resource_option [resource_option] ...]
    [password_option | lock_option] ...
```

从上面的用法中可以看出，`ALTER USER`语句和`CREATE USER`语句基本类似，各种选项基本相同，因此我们参考`CREATE USER`的用法。

比如我们要修改当前用户的密码，可以这样：

```
mysql> ALTER USER USER() IDENTIFIED BY 'test123457';
```

### 删除用户

MySQL删除用户使用`DROP USER`语句,该语句用法如下：

```
DROP USER [IF EXISTS] user [, user] ...
```

从上面的语法看现，MySQL支持删除一个或多个用户，其用法如下所示：

```
mysql> DROP USER 'test'@'test.example.com','test1'@'test1.example.com';
```

### 设置密码

如果在创建用户的时候没有指定密码，或者想重新设置某个用户的密码，可以使用`SET PASSWORD`语句，该语句的用法如下所示：

```
SET PASSWORD [FOR user] = password_option

password_option: {
    'auth_string'
  | PASSWORD('auth_string')
}
```

上面的语句中，`auth_string`表示未加密的明文密码，也可以使用PASSWORD()函数进行加密，不过在MySQL5.7.6之后不推荐这种用法，在未来的版本这种用法会移除。

设置密码语句中的`FOR子句`可以省略，如果省略则表示修改当前用户的密码，如：

```
SET PASSWORD = "123456"
```

使用`FOR子句`，则可以修改指定账号的密码，如下：

```
SET PASSWORD FOR CURRENT_USER() = "123456"
```

> 在`MySQL`用户管理中，并不推荐使用`SET PASSWORD`语句来设置用户密码，因为这个步骤在`CREATE USER`和`ALTER USER`便可以完成了。

## 授权管理

前面讲了`MySQL`权限的存储细节以及用户管理的相关知识，对这些有所了解之后，我们来详细了解如何给用户授权权限。

在`MySQL`中，对于用户的权限划分可以分为两种，一种是数据管理相关的权限，比如对数据库、数据表进行操作，创建存储过程和视图的权限，另一种是对用户的管理权限，比如创建、删除用户，为用户分配权限。

### 用户授权

`MySQL`为我们提供了`GRANT`语句进行授权管理，其语法结构如下所示：

```
GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH {GRANT OPTION | resource_option} ...]

GRANT PROXY ON user
    TO user [, user] ...
    [WITH GRANT OPTION]

object_type: {
    TABLE
  | FUNCTION
  | PROCEDURE
}

priv_level: {
    *
  | *.*
  | db_name.*
  | db_name.tbl_name
  | tbl_name
  | db_name.routine_name
}

user:
    (see Section 6.2.4, “Specifying Account Names”)

auth_option: {
    IDENTIFIED BY 'auth_string'
  | IDENTIFIED WITH auth_plugin
  | IDENTIFIED WITH auth_plugin BY 'auth_string'
  | IDENTIFIED WITH auth_plugin AS 'auth_string'
  | IDENTIFIED BY PASSWORD 'auth_string'
}

tls_option: {
    SSL
  | X509
  | CIPHER 'cipher'
  | ISSUER 'issuer'
  | SUBJECT 'subject'
}

resource_option: {
  | MAX_QUERIES_PER_HOUR count
  | MAX_UPDATES_PER_HOUR count
  | MAX_CONNECTIONS_PER_HOUR count
  | MAX_USER_CONNECTIONS count
}
```

#### 执行的条件

为别的用户授权，当前用户必须拥有授权的权限才可以，比如我们以普通用户`test`登录`MySQL`为别的用户分配权限，由于test没有这种权限，因此会报以下的错误：

```
ERROR 1045 (28000): Access denied for user 'test'@'localhost' (using password: NO)
```

#### 创建用户

使用`GRANT`进行授权时，如果该用户不存在，可以跟上`GRANT`语句后面跟上`IDENTIFIED BY`直接创建该用户，不过如果在创建用户时要设置更详细的信息，则应该使用`CREATE USER`语句来创建用户。

```
GRANT ALL ON *.* TO test_1@localhost IDENTIFIED BY "123456";
```

上面的语句在用户不存在，会自动创建用户。

#### 权限的级别

在分配权限时，想分配什么级别的权限，可以加在关键字`ON`后面,其格式如下所示：

```
 *
 *.*
 db_name.*
 db_name.tbl_name
 tbl_name
 db_name.routine_name
```

比如我想给test@localhost用户分配school数据库的student表查询权限，可以这样写：

```
mysql> GRANT SELECT ON school.student TO 'test'@'localhost'；
```

#### 权限列表

下面是使用GRANT语句可以授予的全部权限，不同的权限有不同的作用域，比如有的是全局权限，有的只作用于数据库等，如下所示：

| 权限                    | 权限说明及作用的级别                                         |
| ----------------------- | ------------------------------------------------------------ |
| ALTER[PRIVILEGES]       | 除了GRANT OPTION和PROXY之外，以指定的访问级别授予所有特权。  |
| ALTER                   | 修改权限，作用于全局，数据库，数据表                         |
| ALTER_ROUTINE           | 修改存储过程，作用于全局，数据库，存储过程                   |
| CREATE                  | 创建权限，作用于全局，数据库，数据表                         |
| CREATE_ROUTINE          | 创建存储过程的权限，作用于全局，数据库                       |
| CREATE_TABLESPACE       | 表空间和日志文件组的创建、更改、删除，全局权限               |
| CREATE_TEMPORARY_TABLES | 创建临时表的权限，作用于数据库，数据表                       |
| CREATE_USER             | 创建、删除，重命用和移除用户权限的权限，全局权限             |
| CREATE_VIEW             | 创建视图权限，作用于全局，数据库，数据表                     |
| DELETE                  | 删除数据权限，作用于全局，数据库，数据表                     |
| DROP                    | 删除数据库、数据表、视图的权限，作用于全局，数据库，数据表   |
| EVENT                   | 使用事件的权限，作用于全局，数据库                           |
| EXECUTE                 | 执行存储过程的权限，作用于全局，数据库，存储过程             |
| FILE                    | 读取或写入文件的权限，全局权限                               |
| GRANT_OPTION            | 允许授权或取消授权的权限，作用于全局，数据库，数据表，存储过程，代理 |
| INDEX                   | 使用索引的权限，作用于全局，数据库，数据表                   |
| INSERT                  | 写入权限，作用于全局，数据库，数据表，数据列                 |
| LOCK_TABLES             | 在执行SELECT时可以启动LOCK_TABLES的权限，全局或数据库级别的权限 |
| PROCESS                 | 使用SHOW PROCESSLIST查询全部存储过程的权限，全局权限         |
| PROXY                   | 启用用户代理，作用级别从用户到用户                           |
| REFERENCES              | 创建外健权限，作用于全局，数据库，数据表，数据列             |
| RELOAD                  | 启动FLUSH操作，全局权限                                      |
| REPLICATION CLIENT      | 使用户能够询问主服务器或从服务器在哪里，全局权限             |
| REPLICATION SLAVE       | 启用复制从属服务器以从主服务器读取二进制日志事件,全局权限    |
| SELECT                  | 查询权限，作用于全局，数据库，数据表，数据列                 |
| SHOW_DATABASES          | 查询全部数据库，全局权限                                     |
| SHOW_VIEWS              | 启用使用SHOW CREATE VIEW,作用于全局，数据库，数据表          |
| SHUTDOWN                | 关闭数据库服务器权限，全局权限                               |
| SUPER                   | 启用其他管理操作的使用，例如CHANGE MASTER TO，KILL，PURGE BINARY LOGS，SET GLOBAL和mysqladmin debug命令。 全局权限 |
| TRIGGER                 | 启用触发器的权限，作用于全局，数据库，数据表                 |
| UPDATE                  | 更新权限，作用于全局，数据库，数据表，数据列                 |
| USAGE                   | 无特权                                                       |

#### 创建超级管理员

使用All权限分配给用户时，用户虽然拥有所有数据库的操作权限，但并没有账号管理等权限，如果想创建一直跟root一样的超级管理员，可以加上`WITH GRANT OPTION`参数，如下所示：

```
mysql> GRANT ALL ON *.* TO 'super'@'localhost' WITH GRANT OPTION
```

### 删除用户权限

删除权限与上面的授权是反向操作，可以删除不同级别的权限，就像上面可以授予不同级别的权限一样。

删除权限使用`REVOKE`语句，其语法结构如下：

```
REVOKE
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    FROM user [, user] ...

REVOKE ALL [PRIVILEGES], GRANT OPTION
    FROM user [, user] ...

REVOKE PROXY ON user
    FROM user [, user] ...
```

删除数据库级别的权限：

```
mysql > REVOKE CREATE,DROP ON expenses.* FROM 'custom'@'host47.example.com';
```

删除用户的全部权限：

```
mysql> REVOKE ALL ON *.* FROM 'finley'@'%.example.com';
```

## 小结

`MySQL`作为数据库管理系统，里面保存企业的重要业务数据，因此保证数据库的安全性非常重要，如何保证数据库的安全性呢？用户和用户权限管理是一个很重要的方面。
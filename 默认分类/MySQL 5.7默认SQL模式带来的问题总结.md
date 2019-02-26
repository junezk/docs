# MySQL 5.7默认SQL模式带来的问题总结

在使用MySQL 5.7版本时，已经碰到了两起因为SQL_MODE的改变而导致的问题了，究其原因都是因为MySQL 5.7控制的更加严格了，所以在MySQL 5.6或MySQL 5.5有些SQL语句就无法在MySQL 5.7执行了（会给开发带来一个很诡异的问题，就是代码一样、环境一样、SQL一样、测试跟正式怎么就报错呢？）。

说一下我碰到的两起事件：

1、某些GROUP BY的SQL语句无法执行了。

2、创建表时使用日期数据类型指定的默认值为0000-00-00时报错。

先看一下，MySQL 5.6&MySQL 5.7默认的SQL模式。

    # MySQL 5.6 Default SQL_MODE;
    mysql> select @@sql_mode;
    +--------------------------------------------+
    | @@sql_mode                                 |
    +--------------------------------------------+
    | STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION |
    +--------------------------------------------+
    1 row in set (0.00 sec)
    
    # MySQL 5.7 Default SQL_MODE;
    mysql> select @@sql_mode;
    +-------------------------------------------------------------------------------------------------------------------------------------------+
    | @@sql_mode                                                                                                                                |
    +-------------------------------------------------------------------------------------------------------------------------------------------+
    | ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
    +-------------------------------------------------------------------------------------------------------------------------------------------+
    1 row in set (0.00 sec)

先简单说一下每个参数的含义：

ONLY\_FULL\_GROUP_BY  
在严格模式下，不要让GROUP BY部分中的查询指向未选择的列，否则报错。

NO\_ZERO\_DATE  
在严格模式，不要将’0000-00-00’做为合法日期。你仍然可以用IGNORE选项插入零日期。在非严格模式，可以接受该日期，但会生成警告。

NO\_ZERO\_IN_DATE  
在严格模式，不接受月或日部分为0的日期，对年不限制。如果使用IGNORE选项，我们为类似的日期插入’0000-00-00’。在非严格模式，可以接受该日期，但会生成警告。

ERROR\_FOR\_DIVISION\_BY\_ZERO  
在严格模式，在INSERT或UPDATE过程中，如果被零除(或MOD(X，0))，则产生错误(否则为警告)。如果未给出该模式，被零除时MySQL返回NULL。如果用到INSERT IGNORE或UPDATE IGNORE中，MySQL生成被零除警告，但操作结果为NULL。

NO\_AUTO\_CREATE_USER  
在严格模式下，防止GRANT自动创建新用户，除非还指定了密码。

NO\_ENGINE\_SUBSTITUTION  
如果需要的存储引擎被禁用或未编译，可以防止自动替换存储引擎。

STRICT\_TRANS\_TABLES  
为事务存储引擎启用严格模式，也可能为非事务存储引擎启用严格模式，非法数据值被拒绝，下面有详细说明。

严格模式控制MySQL如何处理非法或丢失的输入值。有几种原因可以使一个值为非法。例如，数据类型错误，不适合列，或超出范围。当新插入的行不包含某列的没有显示定义DEFAULT子句的值，则该值被丢失。

对于事务表，当启用STRICT\_ALL\_TABLES或STRICT\_TRANS\_TABLES模式时，如果语句中有非法或丢失值，则会出现错误。语句被放弃并滚动。

对于非事务表，如果插入或更新的第1行出现坏值，两种模式的行为相同。语句被放弃，表保持不变。如果语句插入或修改多行，并且坏值出现在第2或后面的行，结果取决于启用了哪个严格选项：

对于STRICT\_ALL\_TABLES，MySQL返回错误并忽视剩余的行。但是，在这种情况下，前面的行已经被插入或更新。这说明你可以部分更新，这可能不是你想要的。要避免这点，最好使用单行语句，因为这样可以不更改表即可以放弃。

对于STRICT\_TRANS\_TABLES，MySQL将非法值转换为最接近该列的合法值并插入调整后的值。如果值丢失，MySQL在列中插入隐式 默认值。在任何情况下，MySQL都会生成警告而不是给出错误并继续执行语句。

严格模式不允许非法日期，例如’2004-04-31’。它不允许禁止日期使用“零”部分，例如’2004-04-00’或”零”日期。要想禁止，应在严格模式基础上，启用NO\_ZERO\_IN\_DATE和NO\_ZERO_DATE SQL模式。

如果你不使用严格模式(即不启用STRICT\_TRANS\_TABLES或STRICT\_ALL\_TABLES模式)，对于非法或丢失的值，MySQL将插入调整后的值并给出警告。在严格模式，你可以通过INSERT IGNORE或UPDATE IGNORE来实现。

回答刚开始提出的两个问题？

1、某些GROUP BY的SQL语句无法执行了。

这是因为MySQL 5.7默认加入了ONLY\_FULL\_GROUP\_BY参数。在MySQL的sql\_mode是非ONLY\_FULL\_GROUP_BY语义时。一条select语句，MySQL允许target list中输出的表达式是除聚集函数或group by column以外的表达式，这个表达式的值可能在经过group by操作后变成undefined，例如：

    select * from tt group by id;

而对于语义限制都比较严谨的多家数据库，如SQLServer、Oracle、PostgreSql都不支持select target list中出现语义不明确的列，这样的语句在这些数据库中是会被报错的，所以从MySQL 5.7版本开始修正了这个语义，就是我们所说的ONLY\_FULL\_GROUP_BY语义。而正确的写法如下：

    select id from tt group by id;
    select id,max(age) from tt group by id;

所以ONLY\_FULL\_GROUP\_BY的语义就是确定select target list中的所有列的值都是明确语义，简单的说来，在ONLY\_FULL\_GROUP\_BY模式下，target list中的值要么是来自于聚集函数的结果，要么是来自于group by list中的表达式的值。更详细的请看：MySQL 5.7默认ONLY\_FULL\_GROUP_BY语义介绍

2、创建表时使用日期数据类型指定的默认值为0000-00-00时报错。

这是因为MySQL 5.7默认加入了NO\_ZERO\_DATE和NO\_ZERO\_IN\_DATE参数。通过上面的介绍，这两个参数还是有些区别的，NO\_ZERO\_DATE是完全匹配默认值为0000-00-00时才限制，而NO\_ZERO\_IN\_DATE是在匹配到月或日任意为00时限制（对年不限制）。

当数据类型为date或datetime时，并且使用了NO\_ZERO\_IN_DATE限制，那么会有如下几种情况：

    # 不合法默认值;
    CREATE TABLE `test` (`time` datetime NOT NULL DEFAULT '1111-00-01 00:00:00'
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
    
    # 合法默认值;
    CREATE TABLE `test` (`time` datetime NOT NULL DEFAULT '1111-01-01 00:00:00'
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
    
    # 合法默认值;
    CREATE TABLE `test` (`time` datetime NOT NULL DEFAULT '0000-01-01 00:00:00'
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

如果使用数据类型为timestamp就有点不太一样了。

timestamp类型取值范围：1970-01-01 00:00:00到2037-12-31 23:59:59。也就是说默认值最少需要调整为1970-01-02 00:00:00。

    CREATE TABLE `test` (
      `mid` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
      `start_time` timestamp NOT NULL DEFAULT '1970-01-02 00:00:00',
      `end_time` timestamp NOT NULL DEFAULT '2037-12-31 23:59:59',
      PRIMARY KEY (`mid`)
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

最大值或最小值在timestamp合法范围内才行。网上大概看了看，部分人碰到这个问题都是数据库升级的时候，老的表结构无法在MySQL 5.7应用了。如果想调整SQL模式，直接在配置文件写入sql_mode参数跟上对应的模式参数即可。

3、修改字段类型小于小于最长字段时报错。

当启用严格模式时”STRICT\_TRANS\_TABLES或STRICT\_ALL\_TABLES”时，控制MySQL如何处理非法或丢失的输入值。有几种原因可以使一个值为非法。例如，数据类型错误，不适合列，或超出范围。当新插入的行不包含某列的没有显示定义DEFAULT子句的值，则该值被丢失。

    mysql> select * from dd;
    +--------------------------------+
    | log                            |
    +--------------------------------+
    | 一二三四五六七八九十              |
    +--------------------------------+
    1 row in set (0.00 sec)
    
    mysql> set sql_mode='STRICT_TRANS_TABLES';
    Query OK, 0 rows affected (0.00 sec)
    
    mysql> alter table dd change column log log varchar(9);         
    ERROR 1265 (01000): Data truncated for column 'log' at row 1
    
    mysql> set sql_mode='';                                
    Query OK, 0 rows affected (0.00 sec)
    
    mysql> alter table dd change column log log varchar(9);
    Query OK, 1 row affected, 1 warning (0.02 sec)     
    Records: 1  Duplicates: 0  Warnings: 1
    
    MariaDB [dkey]> select * from dd;                               
    +-----------------------------+
    | log                         |
    +-----------------------------+
    | 一二三四五六七八九             |
    +-----------------------------+
    1 row in set (0.00 sec)

可以看到非严格模式，默认把多余数据给截断了。

### mysql v5.7 change

* explicit\_defaults\_for_timestamp=1
* 添加json

    ### sql_mode

    *   ONLY\_FULL\_GROUP_BY： 出现在select语句、HAVING条件和ORDER BY语句中的列，必须是GROUP BY的列或者依赖于GROUP BY列的函数列。

    *   NO\_AUTO\_VALUE\_ON\_ZERO： 该值影响自增长列的插入。默认设置下，插入0或NULL代表生成下一个自增长值。如果用户 希望插入的值为0，而该列又是自增长的，那么这个选项就有用了。

    *   STRICT\_ALL\_TABLES：严格模式
    *   STRICT\_TRANS\_TABLES： 严格模式，在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做限制

    *   NO\_ZERO\_IN_DATE： 这个模式影响了是否允许日期中的月份和日包含0。如果开启此模式，2016-01-00是不允许的，但是0000-02-01是允许的。它实际的行为受到 strict mode是否开启的影响1。

    *   NO\_ZERO\_DATE： 设置该值，mysql数据库不允许插入零日期。它实际的行为受到 strict mode是否开启的影响2。

    *   ERROR\_FOR\_DIVISION\_BY\_ZERO： 在INSERT或UPDATE过程中，如果数据被零除，则产生错误而非警告。如 果未给出该模式，那么数据被零除时MySQL返回NULL

    *   NO\_AUTO\_CREATE_USER： 禁止GRANT创建密码为空的用户

    *   NO\_ENGINE\_SUBSTITUTION： 如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常

    *   PIPES\_AS\_CONCAT： 将”||”视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样的，也和字符串的拼接函数Concat相类似

    *   ANSI\_QUOTES： 启用ANSI\_QUOTES后，不能用双引号来引用字符串，因为它被解释为识别符

    *   Strict SQL Mode: strict mode控制MySQL如何处理当SQL语句块在修改数值时出现不正确的值或者缺少值时的情况（是警告还是报错），会影响到非常多的方面.


mysql sql_mode:[https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar\_sql\_mode](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_sql_mode)  
转载原文链接:[http://www.ywnds.com/?p=8865](http://www.ywnds.com/?p=8865)
# Mysql 碎片知识



## [不同版本mysql语句不兼容原因](https://www.cnblogs.com/guochunyi/p/5888123.html)

一般是sql_mode不相同，可以认为规则不一致。（语法的变化非常小，一般可以忽略）

如果想要导入不同版本的数据。可以：

 手动处理一些导入错误或者采用其他的办法。

或者

 修改sql_mode。具体修改方法参考http://www.mysqlfaqs.net/mysql-client-server-commands/what-is-sql-mode-in-mysql-and-how-can-we-set-it

一般需要两个命令。

查看：select @@sql_mode

会话级别修改：set sql_mode='XXXX,YYYY'; －－重启或者其他客户端无法使用修改

​         set sql_mode=''; －－取消所有约束规则 －－重启或者其他客户端无法使用修改

global级别修改：set global sql_mode ='XXXX,YYYY'; －－重启无法使用修改

配置文件修改：

[mysqld]

sql_mode = STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
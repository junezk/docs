# mysql ON DUPLICATE KEY UPDATE 引起自增ID变化的解决办法



工作中负责开发的一个准实时的异步写入系统,需要每天将用户的听歌记录写入 [数据库](http://www.codercto.com/category/database.html) 中,

写入量比较大,每天一个表的写入量大概有5000万次左右,有update,也有insert.

[数据库](http://www.codercto.com/category/database.html)用的是percona的[MySQL](http://www.codercto.com/category/mysql.html),上线后一直运行挺好,基本上都是实时的,但是突然有一天发现一个统计用户听歌次数的表数据不更新了,

update和insert 都不起作用了,非常的诡异,后来运维show create table的时候

发现表的数据引擎是这样的

```
ENGINE=InnoDB AUTO_INCREMENT=2147483647 DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT 
```

auto_increament 已经达到极限了.怎么会这样呢? 

设计表结构时,为了innodb的插入性能,表中使用了一个id自增主键,步长是2,表本身也有一个uniq的索引:key1

当前表中的数据大概有5000w条,相邻的2条的自增主键间隔并不是2,而是有很多的空洞,一直到目前AUTO_INCREMENT值已经达到极限,因此出现了这个情况.

赶紧和开发人员查了一下代码中的sql语句,发现所有的insert和update都使用了这条语句

```
insert into table1 (key1,key2,key3) values (x,x,x) on duplicated key update key2 = key2+1 ,key3=key3+1;
```

但是为什么这条语句会使数据之间产生这么大的空洞呢? 

手动试验了一下,

比如说当前表中有一个key1=1 的记录,运行这条语句

```
insert into table1 (key1,key2,key3) values (1,1,1) on duplicate key update key2 = key2+1 ,key3=key3+1
```

的时候,每次都是update,运行10次后,在运行一条

insert into table1 (key1,key2,key3) values (2,1,1) on duplicate key update key2 = key2+1 ,key3=key3+1

假设当前表中没有key1=2的数据,上面这条语句就会插入一条,这时候会发现插入的数据自增id并不是比key1大2,

而是我之前的每次update都把AUTO_INCREMENT增大了,所以造成了这种数据空洞以及只有5000w条记录的时候

AUTO_INCREMENT值已经到极限了.

后来翻查了一下 [mysql](http://www.codercto.com/category/mysql.html) 的手册,发现有这么一个配置

```
innodb_autoinc_lock_mode
```

这个有3中模式,0,1和2,mysql5的默认配置是1,

0是每次分配自增id的时候都会锁表.

1只有在bulk insert的时候才会锁表,简单insert的时候只会使用一个light-weight mutex,比0的并发性能高

2.没有仔细看,好像是很多的不保证...不太安全.

数据库默认是1的情况下,就会发生上面的那种现象,每次使用insert into .. on duplicate key update 的时候都会把简单自增id增加,不管是发生了insert还是update

而改为0后,就解决了这个问题,只有实际的发生insert的时候才增加,但是每次都会锁表,并发性不太好.

目前线上先是简单的把这个配置改成了0,观察了一下,高峰期的时候更新数据还是很实时的,没什么太大问题.

后序可能还要持续观察性能问题~~
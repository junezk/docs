# Redis在Web项目中的应用与实践

Redis作为一个开源的(BSD)基于内存的高性能存储系统，已经被各大互联网公司广泛使用，并且有着诸多的应用场景。本篇文章将基于PHP来详细讲解Redis在Web项目中的主要应用与实践。

### 缓存

这里所介绍的缓存是指可以丢失或过期的数据。常用的命令有 `set`, `hset`, `get`, `hget`，使用redis作为缓存时需要注意一下几个问题:

*   由于redis的可用内存是有限的，不能容忍redis内存的无限增长，建议设置 `maxmemory` 最大内存。
*   在开启maxmemory的情况下，可以启用lru机制，设置key的expire，当到达Redis最大内存时，Redis会根据最近最少用算法对key进行自动淘汰。
*   Redis的持久化策略和Redis故障恢复时间是一个博弈的过程，如果你希望在发生故障时能够尽快恢复，应该启用dump备份机制，但这样需要更多的可用内存空间来进行持久化。如果能够容忍Redis漫长的故障恢复时间，可以使用AOF持久化机制，同时关闭dump机制，这样不需要额外的内存空间。

### 存储

在web项目中，redis可存储读写非常频繁的数据来缓解MySQL等数据库的压力。redis如果作为存储系统的话，为了防止数据丢失，持久化必须开启。

**典型场景**

*   **计数器**

计数器的需求非常普遍，例如微博点赞数、帖子收藏数、文章分享数、用户关注数等。

*   **社交列表**

比如使用Sets结构存储关注列表、收藏列表、点赞列表等。

*   **Session**

借助redis高性能的key-value存储，可将用户登录状态保存到redis中。

*   ...

### 队列

**简单队列**

一般使用redis的list结构作为队列，`rpush` 生产消息，`lpop` 消费消息，当 `lpop` 没有消息的时候，要进行适当的sleep操作。

    $queueKey = "queue";
    
    // 生产者
    $redis->rpush($queueKey, $data)
    
    // 消费者
    while (true) {
        $data = $redis->lpop($queueKey);
        if (null === $data) {
            usleep(100000);
            continue;
        }
        // 业务逻辑
        ...
    }
    复制代码

由于没有消息时使用的sleep事件不好控制，生产环境尽量不要使用sleep来休眠，可使用 `blpop` 来消费消息，在没有新消息的时候它会阻塞到消息到来。

**延时队列**

延时队列可使用redis的 `sorted set` 数据结构，使用时间戳作为 `score` ，消息内容作为 `member`，使用 `zadd` 命令来生产消息，消费者使用 `zrangebyscore` 命令获取指定时间之前的消息数据轮询进行处理。

    $queueKey = "queue";
    
    // 生产消息
    
    // 消费时间, 这里设置为1小时候
    $consumeTimestamp = time() + 3600;
    // $data需要添加随机串前缀(or后缀)，防止出现重复member被丢弃
    $data = $data . md5(uniqid(rand(), true));
    $redis->zadd($queueKey, $consumeTimestamp, $data);
    
    // 消费消息
    while (tue) {
        $arrData = $redis->zrangebyscore($queueKey, 0, time());
        if (!$arrData) {
            usleep(100000);
            continue;
        }
        // 业务逻辑
        foreach ($arrData as $data) {
            $data = substr($data, 0, strlen($data) - 32);
            
            // 消费$data
    
        }
    }
    复制代码

**多消费者**

使用pub/sub主题订阅者模式，可以实现1:N的消息队列。这种模式中在消费者下线的情况下，生产的消息会丢失，在这里不推荐使用。

> 需要强调的是不推荐使用redis作为消息队列服务，这不是redis的设计目标。如果一定要用可考虑 [disque](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fantirez%2Fdisque)，是由redis的作者开发。

### 分布式锁

分布式锁主要解决的几个问题:

*   互斥性: 同一时刻只能有一个服务(或应用)访问资源
*   安全性: 锁只能被持有该锁的服务(或应用)释放
*   容错: 在持有锁的服务crash时，锁仍能得到释放
*   避免死锁

**方案1**

我们可能会考虑使用 `setnx` 和 `expire` 命令来实现加锁，即当没有key存在时才会成功写入value:

    $lockStatus = $redis->setnx($lockKey, 1);
    if (1 === $lockStatus) {
        // 加锁成功，为锁设置超时时间
        $redis->expire($lockKey, 300);
    
        // 进行后续操作
    
    } elseif (0 === $lockStatus) {
        // 加锁失败
    } else {
        // 其他异常
    }
    复制代码

但这种操作不是原子性的，如果在进行setnx时服务崩溃，没有来得及对Key进行超时设置，该锁将一直无法释放。

**方案2**

我们推荐 `set key value [EX seconds] [PX milliseconds] [NX|XX]` 命令来进行加锁

* EX: key在多少秒之后过期
* PX：key在多少毫秒之后过期
* NX: 当key不存在的时候，才创建key，效果等同于setnx
* XX：当key存在的时候，覆盖key

    $lockStatus = $this->redis->set($lockKey, 1, "EX", 30, "NX");
    if ("OK" === $lockStatus) {
        // 加锁成功，可进行后续操作
        
        //业务逻辑执行完毕，释放锁
        $this->redis->del($lockKey);

    } elseif (null === $lockStatus) {
        // 加锁失败
    }
    复制代码

如上代码所示，如果 `set` 命令返回OK，那么客户端就可以获得锁（如果返回null，那么应用服务可以在一段时间之后重新尝试获取锁），并且可以通过 `del` 命令来释放锁。

此方法需要注意的问题:

*   a服务获得的锁（键key）已经由于已到过期时间被redis服务器删除，但是这个时候a服务还去执行DEL命令。而b服务经在a设置的过期时间之后重新获取了这个同样key的锁，那么a执行 `del` 就会释放了b服务加好的锁。
*   当同一时刻有大量的key过期的时候，删除key时会增加redis压力，会影响服务稳定。

可以通过如下优化使得上面的锁系统变得更加健壮：

*   不要设置固定的字符串，而是设置为随机的大字符串，可以称为token。
*   通过脚本删除指定锁的key，而不是 `del` 命令。
*   在设置key过期时间的时候加上一个随机值。

优化后的代码可参考如下:

    $lockToken = md5(uniqid(rand(), true));
    // 此处超时时间根据具体业务逻辑配置
    $expire = rand(280, 320);
    $lockStatus = $this->redis->set($lockKey, $lockToken, "EX", $expire, "NX");
    if ("OK" === $lockStatus) {
        // 加锁成功，可进行后续操作
        
        // 业务逻辑执行完毕，释放锁
        // 删除锁之前需要判断是否是自己上的锁
        $currentToken = $this->redis->get($lockKey);
        if ($currentToken === $lockToken) {
            $this->redis->del($lockKey);
        }
    
    } elseif (null === $lockStatus) {
        // 加锁失败
    }
    复制代码

### 计算

redis提供的原子自增减方法以及有序集合结构等可以承担一些计算任务，例如浏览量统计等。

**浏览计数**

文章浏览量+1

    $redis->incr($postsKey);
    复制代码

批量获取文章浏览量

    $arrPostsKey = [
        //...
    ];
    $arrPostsViewNum = $redis->mget($arrPostsKey);
    复制代码

**排行榜**

可以使用redis的有序集合来实现排行榜的功能，score作为权重排序并取前n条记录。

    // 存储数据
    $sortKey = "sort_key";
    $redis->zadd($sortKey, 100, "tom");
    $redis->zadd($sortKey, 80, "Jon");
    $redis->zadd($sortKey, 59, "Lilei");
    $redis->zadd($sortKey, 87, "Hanmeimei");
    
    // 获取排行
    
    // 由大到小排序
    $arrRet = $redis->zrevrange($sortKey, 0, -1, true);
    
    // 由小到大排序
    $arrRet = $redis->zrange($sortKey, 0, -1, true);
    复制代码

### 结尾

redis涉及的应用实践非常繁多的，由于篇幅所限无法全部顾及，本文只针对web应用中最常用的几个场景进行了展开介绍，渴望进一步拓展redis知识的同学可参考以下链接进一步学习。

*   [Redis官网](https://link.juejin.im?target=https%3A%2F%2Fredis.io%2F)
*   [Antirez](https://link.juejin.im?target=http%3A%2F%2Fantirez.com)

> 原文链接：[Redis在Web项目中的应用与实践](https://link.juejin.im?target=https%3A%2F%2Frsy.me%2Fposts%2Fredis-application-in-web-development%2F)
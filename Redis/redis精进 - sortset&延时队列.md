# redis精进 - sortset&延时队列

最近精进学习Redis，边学边写

> 先赞后读，养成习惯

## 一、SortSet类型使用说明

zset 可能是 Redis 提供的最为特色的数据结构，它也是在面试中面试官最爱问的数据结构。

- 一方面它是set，保证 value 的唯一性，
- 一方面它可以给每个 value 一个 score，代表排序权重。

> 它的内部实现用的是一种叫做「跳跃列表」的数据结构。

## 二、SortSet常用命令

> zset 中最后一个 value 被移除后，数据结构自动删除，内存被回收。

```
> zadd books 9.0 "think in java"
> zadd books 8.9 "java concurrency"
> zadd books 8.6 "java cookbook"

> zrange books 0 -1     # 按 score 排序列出，参数区间为排名范围
1) "java cookbook"
2) "java concurrency"
3) "think in java"

> zrevrange books 0 -1  # 按 score 逆序列出，参数区间为排名范围
1) "think in java"
2) "java concurrency"
3) "java cookbook"

> zcard books           # 相当于 count()
(integer) 3

> zscore books "java concurrency"   # 获取指定 value 的 score
"8.9000000000000004"                # 内部 score 使用 double 类型进行存储，所以存在小数点精度问题

> zrank books "java concurrency"    # 排名
(integer) 1

> zrangebyscore books 0 8.91        # 根据分值区间遍历 zset
1) "java cookbook"
2) "java concurrency"

> zrangebyscore books -inf 8.91 withscores  # 根据分值区间 (-∞, 8.91] 遍历 zset，同时返回分值。inf 代表 infinite，无穷大的意思。
1) "java cookbook"
2) "8.5999999999999996"
3) "java concurrency"
4) "8.9000000000000004"

> zrem books "java concurrency"             # 删除 value
(integer) 1
> zrange books 0 -1
1) "java cookbook"
2) "think in java"
```

## 三、使用场景

### 排行榜

- 粉丝列表，value 值是粉丝的用户 ID，score 是关注时间
- 视频网站需要对用户上传的视频做排行榜，榜单维护可能是多方面：按照时间、按照播放量、按照获得的赞数等。

恰巧之前写的 [《Redis实践热门文章列表》](https://juejin.im/post/5da713536fb9a04e092614f5) 就是很好的例子

### 权重队列 / 延时队列

比如：订单超时未支付，取消订单，恢复库存.

> 对消息队列有严格要求(不能丢)的建议还是使用kafka,专业的MQ。这些专业的消息中间件提供了很多功能特性，当然他的部署使用维护都是比较麻烦的。如果你对消息队列没那么高要求，想要轻量级的，使用Redis就没错啦。

### PHP - 延时队列实例

下面代码如何增强健壮性：

- `function run()`需要 lua 来实现器`原子性`操作
- 可以考虑 在MySQL，MongoDB等持久数据库建立 任务队列 & 已执行队列 进一步容错

```php
<?php

trait RedisConnectTrait
{
    private $servers = array();
    private $instances = array();

    /**
     * 设置 Redis 配置
     * @param array $serversConfig
     * @return $this
     * @throws Exception
     */
    private function setServers( array $serversConfig = [ [ '127.0.0.1', 6379, 0.01 ] ] )
    {
        if ( !$serversConfig )
            throw new \Exception( 'Redis链接配置不能为空', false );

        $this->servers = $serversConfig;
        return $this;
    }

    private function initInstances()
    {
        if (empty($this->instances)) {
            foreach ($this->servers as $server) {
                list($host, $port, $timeout) = $server;

                $redis = new \Redis();
                $redis->connect($host, $port, $timeout);
                // $redis->select( ['index'] );

                $this->instances[] = $redis;
            }
        }
    }
}

class RedisDelayQueueUtil
{
    use RedisConnectTrait;

    const QUEUE_PREFIX = 'delay_queue:';
    protected $redis = null;
    protected $key = '';

    public function __construct( string $queueName, array $config = [] )
    {
        $instances = $this->setServers( $config )->initInstances();

        $this->key = self::QUEUE_PREFIX . $queueName;
        $this->redis = $instances[ 0 ];
        // $this->redis->auth($config['auth']);
    }

    public function delTask($value)
    {
        return $this->redis->zRem($this->key, $value);
    }

    public function getTask()
    {
        //获取任务，以0和当前时间为区间，返回一条记录
        return $this->redis->zRangeByScore( $this->key, 0, time(), [ 'limit' => [ 0, 1 ] ] );
    }

    public function addTask($name, $time, $data)
    {
        //添加任务，以时间作为score，对任务队列按时间从小到大排序
        return $this->redis->zAdd(
            $this->key,
            $time,
            json_encode([
                'task_name' => $name,
                'task_time' => $time,
                'task_params' => $data,
            ], JSON_UNESCAPED_UNICODE )
        );
    }

    public function run()
    {
        //每次只取一条任务
        $task = $this->getTask();
        if (empty($task)) {
            return false;
        }

        $task = $task[0];
        if ($this->delTask($task)) {
            $task = json_decode($task, true);
            //处理任务
            echo '任务：' . $task['task_name'] . ' 运行时间：' . date('Y-m-d H:i:s') . PHP_EOL;

            return true;
        }

        return false;
    }
}

$dq = new RedisDelayQueueUtil('close_order', [
    'host' => '127.0.0.1',
    'port' => 6379,
    'auth' => '',
    'timeout' => 60,
]);

$dq->addTask('close_order_111', time() + 30, ['order_id' => '111']);
$dq->addTask('close_order_222', time() + 60, ['order_id' => '222']);
$dq->addTask('close_order_333', time() + 90, ['order_id' => '333']);

set_time_limit(0);

$i2Count = 0;
while ( 10 < $i2Count ) {
    $dq->run();
    usleep(100000);
    $i2Count++;
}
```
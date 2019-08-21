# RabbitMQ 的 Docker 部署



关于RabbitMQ的一个重要注意事项是它根据所谓的“节点名称”存储数据，默认为主机名。在Docker中使用这意味着我们应该为每个守护进程明确指定`-h`/ `--hostname，这样我们就不会获得随机主机名并且可以跟踪我们的数据：

```bash
$ docker run -d --hostname my-rabbit --name some-rabbit rabbitmq:3
```

这将启动一个侦听默认端口5672的RabbitMQ容器。如果你给它一分钟，那么`docker logs some-rabbit`你会在输出中看到类似于的块：

```bash
=INFO REPORT==== 6-Jul-2015::20:47:02 ===
node           : rabbit@my-rabbit
home dir       : /var/lib/rabbitmq
config file(s) : /etc/rabbitmq/rabbitmq.config
cookie hash    : UoNOcDhfxW9uoZ92wh6BjA==
log            : tty
sasl log       : tty
database dir   : /var/lib/rabbitmq/mnesia/rabbit@my-rabbit
```

请注意`database dir`那里，尤其是它的“节点名称”附加到文件存储的末尾。`/var/lib/rabbitmq`默认情况下，此图像会生成所有卷。

### 内存限制

RabbitMQ包含明确跟踪和管理内存使用的功能，因此需要了解cgroup强加的限制。

上游配置设置为`vm_memory_high_watermark`，文档中的[“Memory Alarms”](https://www.rabbitmq.com/memory.html)中对此进行了描述。

在此图像中，此值通过设置`RABBITMQ_VM_MEMORY_HIGH_WATERMARK`。此环境变量的值解释如下：

- `0.49`被视为`49%`，就像上游（`{ vm_memory_high_watermark, 0.49 }`）
- `56%`被视为`56%`（`0.56`; `{ vm_memory_high_watermark, 0.56 }`）
- `1073741824`被视为绝对字节数（`{ vm_memory_high_watermark, { absolute, 1073741824 } }`）
- `1024MiB`被视为具有单位（`{ vm_memory_high_watermark, { absolute, "1024MiB" } }`）的绝对字节数

主要的行为差异在于如何处理百分比。如果当前容器具有内存限制（`--memory`/ `-m`），则将根据内存限制将百分比值计算为绝对字节值，而不是按原样传递给RabbitMQ。例如，对于一个容器运行`--memory 2048m`（以及隐含的上游默认`RABBITMQ_VM_MEMORY_HIGH_WATERMARK`的`40%`）将设置有效限制`819MB`（这是`40%`的`2048MB`）。

### Erlang Cookie

有关cookie及其必要原因的详细信息，请参阅[RabbitMQ“群集指南”](https://www.rabbitmq.com/clustering.html#erlang-cookie)。

要设置一致的cookie（特别适用于群集，也适用于远程/跨容器管理`rabbitmqctl`），请使用`RABBITMQ_ERLANG_COOKIE`：

```bash
$ docker run -d --hostname some-rabbit --name some-rabbit --network some-network -e RABBITMQ_ERLANG_COOKIE='secret cookie here' rabbitmq:3
```

然后可以从单独的实例使用它来连接：
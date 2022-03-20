## 使用 prom/prometheus 容器镜像

Running Prometheus on Docker is as simple as `docker run -p 9090:9090 prom/prometheus`. This starts Prometheus with a sample configuration and exposes it on port 9090.

The Prometheus image uses a volume to store the actual metrics. For production deployments it is highly recommended to use a [named volume](https://docs.docker.com/storage/volumes/) to ease managing the data on Prometheus upgrades.

To provide your own configuration, there are several options. Here are two examples.

### Volumes & bind-mount

Bind-mount your `prometheus.yml` from the host by running:

```bash
docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

Or bind-mount the directory containing `prometheus.yml` onto `/etc/prometheus` by running:

```bash
docker run \
    -p 9090:9090 \
    -v /path/to/config:/etc/prometheus \
    prom/prometheus
```

### Custom image

To avoid managing a file on the host and bind-mount it, the configuration can be baked into the image. This works well if the configuration itself is rather static and the same across all environments.

For this, create a new directory with a Prometheus configuration and a `Dockerfile` like this:

```Dockerfile
FROM prom/prometheus
ADD prometheus.yml /etc/prometheus/
```

Now build and run it:

```bash
docker build -t my-prometheus .
docker run -p 9090:9090 my-prometheus
```

A more advanced option is to render the configuration dynamically on start with some tooling or even have a daemon update it periodically.

普罗米修斯是一个系统和服务监控系统。它以给定的时间间隔从配置的目标收集指标，评估规则表达式，显示结果，并在观察到某些条件为真时触发警报。

与其他监控系统相比，普罗米修斯的主要特点是：

- **多维**数据模型（由指标名称和键/值维度集定义的时间序列）
- 一种**灵活的查询语言**，以利用这种维度
- 不依赖于分布式存储;**单个服务器节点是自治的**
- 时间序列收集通过 HTTP 上的**拉取模型**进行
- 通过中间网关支持**推送时间序列**
- 通过**服务发现**或**静态配置**发现目标
- 支持多种**模式的图形和仪表板**
- 支持分层和横向**联合**

## 使用此容器映像

在 Docker 上运行 Prometheus 就像 一样简单。这将从 Prometheus 开始一个示例配置，并在端口 9090 上公开它。`docker run -p 9090:9090 prom/prometheus`

Prometheus 映像使用卷来存储实际指标。对于生产部署，强烈建议使用[命名卷](https://docs.docker.com/storage/volumes/)来简化 Prometheus 升级时的数据管理。

要提供您自己的配置，有几个选项。下面是两个示例。

### 卷和绑定装载

通过运行以下命令从主机绑定装载您的：`prometheus.yml`

```bash
docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

或者通过运行以下命令将包含的目录绑定挂载到 上：`prometheus.yml``/etc/prometheus`

```bash
docker run \
    -p 9090:9090 \
    -v /path/to/config:/etc/prometheus \
    prom/prometheus
```

### 自定义镜像

为避免管理主机上的文件并绑定装载该文件，可以将配置烘焙到映像中。如果配置本身相当静态且在所有环境中都是相同的，则这很有效。

为此，请使用 Prometheus 配置创建一个新目录，如下所示：`Dockerfile`

```Dockerfile
FROM prom/prometheus
ADD prometheus.yml /etc/prometheus/
```

现在构建并运行它：

```bash
docker build -t my-prometheus .
docker run -p 9090:9090 my-prometheus
```

更高级的选项是使用某些工具在开始时动态呈现配置，甚至让守护程序定期更新它。

# Node exporter

Prometheus exporter for machine metrics, written in Go with pluggable metric collectors.

## Building and running

```
make
./node_exporter <flags>
```

## Running tests

```
make test
```

## Available collectors

By default the build will include the native collectors that expose information from /proc.

Which collectors are used is controlled by the flag.`--collectors.enabled`

### Enabled by default

| Name       | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| attributes | Exposes attributes from the configuration file. Deprecated, use textfile module instead. |
| diskstats  | Exposes disk I/O statistics from /proc/diskstats.            |
| filesystem | Exposes filesystem statistics, such as disk space used.      |
| loadavg    | Exposes load average.                                        |
| meminfo    | Exposes memory statistics from /proc/meminfo.                |
| netdev     | Exposes network interface statistics from /proc/netstat, such as bytes transferred. |
| netstat    | Exposes network statistics from /proc/net/netstat. This is the same information as .`netstat -s` |
| stat       | Exposes various statistics from /proc/stat. This includes CPU usage, boot time, forks and interrupts. |
| textfile   | Exposes statistics read from local disk. The flag must be set.`--collector.textfile.directory` |
| time       | Exposes the current system time.                             |

### Disabled by default

| Name       | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| bonding    | Exposes the number of configured and active slaves of Linux bonding interfaces. |
| gmond      | Exposes statistics from Ganglia.                             |
| interrupts | Exposes detailed interrupts statistics from /proc/interrupts. |
| lastlogin  | Exposes the last time there was a login.                     |
| megacli    | Exposes RAID statistics from MegaCLI.                        |
| ntp        | Exposes time drift from an NTP server.                       |
| runit      | Exposes service status from [runit](http://smarden.org/runit/). |

## Textfile Collector

The textfile collector is similar to the [Pushgateway](https://github.com/prometheus/pushgateway), in that it allows exporting of statistics from batch jobs. It can also be used to export static metrics, such as what role a machine has. The Pushgateway should be used for service-level metrics. The textfile module is for metrics that are tied to a machine.

To use set the flag on the Node exporter. The collector will parse all files in that directory matching the glob using the [text format](http://prometheus.io/docs/instrumenting/exposition_formats/).`--collector.textfile.directory``*.prom`

To atomically push completion time for a cron job:

```
echo my_batch_job_completion_time $(date +%s) > /path/to/directory/my_batch_job.prom.$$
mv /path/to/directory/my_batch_job.prom.$$ /path/to/directory/my_batch_job.prom
```

To statically set roles for a machine using labels:

```
echo 'role{role="application_server"} 1' > /path/to/directory/role.prom.$$
mv /path/to/directory/role.prom.$$ /path/to/directory/role.prom
```

# 节点导出器

用于计算机指标的 Prometheus 导出器，使用可插入的指标收集器用 Go 编写。

## 构建和运行

```
make
./node_exporter <flags>
```

## 运行测试

```
make test
```

## 可用的收集器

默认情况下，生成将包括公开来自 /proc 的信息的本机收集器。

使用哪些收集器由标志控制。`--collectors.enabled`

### 默认启用

| 名字     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 属性     | 公开配置文件中的属性。已弃用，请改用文本文件模块。           |
| 磁盘统计 | 公开来自 /proc/diskstats 的磁盘 I/O 统计信息。               |
| 文件系统 | 公开文件系统统计信息，如使用的磁盘空间。                     |
| loadavg  | 公开平均负载。                                               |
| 记忆信息 | 公开来自 /proc/meminfo 的内存统计信息。                      |
| netdev   | 公开来自 /proc/netstat 的网络接口统计信息，例如传输的字节数。 |
| 网盘     | 公开来自 /proc/net/netstat 的网络统计信息。此信息与 相同。`netstat -s` |
| 统计     | 公开来自 /proc/stat 的各种统计信息。这包括 CPU 使用率、启动时间、分叉和中断。 |
| 文本文件 | 公开从本地磁盘读取的统计信息。必须设置标志。`--collector.textfile.directory` |
| 时间     | 公开当前系统时间。                                           |

### 默认情况下处于禁用状态

| 名字     | 描述                                                 |
| -------- | ---------------------------------------------------- |
| 粘 接    | 公开 Linux 绑定接口的已配置和活动从属程序的数量。    |
| 格蒙德   | 公开来自神经节的统计信息。                           |
| 中断     | 公开来自 /proc/interrupts 的详细中断统计信息。       |
| 最后登录 | 公开上次登录的时间。                                 |
| 巨型     | 公开来自 MegaCLI 的 RAID 统计信息。                  |
| ntp      | 公开来自 NTP 服务器的时间偏移。                      |
| 符文     | 从 [runit](http://smarden.org/runit/) 公开服务状态。 |

## 文本文件收集器

文本文件收集器类似于 [Pushgateway](https://github.com/prometheus/pushgateway)，因为它允许从批处理作业导出统计信息。它还可用于导出静态指标，例如计算机具有的角色。Pushgateway 应用于服务级别指标。文本文件模块用于绑定到计算机的指标。

要使用，请在节点导出器上设置标志。收集器将使用[文本格式](http://prometheus.io/docs/instrumenting/exposition_formats/)解析该目录中与 glob 匹配的所有文件。`--collector.textfile.directory``*.prom`

以原子方式推送 cron 作业的完成时间：

```
echo my_batch_job_completion_time $(date +%s) > /path/to/directory/my_batch_job.prom.$$
mv /path/to/directory/my_batch_job.prom.$$ /path/to/directory/my_batch_job.prom
```

要使用标签静态设置计算机的角色：

```
echo 'role{role="application_server"} 1' > /path/to/directory/role.prom.$$
mv /path/to/directory/role.prom.$$ /path/to/directory/role.prom
```

# Grafana Docker image

## Run the Grafana Docker container

Start the Docker container by binding Grafana to external port `3000`.

```bash
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

Try it out, default admin user credentials are admin/admin.

## 运行 Grafana Docker 容器

通过将 Grafana 绑定到外部端口来启动 Docker 容器。`3000`

```bash
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

试试看，默认的管理员用户凭据是管理员/管理员。

更多文档可以在 http://docs.grafana.org/installation/docker/ 上找到。
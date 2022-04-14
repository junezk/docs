# Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解)

## 一：前言

一个服务上线了后，你想知道这个服务是否可用，需要监控。假如线上出故障了，你要先于顾客感知错误，你需要监控。还有对数据库，服务器的监控，等等各层面的监控。
近年来，微服务架构的流行，服务数越来越多，监控指标变得越来越多，所以监控也变得越来越复杂，需要新的监控系统适应这种变化。

以前我们用zabbix，StatsD监控，但是随着容器化，微服务的流行，我们需要新的监控系统来适应这种变化。于是监控项目Prometheus就应运而生。

## 二：Prometheus介绍

### 介绍[#](https://www.cnblogs.com/jiujuan/p/13262380.html#4275837037)

- 网站地址：https://prometheus.io/
  https://prometheus.io/docs/introduction/overview/
  https://github.com/prometheus/docs
- github：[github.com/prometheus](https://github.com/prometheus)

Prometheus是一款基于时序数据库的开源监控告警系统，它是[SoundCloud](http://soundcloud.com/)公司开源的，SoundCloud的服务架构是微服务架构，他们开发了很多微服务，由于服务太多，传统的监控已经无法满足它的监控需求，于是他们在2012就着手开发新的监控系统。Prometheus的原作者[Matt T. Proud](https://github.com/matttproud)在2012年加入SoundCloud公司，他之前服务于Google公司，他从google的监控系统Borgmon中获取灵感，与另外一名工程师Julius Volz合作开发了开源监控系统Prometheus。（总之感觉是因为有了这个前google工程师到来，才有能力开发了Prometheus）。后来其他开发人员陆续加入了这个项目，并在 SoundCloud 内部继续开发，最终于 2015 年 1 月将其发布。后来在2016年，SoundCloud把它捐献给了[云原生基金会](https://www.cncf.io/)(Cloud Native Computing Foundation)，在它下面继续孵化。

Prometheus是用go语言开发。它的很多理念跟google的SRE不谋而合。所以有时间，可以去看看[google SRE](https://book.douban.com/subject/26875239/)那本书，可以更好的理解Prometheus。

### 主要特性（功能）

- 多维[数据模型](https://prometheus.io/docs/concepts/data_model/)（时序由 metric 名字和 k/v 的labels构成）
- 灵活的查询语言（[PromQL](https://prometheus.io/docs/querying/basics/)）
- 无依赖的分布式存储；单节点服务器都是自治的
- 采用 http 协议，使用pull模式拉取数据，简单易懂
- 监控目标，可以采用服务发现和静态配置方式
- 支持多种统计数据模型和界面展示。可以和Grafana结合展示。

## 三：Prometheus架构原理

### 架构

来自官方的一张架构图
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707173929262-886048779.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707173929262-886048779.png)

> 图片来自: https://prometheus.io/docs/introduction/overview/

**主要模块**：

- the main [Prometheus Server](https://github.com/prometheus/prometheus)，主要用于抓取数据和存储时序数据，另外还提供查询和 Alert Rule 配置管理。就是数据的采集和存储，用PromQL查询，报警配置。
- [client libraries](https://prometheus.io/docs/instrumenting/clientlibs/)，用于对接Prometheus Server，用于对接Prometheus Server，可以查询和上报数据。
- a [push gateway](https://github.com/prometheus/pushgateway)，用于批量，短期的监控数据的汇报总节点，主要用于业务数据汇报等。
- 各种汇报数据的 [exporters](https://prometheus.io/docs/instrumenting/exporters/)，例如汇报机器数据的node_exporter，汇报MondogDB信息的 [MongoDB_exporter](https://github.com/dcu/mongodb_exporter) 等等。
- 用于高级通知管理的 [alertmanager](https://github.com/prometheus/alertmanager) 。
- 各种各样的支持工具

### 怎么采集监控数据

要采集目标的监控数据，首先就要在被采集目标地方安装采集组件，这种采集组件被称为Exporter。prometheus.io官网上有很多这种exporter，官方 [exporter列表](https://prometheus.io/docs/instrumenting/exporters/)。

**采集完了怎么传输到Prometheus？**

采集了数据，要传输给prometheus。怎么做？
Exporter 会暴露一个HTTP接口，prometheus通过Pull模式的方式来拉取数据，会通过HTTP协议周期性抓取被监控的组件数据。
不过prometheus也提供了一种方式来支持Push模式，你可以将数据推送到Push Gateway，prometheus通过pull的方式从Push Gateway获取数据。

### 主要流程

1. Prometheus server定期从静态配置的 targets 或者服务发现的 targets 拉取数据（zookeeper，consul，DNS SRV Lookup等方式）
2. 当新拉取的数据大于配置内存缓存区的时候，Prometheus会将数据持久化到磁盘，也可以远程持久化到云端。
3. Prometheus通过PromQL、API、Console和其他可视化组件展示数据。Prometheus支持很多方式图表可视化，比如Grafana，自带的Promdash。它还提供HTTP API的查询方式，自定义输出。
4. Prometheus 可以配置rules，然后定时查询数据，当条件触发的时候，会将alert推送到配置的Alertmanager。
5. Alertmanager收到告警的时候，会根据配置，聚合，去重，降噪，最后发出警告。

## 四：安装Prometheus

要整好prometheus监控系统，还是有很多软件需要安装。
安装的主要组件如下：

- Prometheus Server
- 被监控对象exporter组件
- 数据可视化工具 Grafana
- 数据上报网关 push gateway
- 告警系统 Alertmanager

### 第1种：直接安装[#](https://www.cnblogs.com/jiujuan/p/13262380.html#3343126151)

到官网下载最新版的Prometheus，[下载地址](https://prometheus.io/download/)。
因为它是用go开发的，可以做到开箱即用。

```shell
wget https://github.com/prometheus/prometheu
```

s/releases/download/v2.19.2/prometheus-2.19.2.linux-amd64.tar.gz

解压：

```shell
tar xvfz prometheus-2.19.2.linux-amd64.tar.gz
```

运行启动：

```shell
cd ./prometheus-2.19.2.linux-amd64
./prometheus --version
./prometheus --config.file=prometheus.yml
```

### 第2种：docker镜像安装[#](https://www.cnblogs.com/jiujuan/p/13262380.html#1904792919)

1. 先在本机 /etc/docker/prometheus/ 下创建一个配置文件 vim prometheus.yml

```bash
Copyglobal:
  scrape_interval: 15s
  external_labels:
    monitor: 'first-monitor'
scrape_configs:
  - job_name: prometheus
    scrape_interval: 5s
    static_configs:
      - targets: ['127.0.0.1:9090']
```

官方有一个模板：[documentation/examples/prometheus.yml](https://github.com/prometheus/prometheus/blob/master/documentation/examples/prometheus.yml)

配置参数可以参考这里： [configuration](https://prometheus.io/docs/prometheus/2.19/configuration/configuration/)，选择你安装版本所对应的配置信息。

1. 执行下面docker命令：

> Copy提示：请提前安装好docker。

```shell
docker run --name=prometheus -d -p 9090:9090 -v /etc/docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

上面的命令看起来有点不容易理解，重新排列格式后：

```shell
docker run --name=prometheus -d -p 9090:9090 \
-v /etc/docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
prom/prometheus
```

说明：

- -p 9090:9090，用这个接口可以查看promethdus的web界面
- -v /etc/docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
  ，将服务器本地的prometheus配置文件挂载到docker目录 /etc/prometheus/ 下，这个就是prometheus在容器中默认加载配置文件位置。 `-v` 参数就是将本地的配置文件挂载到docker里面。

用上面的命令安装完后，会出来一个很长的id信息：

```shell
# docker run --name=prometheus -d -p 9090:9090 -v /etc/docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
cddca15bad0eea0c249cb4a5dfe1a148d7779a00b0dd514c654c5cddce4e951d
```

可以用 `docker inspect + id 前面部分信息` 这样的命令来查看容器运行时默认配置参数有哪些，这个信息内容很长，截取需要的部分来看：

```shell
# docker inspect cddc
[
{
"Id": "cddca15bad0eea0c249cb4a5dfe1a148d7779a00b0dd514c654c5cddce4e951d",
"Created": "2020-07-04T10:10:33.792265269Z",
"Path": "/bin/prometheus",
"Args": [
"--config.file=/etc/prometheus/prometheus.yml",
"--storage.tsdb.path=/prometheus",
"--web.console.libraries=/usr/share/prometheus/console_libraries",
"--web.console.templates=/usr/share/prometheus/consoles"
],
"State": {
"Status": "running",
"Running": true,
"Paused": false,
"Restarting": false,
"OOMKilled": false,
"Dead": false,
"Pid": 18313,
"ExitCode": 0,
"Error": "",
"StartedAt": "2020-07-04T10:10:34.13215448Z",
"FinishedAt": "0001-01-01T00:00:00Z"
},
"Image": "sha256:9f345bfa8fefdd9580d5bd951a99e105af38d6047878c4bfb7c5c0250f77998e",
"ResolvConfPath": "/var/lib/docker/containers/cddca15bad0eea0c249cb4a5dfe1a148d7779a00b0dd514c654c5cddce4e951d/resolv.conf",
"HostnamePath": "/var/lib/docker/containers/cddca15bad0eea0c249cb4a5dfe1a148d7779a00b0dd514c654c5cddce4e951d/hostname",
"HostsPath": "/var/lib/docker/containers/cddca15bad0eea0c249cb4a5dfe1a148d7779a00b0dd514c654c5cddce4e951d/hosts",
```

可以看到上面的Args就是默认配置文件位置

#### 配置文件[#](https://www.cnblogs.com/jiujuan/p/13262380.html#2409977821)

prometheus主配置文件

- prometheus.yml ， 主配置文件，四大块：global，alerting，rule_files，scrape_config

其实它还有很多其他配置文件，比如rules.yml，你可能会问，上面没有看到rules.yml这个文件？是的上面没有加。可以用这个命令加上：
`-v /etc/docker/prometheus/rules.yml:/etc/prometheus/rules.yml`
其实跟加上promethdus.yml命令是一样的。

配置参数项以及说明可以参考这里： [configuration](https://prometheus.io/docs/prometheus/2.19/configuration/configuration/)，选择你安装版本所对应的配置信息。

配置说明：

```bash
Copyglobal:
  scrape_interval: 15s           #默认采集监控数据时间间隔
  external_labels:
    monitor: 'first-monitor'
 
scrape_configs:                #监控对象设置
  - job_name: prometheus #任务名称
    scrape_interval: 5s       #每隔5s获取一次监控数据
    static_configs:          #监控对象地址
      - targets: ['127.0.0.1:9090']
 
   - job_name: server-redis  # 还可以加其他监控对象
      static_configs:
        - targets: ['192.168.10.20:9100']
          labels:   # 标签
            instance: server-redis
```

#### 查看web界面[#](https://www.cnblogs.com/jiujuan/p/13262380.html#4224559292)

在浏览器上输入 http://127.0.0.1:9090/ , 如果显示下面的web界面，说明promethdus启动成功：
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174048583-2061290997.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707174048583-2061290997.png)

## 五：Exporter采集监控信息

前面已经讲过，如果要监控服务器或者应用程序的各种信息，比如cpu、内存、网卡流量等等。就要在监控目标上安装指标收集程序，并暴露HTTP接口供Prometheus拉取数据，这个指标收集程序就是Exporter。不同的指标需要不同的Exporter收集。

这种Exporter需要自己写吗？
一般不需要，官网上已经有大量的Exporter，上面我们已经列出过官网的[Exporter清单](https://prometheus.io/docs/instrumenting/exporters/) 地址。
而且有的软件已经集成了Prometheus的Exporter，也就是说软件本身就提供了Prometheus需要的各种指标数据。最典型的就是k8s，他们是云原生基金会的第一和第二个项目。

如果需要特殊的监控，可能就要你自己写Exporter了。

### 实例： node-exporter监控服务器[#](https://www.cnblogs.com/jiujuan/p/13262380.html#558718621)

上面prometheus已经安装好了，现在来安装一个Exporter监控实例。

来安装一个监控服务器主机cpu、内存和磁盘等信息的exporter，直接用[node-exporter](https://github.com/prometheus/node_exporter)。它主要用于收集类 UNIX 系统的信息。

**步骤：**

1.先修改prometheus.yml信息，

```bash
Copyglobal:
  scrape_interval: 15s
  external_labels:
    monitor: 'first-monitor'
scrape_configs:
  - job_name: prometheus
    scrape_interval: 5s
    static_configs:
      - targets: ['127.0.0.1:9090']
      - targets: ['127.0.0.1:9100'] # 这里开始增加的监控信息
        labels:
          group: 'local-node-exporter'
```

2.用docker安装并启动node-exporter：

> docker run -d --name=node-exporter -p 9100:9100 prom/node-exporter

3.然后重启docker prometheus，让刚才修改的配置生效：

> docker restart prometheus

4.在浏览器上直接输入: http://127.0.0.1:9090/targets。或者，你在界面上点击 Status 菜单 -> Targets 菜单，来浏览metrics信息。
如果你是在服务器上安装，那么这里的 127.0.0.1 就是服务器IP地址，或者域名。

浏览器输出的web界面如下：（我用的远程服务器测试，所以用了ip）
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174124052-1443937851.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707174124052-1443937851.png)

可以看到里面有一个 9100 端口的 metrics 连接，点进去后，就可以看到一些采集信息。
说明刚才配置的node-exporter已经加入到prometheus的targets中了。如下图：

![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707182854188-358918060.png)

#### 查看监控信息[#](https://www.cnblogs.com/jiujuan/p/13262380.html#1278923745)

点击web界面最上面的菜单 Graph
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174236950-1275462648.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707174236950-1275462648.png)

选择下面的 Graph，然后我们选择一个 node_memory_Active_bytes 来看一看

![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174421422-1072381951.png)

然后点击 Execute 按钮 , 就会出来如下图所示图形：
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174510243-505503629.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707174510243-505503629.png)

## 六：可视化系统：Grafana

上面我们通过Prometheus自带的UI，查看不同指标视图，但是它的功能很简单。如果需要强大的展示系统，能定制不同指标的面板，支持不同类型的展示方式，如曲线图、热点图，TopN等，那么[grafana](https://grafana.com/)比较合适。它可以对promethdus数据进行可视化的展示。

grafana是一个大型可视化系统，功能强大，可以创建自己的自定义面板，支持多种数据来源，
比如：OpenTSDB、Elasticsearch、Prometheus 等，可以到官网去查看[支持的数据源种类](https://grafana.com/grafana/plugins?type=datasource)，而且它[插件](https://grafana.com/grafana/plugins)也很多。

- [官网](https://grafana.com/)
- [doc](https://grafana.com/docs/)

### 安装[#](https://www.cnblogs.com/jiujuan/p/13262380.html#1801800864)

官网[安装文档](https://grafana.com/docs/grafana/latest/installation/requirements/)，它有不同平台安装的Doc。
我选择最简单的一种，直接用docker安装，命令如下：

> docker run -d -p 3000:3000 --name=grafana grafana/grafana

Docker安装完后，最后会出来一些提示信息：

> ... ...
> Digest:sha256:0e8b556a7fc9b95c03669509ec50be19c16b82b9e9078f79fa35a71f484bc047
> Status: Downloaded newer image for docker.io/grafana/grafana:latest 97d1c768ce6c541fa58790ec97fd06783633833cd9e74b12c16266dd264f8d0f

说明安装成功了。我们在浏览器上看看界面，输入下面地址：http://127.0.0.1:3000/login

![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174543250-845977115.png)

然后输入初始密码 admin/admin 登录进入。

> 我安装的版本是：`Grafana v7.0.5`

### grafana设置[#](https://www.cnblogs.com/jiujuan/p/13262380.html#820323070)

#### 增加prometheus数据源并展示[#](https://www.cnblogs.com/jiujuan/p/13262380.html#3555690621)

1.点击如下图的Data Source：
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174617612-1317641130.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707174617612-1317641130.png)

2.点击 Add data source 按钮后，出来下面界面：
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174648663-1081361439.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707174648663-1081361439.png)

3.鼠标移到 Prometheus 上，点击 Select 按钮：
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174711462-1808511410.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707174711462-1808511410.png)

4.prometheus相关设置：

![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174731690-759888346.png)

最主要设置获取数据的HTTP URL。

5.点击 save&test 按钮，它会提示你是否设置成功。

6.设置Dashboards
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174807602-164658600.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707174807602-164658600.png)

7.回到home

![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174828391-292368667.png)

8：点击 prometheus
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707174854170-1524054549.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707174854170-1524054549.png)

9：出来很多图表展示
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175339410-1784526624.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707175339410-1784526624.png)

#### 其他dashboard模板设置[#](https://www.cnblogs.com/jiujuan/p/13262380.html#3571660174)

grafana不仅有我们上面设置的那些图表模板，它还有其他很多模板，我们也可以设置。
官方[模板dashboard 地址](https://grafana.com/dashboards)。

比如我们查找node exportet的模板，[https://grafana.com/grafana/dashboards?search=node%20exporter](https://grafana.com/grafana/dashboards?search=node exporter)，有一个模板 downloads 比较多，
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175006759-467917087.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707175006759-467917087.png)

它的地址为：
https://grafana.com/grafana/dashboards/8919

我们在grafana上来设置这个dashboard，import进来：

![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175439423-633850176.png)

可以填写id和url，我们填写id，为 8919：

![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175556290-1505142659.png)

点击 load 出来下面界面：

![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175632464-372201266.png)

然后选择prometheus-1，点击 import， 出来如下图的界面：
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175717512-1359004360.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707175717512-1359004360.png)

## 七、告警通知[#](https://www.cnblogs.com/jiujuan/p/13262380.html#1259508208)

我们已经能够对收集的数据，通过grafana展示出来了，能查看数据。想一想，系统还缺失什么功能？

监控最重要的目的是什么？

- 第一：监控系统是否正常
- 第二：系统不正常时，可以告知相关人员及时的排查和解除问题，这就是[告警通知](https://prometheus.io/docs/alerting/overview/)。

所以，还缺一个告警通知的模块。
prometheus的告警机制由2部分组成：

1. 告警规则
   prometheus会根据告警规则rule_files，将告警发送给Alertmanager
2. 管理告警和通知
   模块是[Alertmanager](https://github.com/prometheus/alertmanager)。它负责管理告警，去除重复的数据，告警通知。通知方式有很多如Email、HipChat、Slack、WebHook等等。

### 配置[#](https://www.cnblogs.com/jiujuan/p/13262380.html#930585546)

#### 1.告警规则配置[#](https://www.cnblogs.com/jiujuan/p/13262380.html#3854271502)

告警文档地址：[告警规则官方文档](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)。

我们新创建一个规则文件：`alert_rules.yml`，把它和prometheus.yml放在一起，官方有一个模板 Templating，直接copy过来：

```bash
Copygroups:
- name: example
  rules:

  # Alert for any instance that is unreachable for >5 minutes.
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."

  # Alert for any instance that has a median request latency >1s.
  - alert: APIHighRequestLatency
    expr: api_http_request_latencies_second{quantile="0.5"} > 1
    for: 10m
    annotations:
      summary: "High request latency on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has a median request latency above 1s (current value: {{ $value }}s)"
```

上面规则文件大意：就是创建了2条alert规则 `alert: InstanceDown` 和 `alert: APIHighRequestLatency` ：

- InstanceDown 就是实例宕机（up==0）触发告警，5分钟后告警（for: 5m）;
- APIHighRequestLatency 表示有一半的 API 请求延迟大于 1s 时（api_http_request_latencies_second{quantile="0.5"} > 1）触发告警

> 更多rules规则说明，请看[这里 recording_rules](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/)。

然后把alrt_rules.yml添加到prometheus.yml 里：

![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175750992-2124795714.png)

我们要把alert_rules.yml规则映射到docker里：
先用docker ps查看prometheus容器ID, CONTAINER ID: ac99a89d2db6， 停掉容器 `docker stop ac99`，然后删掉这个容器 `docker rm ac99`。
重新启动容器：

> docker run --name=prometheus -d -p 9090:9090 -v /etc/docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml -v /etc/docker/prometheus/alert_rules.yml:/etc/prometheus/alert_rules.yml prom/prometheus

启动时主要添加这个参数：`-v /etc/docker/prometheus/alert_rules.yml:/etc/prometheus/alert_rules.yml`

然后在浏览器上查看，rules是否添加成功，在浏览器上输入地址 `http://127.0.0.1:9090/rules`
![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175834513-2093149858.png)

也可以查看alers情况，点击菜单 Alerts：
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175909205-1728356047.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707175909205-1728356047.png)

#### 告警通知配置[#](https://www.cnblogs.com/jiujuan/p/13262380.html#2527971900)

**alertmanager配置：**
[官方配置文档](https://prometheus.io/docs/alerting/latest/configuration/)，官方[配置例子](https://prometheus.io/docs/alerting/latest/configuration/#example)。

在上面我们可以看到alerts页面的告警信息，但是怎么通知到研发和业务相关人员呢？这个就是由Alertmanager完成，先配置alertmanager文件 alertmanager.yml，：

```bash
Copyglobal:
  resolve_timeout: 5m
route:
  group_by: ['example']  #与prometheus配置文件alert_rules.yml中配置规则名对应
  group_wait: 10s #报警等待时间
  group_interval: 10s #报警间隔时间
  repeat_interval: 1m #重复报警间隔时间
  receiver: 'web.hook' #告警处理方式，我们这里通过web.hook方式，也可以配置成邮件等方式
receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://127.0.0.1:8080/example/test' #告警web.hook地址，告警信息会post到该地址，需要编写服务接收该告警数据
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning' #目标告警状态
    equal: ['alertname', 'dev', 'instance']
```

启动alertmanager服务：

> docker run -d -p 9093:9093 --name alertmanager -v /etc/docker/prometheus/alertmanager.yml:/etc/prometheus/alertmanager.yml prom/alertmanager

在浏览器上输入 ： [http://127.0.0.1:9093](http://127.0.0.1:9093/)，出现下面界面：
[![img](Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解).assets/650581-20200707175952813-443825387.png)](https://img2020.cnblogs.com/blog/650581/202007/650581-20200707175952813-443825387.png)

**prometheus配置：**
在promethdus加上下面的配置，

```bash
Copyalerting:
  alertmanagers:
    - static_configs:
      - targets: ['127.0.0.1:9093'] 
```

配置说明：告诉prometheus，放生告警时，将告警信息发送到Alertmanager，Alertmanager地址为 [http://127.0.0.1:9093](http://127.0.0.1:9093/)
先docker rm 删除掉原来容器，在运行：

> docker run --name=prometheus -d -p 9090:9090 -v /etc/docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml -v /etc/docker/prometheus/alert_rules.yml:/etc/prometheus/alert_rules.yml prom/prometheus

再次运行http://127.0.0.1:9093，正常说明配置成功

**如果您看到这里，觉得还可以的话，随手点个 推荐，评论一下，让更多人看到**

## 八：参考链接[#](https://www.cnblogs.com/jiujuan/p/13262380.html#1911397811)

- https://prometheus.io/docs/introduction/overview/
- https://github.com/prometheus/prometheus
- https://www.aneasystone.com/archives/2018/11/prometheus-in-action.html
- https://github.com/prometheus/alertmanager
- https://github.com/songjiayang/prometheus_practice
- https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/
- https://grafana.com/
- https://grafana.com/plugins
- https://hub.docker.com/r/prom/prometheus/
- https://www.bookstack.cn/read/prometheus-manual/prometheus

[完]

作者： 九卷

出处：https://www.cnblogs.com/jiujuan/p/13262380.html

版权：本文采用「[署名-非商业性使用-相同方式共享 4.0 国际](https://creativecommons.org/licenses/by/4.0)」知识共享许可协议进行许可。
# 通过 Docker 快速搭建各种测试环境(Mysql, Redis, Elasticsearch, MongoDB)

> 个人网站: [www.exception.site](https://link.juejin.im/?target=https%3A%2F%2Fwww.exception.site)
>
> 小哈总结的 [GitHub《 Docker 学习教程》](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fweiwosuoai%2Fdocker-tutorial) 教程，持续更新中，不妨 Star 下吧

小哈今天给大家分享的主题是，如何通过 Docker 快速搭建各种测试环境，本文列举的，也是小哈在工作中经常用到的，其中包括 Mysql, Redis, Elasticsearch, MongoDB 安装步骤, 通过几行命令秒秒钟就能轻松搞定。

友情提示：搭建之前，你需要先安装 Docker 哟，本文基于您已经安装好 Docker 的基础上！

废话少说，正文开始！

## 目录

- 一、镜像加速
- 二、快速安装&搭建 Mysql 环境
- 三、快速安装&搭建 Redis 环境
- 四、快速安装&搭建 MongDB 环境
- 五、快速安装&搭建 Elasticsearch 环境
- 六、总结

## 一、镜像加速

Docker 默认是从官方镜像地址 Docker Hub 下下载镜像，由于服务器在国外的缘故，导致经常下载速度非常慢。为了提升镜像的下载速度，我们可以手动配置国内镜像加速器，让下载速度飚起来。

国内的镜像加速器选项较多，如：阿里云，DaoCloud 等。

本文主要说说如何配置阿里云的镜像加速器。

### 2.1 登录阿里云获取加速信息

1. 首先你需要注册一个阿里云账号，没有的话，通过下面的连接跳转注册：

[dev.aliyun.com/](https://link.juejin.im/?target=https%3A%2F%2Fdev.aliyun.com%2F)

1. 跳转镜像加速页 [cr.console.aliyun.com/](https://link.juejin.im/?target=https%3A%2F%2Fcr.console.aliyun.com%2F)，获取加速配置信息：

![阿里云Docker镜像加速页](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a8b6a203e6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)阿里云Docker镜像加速页

### 2.2 配置 Docker

#### 2.2.1 确定 Docker Client 版本

在配置之前，首先需要**确定 Docker Client 的版本，推荐是 1.10.0+**:

![确定docker client版本](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a8c3834961?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)确定docker client版本

#### 2.2.2 配置镜像加速器

> PS: 这里以 CentOS 系统为例，如果你是别的系统，可以参考阿里云配置加速器官方文档。

通过修改 daemon 配置文件 `/etc/docker/daemon.json` 来使用加速器：

执行下面命令：

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://bjtzu1jb.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 2.3 验证一下速度

以下载 mongodb 为例，看下速度：

![docker下载MongoDB](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a8b81ffe56?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)docker下载MongoDB

配置了加速器过后，速度终于飚起来了。

## 二、快速安装&搭建 Mysql 环境

本节中，我们将学习如何通过 Docker 快速安装与搭建 Mysql 环境。

### 2.1 下载 Mysql 镜像

这里以 Mysql 5.7 为例：

```
docker pull mysql:5.7
```

下载完成后，通过 `docker images` 检查一下镜像是否下载成功:

![验证本地是否下载好Mysql 5.7镜像](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a8b9402640?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)验证本地是否下载好Mysql 5.7镜像

### 2.2 先以最简单方式启动

先以简单的方式启动：

```
docker run -d \
--name mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.7
```

- `-d`：以后台的方式运行；
- `--name mysql`：指定容器的名称为 mysql;
- `-p 3306:3306` 将容器的 3306 端口挂载到宿主机的 3306 端口上；
- `-e MYSQL_ROOT_PASSWORD=123456`：指定 root 的密码为 123456

命令执行完成后，你也可以通过 `docker ps`命令来确认下容器是否启动成功。若成功，我们需要将容器中的目录文件复制到宿主机中，分别包括：

- mysql 配置文件；
- 数据存储目录，以便挂载(PS: 若不挂载到宿主机，每次启动容器数据都会丢失)

```
# 将容器中的 mysql 配置文件复制到宿主机中指定路径下，路径你可以根据需要，自行修改
docker cp mysql:/etc/mysql/mysql.conf.d/mysqld.cnf /usr/local/docker/mysql/config
# 将容器中的 mysql 存储目录复制到宿主机中
docker cp mysql:/var/lib/mysql/ /usr/local/docker/mysql/data
```

完成这一切后，让我们将刚刚运行的容器删除掉。

```
docker rm -f mysql
```

> PS: mysql 是我们运行容器时，指定的名称，当然，你也可以先执行 `docker ps`, 通过容器 ID 来删除。

### 2.3 正式运行 Mysql 容器

接下来，正式运行 Mysql 容器：

```
docker run -d \
--name mysql \
-p 3306:3306 \
-v /usr/local/docker/mysql/config/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf \
-v /usr/local/docker/mysql/data/mysql:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.7
```

其他不变，额外添加了两个挂载子命令：

- `-v /usr/local/docker/mysql/config/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf`: 将容器中 /etc/mysql/mysql.conf.d/mysqld.cnf 配置文件挂载到宿主机的 /usr/local/docker/mysql/config/mysqld.cnf 文件上；
- `-v /usr/local/docker/mysql/data:/var/lib/mysql`: 将容器中 /var/lib/mysql 数据目录挂载到宿主机的 /usr/local/docker/mysql/data 目录下；

执行命令完成后，查看下容器是否启动：

![查看mysql容器是否启动成功](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a8e3e87138?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)查看mysql容器是否启动成功

可以看到，容器运行成功

### 2.4 通过 Mysql 客户端连接一下试试

通过 MySQL 客户端连接刚刚创建的 mysql, 看看能否连接成功：

![通过mysql客户端连接试试看](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a9283994f3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)通过mysql客户端连接试试看

连接成功了！

## 三、快速安装&搭建 Redis 环境

本节中，我们将学习如何利用 Docker 安装&搭建 Redis 环境。

### 3.1 下载 Redis 镜像

首先拉取 Redis 镜像, 这里我选择的是 `redis:alpine` 轻量级镜像版本：

```
docker pull redis:alpine
```

下载完成后，通过 `docker images` 确认镜像是否已经下载到本地：

![查看redis镜像是否下载成功](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a938248435?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)查看redis镜像是否下载成功

### 3.2 运行 Redis 容器

```
docker run -p 6379:6379 --name redis -v /usr/local/docker/redis/redis.conf:/etc/redis/redis.conf -v /usr/local/docker/redis/data:/data -d redis:alpine redis-server /etc/redis/redis.conf --appendonly yes
```

命令说明：

- `-p 6379:6379`: 将容器的 6379 端口映射到宿主机的 6379 端口；
- `-v /usr/local/docker/redis/data:/data` : 将容器中的 /data 数据存储目录, 挂载到宿主机中 /usr/local/docker/redis/data 目录下；
- `-v /usr/local/docker/redis/redis.conf:/etc/redis/redis.conf` ： 将容器中 /etc/redis/redis.conf 配置文件，挂载到宿主机的 /usr/local/docker/redis/redis.conf 文件上；
- `redis-server --appendonly yes`: 在容器执行 redis-server 启动命令，并打开 redis 持久化配置;

命令运行完成后，查看容器是否启动成功：

![查看docker redis是否启动成功](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a957bf1758?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)查看docker redis是否启动成功

可以看到 redis 容器已经启动成功了！

### 3.3 连接刚刚创建好的容器

执行如下命令，连接 redis：

```
docker run -it redis:alpine redis-cli -h 172.17.0.1
```

![img](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a95dca86b5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 四、快速安装&搭建 MongDB 环境

本节中，我们将学习如何通过 Docker 快速安装与搭建 MongoDB 环境。

### 4.1 下载 MongoDB 镜像

这里以 mongo 4 版本为例，下载镜像：

```
docker pull mongo:4
```

下载完成后，确认一下镜像是否下载成功:

![验证mongodb镜像是否下载成功](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a97018f2af?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)验证mongodb镜像是否下载成功

### 4.2 运行 MongoDB 镜像

下载成功后，运行 mongoDB 镜像：

```
docker run -d \
--name mongo \
-v /usr/local/docker/mongo/configdb:/data/configdb \
-v /usr/local/docker/mongo/data:/data/db \
-p 27017:27017 \
mongo:4 \
--auth
```

- `-d`: 以后台的方式运行；
- `--name mongo`: 指定容器名称为 mongo;
- `-v /usr/local/docker/mongo/configdb:/data/configdb`: 将容器中 /data/configdb 目录挂载到宿主机的 /usr/local/docker/mongo/configdb 目录下；
- `-v /usr/local/docker/mongo/data:/data/db`: 将容器中 /data/db 数据目录挂载到宿主机的 /usr/local/docker/mongo/data 目录下；
- `-p 27017:27017`: 将容器的 27017 端口映射到宿主机的 27017 端口；

执行命令完成后，查看下容器是否启动：

![检查mongoDB容器是否启动成功](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a96feb4b69?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)检查mongoDB容器是否启动成功

### 4.3 添加管理员账号

执行命令：

```
docker exec -it mongo mongo admin
```

然后，创建一个拥有最高权限 root 账号:

```
db.createUser({ user: 'admin', pwd: '123456', roles: [ { role: "root", db: "admin" } ] });
```

创建成功后，你会看到 `Successfully added user`:

![创建root账号成功](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a988f60502?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)创建root账号成功

### 4.4 用新创建的 root 账户连接，测试一下

```
docker run -it --rm --link mongo:mongo mongo mongo -u admin -p 123456 --authenticationDatabase admin mongo/admin
```

连接成功后，我们可以执行相关 sql：

**显示所有的数据库：**

```
show dbs
```

**使用某个数据库：**

```
use admin
```

![mongDB显示所有的库](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a9897165a5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)mongDB显示所有的库

输入命令 `exit`，退出连接！

## 五、快速安装&搭建 Elasticsearch 环境

本节中，我们将学习如何通过 Docker 快速安装与搭建 Elasticsearch 环境。

### 5.1 下载 Elasticsearch 镜像

这里以 Elasticsearch 6.5.0 为快速安装&搭建 Elasticsearch 环境例：

```
docker pull elasticsearch:6.5.0
```

下载完成后，通过 `docker images` 检查一下镜像是否下载成功:

![查看es镜像是否下载成功](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a99d9ec1e0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)查看es镜像是否下载成功

### 5.2 先简单运行 Elasticsearch 镜像

下载成功后，简单运行 Elasticsearch 镜像：

```
docker run -d \
--name es \
-p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms200m -Xmx200m" \
elasticsearch:6.5.0
```

- `-d`：以后台的方式运行；
- `--name es`：指定容器的名称为 es;
- `-p 9200:9200 -p 9300:9300` 将容器的 9200、9300 端口挂载到宿主机的 9200、9300 端口上；
- `-e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms200m -Xmx200m"`：指定为单节点模式，JVM 内存占用 200m

命令执行完成后，你也可以通过 `docker ps`命令来确认下容器是否启动成功。

![查看es容器是否运行成功](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a9bca5d6eb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)查看es容器是否运行成功

可以看到 es 容器运行成功了，接下来，进入容器中：

```
docker exec -it es /bin/bash
```

安装 **analysis-ik** 中文分词插件：

```
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.5.0/elasticsearch-analysis-ik-6.5.0.zip
```

> PS: es 从 v5.5.1 版本开始支持自带的 es 插件命令来安装，如果你安装的版本不是 6.5.0，需要将命令中的版本号修改一下，具体参考 [github.com/medcl/elast…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fmedcl%2Felasticsearch-analysis-ik)

![analysis-ik安装成功](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a9cc019ef4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)analysis-ik安装成功

安装成功后，退出容器：

```
exit
```

删除刚刚运行的容器：

```
docker rm -f es
```

> PS: 当然了，你也可以通过容器的 ID 来删除。

### 5.3 复制相关文件

```
# 复制 es 配置文件目录到宿主机指定目录，目标目录你可以根据需要，自行修改
docker cp es:/usr/share/elasticsearch/config /usr/local/docker/es
# 复制 es 持久化数据目录到宿主机指定目录
docker cp es:/usr/share/elasticsearch/data /usr/local/docker/es
# 复制 es 插件目录到宿主机指定目录
docker cp es:/usr/share/elasticsearch/plugins /usr/local/docker/es
```

### 5.4 修改 es 相关配置

进入我们刚刚指定的 config 配置目录，修改 `jvm.options` 文件：

```
-Xms300m
-Xmx300m
```

> PS: 因为小哈测试服务器就 2G 内存，这里我改成了 JVM 内存占用 300m, 如果你的内存够用，可不用改。

修改 `elasticsearch.yml`文件, 添加如下配置：

```
node.name: master
http.cors.enabled: true
http.cors.allow-origin: "*"
```

解释一下添加的配置，设置节点为 master 节点，并允许跨域访问，以便后面使用 head 插件图形化界面访问。

### 5.5 运行 Elasticsearch 容器

```
docker run -d \
--name es \
-p 9200:9200 -p 9300:9300 \
-v /usr/local/docker/es/config:/usr/share/elasticsearch/config \
-v /usr/local/docker/es/data:/usr/share/elasticsearch/data \
-v /usr/local/docker/es/plugins:/usr/share/elasticsearch/plugins \
elasticsearch:6.5.0 
```

这次，我们额外添加了相关挂载命令：

- `-v /usr/local/docker/es/config:/usr/share/elasticsearch/config`: 将容器中的 /usr/share/elasticsearch/config 配置目录挂载到宿主机的 /usr/local/docker/es/config 目录下；
- `-v /usr/local/docker/es/data:/usr/share/elasticsearch/data`： 将容器中的 /usr/share/elasticsearch/data 数据目录挂载到宿主机的 /usr/local/docker/es/data 目录下；
- `-v /usr/local/docker/es/plugins:/usr/share/elasticsearch/plugins`：将容器中的 /usr/share/elasticsearch/plugins 插件目录挂载到宿主机的 /usr/local/docker/es/plugins 目录下；

### 5.6 测试一下，瞅瞅 es 是否能够正常访问

测试一下，看 es 是否启动成功：

```
curl http://localhost:9200
```

![查看es是否能够正常访问](https://user-gold-cdn.xitu.io/2019/5/22/16adf4a9e555fcb8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)查看es是否能够正常访问

OK, 到此 es 的单节点环境就搭建好了！

## 六、总结

好了，到这里，小哈就已经把常用的测试环境搭建介绍完毕了。如果你还有啥疑问，不妨后台私信我！哈哈，祝您看完本文有所收获！
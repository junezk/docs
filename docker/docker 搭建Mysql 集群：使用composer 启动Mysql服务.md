# docker 搭建Mysql 集群：使用composer 启动Mysql服务

[docker 搭建Mysql 集群： 主从配置](https://juejin.im/post/5e148a7de51d454112714324)

[前言](https://juejin.im/post/5e1489b55188253a59093a64#h)[搭建基础服务](https://juejin.im/post/5e1489b55188253a59093a64#h-1)[docker-compose.yml 解析](https://juejin.im/post/5e1489b55188253a59093a64#hdockercomposeyml)[Dockerfile 解析](https://juejin.im/post/5e1489b55188253a59093a64#hdockerfile)[启动Mysql服务](https://juejin.im/post/5e1489b55188253a59093a64#hmysql)[结语](https://juejin.im/post/5e1489b55188253a59093a64#h-2)

### 前言

docker，docker-composer 的安装就不多说明了，请自行安装。
这一系列文章，通过搭建Mysql 服务集群，让大家学会以下知识点：

- docker composer 的使用
- Mysql 的一些配置项



### 搭建基础服务

新建文件夹mysql_composer

```shell
mkdir mysql_composer
cd mysql_composer
```

新建docker-composer.yml

```bash
version: '3'

networks:  #定义网路
  backend:
    driver: ${NETWORKS_DRIVER}

services:
  ### MySQL ################################################
  mysql0:
    build:
      context: ./mysql  #自定义镜像
      args:
        - MYSQL_VERSION=${MYSQL_VERSION} #指定版本
    environment:
      - MYSQL_DATABASE=${MYSQL_DATABASE}  #默认创建数据库
      - MYSQL_USER=${MYSQL_USER}  #创建用户
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}  #用户密码
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} #root 用户秘密
      - TZ=${WORKSPACE_TIMEZONE}
    volumes:
      - ${DATA_PATH_HOST}/mysql:/var/lib/mysql
      - ${MYSQL_ENTRYPOINT_INITDB}:/docker-entrypoint-initdb.d
    ports:
      - "${MYSQL_PORT}:3306"
    networks:
      - backend
```

新建.env

```bash
# All Networks driver
NETWORKS_DRIVER=bridge

WORKSPACE_TIMEZONE=UTC

DATA_PATH_HOST=~/.my_env/data
### MYSQL #################################################

MYSQL_VERSION=5.7
MYSQL_DATABASE=test
MYSQL_USER=test
MYSQL_PASSWORD=test123456
MYSQL_PORT=3308
MYSQL_ROOT_PASSWORD=root
MYSQL_ENTRYPOINT_INITDB=./mysql/docker-entrypoint-initdb.d
```

在当前路径下新建文件夹mysql

```bash
mkdir mysql
```

新建Dockfile文件

```dockerfile
ARG MYSQL_VERSION=latest
FROM mysql:${MYSQL_VERSION}

#####################################
# Set Timezone
#####################################

ARG TZ=UTC
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && chown -R mysql:root /var/lib/mysql/

COPY my.cnf /etc/mysql/conf.d/my.cnf

CMD ["mysqld"]

EXPOSE 3306
```

新建Mysql的配置文件my.cnf

```bash
# The MySQL  Client configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysql]

[mysqld]
sql-mode="STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION"
character-set-server=utf8
```

新建docker-entrypoint-initdb.d目录

```bash
mkdir docker-entrypoint-initdb.d
```

新建文件createdb.sql

```bash
FLUSH PRIVILEGES ;
```

### docker-compose.yml 解析

- 变量：文件中定义了`${NETWORKS_DRIVER}，${MYSQL_VERSION}，${MYSQL_DATABASE}`等变量。这些变量能够在我们定义服务时，提供方便的管理。而变量对应的值我们放在`.env`文件中。默认情况下，docker-compose.yml 会自动读取相同目录下的.env 文件。
- networks 用来定义网络。
- 上面我们定义了一个名为backend（这个名字可以随自己喜欢取）的网络，这个网络的驱动是`bridge`模式。
- services 用来定义要启动的服务。
- 这里我们先只启动一个名为`mysql0`的服务。为了启动服务，我们当然还要配置些参数。
- build 参数用来自定义镜像
  - context: 指定Dockerfile 所在目录
- environment: environment 和 Dockerfile 中的 ENV 指令一样会把变量一直保存在镜像、容器中，类似 docker run -e 的效果。
  - 这个例子中我们配置`MYSQL_DATABASE`,`MYSQL_USER`等变量
- volumes: 挂在目录
  - 例子中我们挂在了2个目录/var/lib/mysql 挂载到${DATA_PATH_HOST}/mysql 中，这样mysql 存储的数据就不会随着容器被删除而消失了。
  - /docker-entrypoint-initdb.d 是用来做些初始化db的工作的，比如你希望创建容器后自动创建一些数据库和表，都可以在这个目录下创建对应sql文件。
- ports: 映射端口
  - "${MYSQL_PORT}:3306" 例子中我们把3306 端口映射到了`${MYSQL_PORT}`端口。
- networks： 指定要使用的网络
  - 这里我们指定了使用`backend`网络。当然目前没有什么用，毕竟只有一个服务。

### Dockerfile 解析

dockerfile 使用来自定义镜像的，可以让我们方便的构建自己的docker 镜像
这个例子中的做的事情其实很简单
设置服务的时区，并且把自定义的配置文件，放入到mysql 的配置文件目录下。

### 启动Mysql服务

```bash
docker-compose up -d
```

你可以使用账户test,密码test123456链接mysql服务，也可以使用账户root,密码root连接mysql服务。由于我本地已经有个Mysql 服务占用了3306端口，所有配置文件中，我给改成了3308。

### 结语

通过本篇文章，相信大家都会快速搭建一个Mysql 服务了。当然我们的目标是打一个集群出来。下一篇我们讲解怎么搭建另一台Mysql 服务，并使两台服务连通。

[docker 搭建Mysql 集群： 主从配置](https://juejin.im/post/5e148a7de51d454112714324)
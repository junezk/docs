# 使用docker部署django技术栈项目

> 本文最早发布DeanWu的技术博客：[pylixm.cc/posts/2019-…](https://link.juejin.im/?target=https%3A%2F%2Fpylixm.cc%2Fposts%2F2019-08-30-docker-django.html)

随着Docker的普及成熟，已经逐渐成为部署项目的首选，今天来和大家分享下如何使用docker部署django技术栈项目。

我们这里说的Django技术栈为：python3.6、Django2.2、redis、mysql、celery、gunicorn和nginx。在实际的生产项目中，这些组件分布在集群的不同机器，如Nginx、redis和Mysql可能会有单独的团队或部门负责。涉及的部署架构和容器的编排会更为复杂，本文暂不去深究。本文主要介绍，如何使用`docker-compose`来编排这些组件，这种方式适用于测试环境的部署或者你的个人`sideproject`的部署。

本文默认你已经了解`docker`和`docker-compose`的一些基本知识，若你不了解，可阅读下面这些资料：

- [Docker知识大全](https://link.juejin.im/?target=https%3A%2F%2Fpylixm.cc%2Fposts%2F2018-05-31-Docker-all.html)
- [Docker官方文档](https://link.juejin.im/?target=https%3A%2F%2Fdocs.docker.com%2F)
- [Docker Compose文档](https://link.juejin.im/?target=https%3A%2F%2Fdocs.docker.com%2Fcompose%2F)

下面我们来说下如何部署。

## 项目组织结构

首先，看下我们的项目组织结构，结构如下：

```
├── LICENSE
├── README.md
├── compose
│   ├── celery
│   │   ├── Dockerfile
│   │   ├── celery-beat.sh
│   │   └── celery.sh
│   ├── mysql
│   │   └── my.cnf
│   ├── nginx
│   │   └── nginx.conf
│   └── web
│       ├── Dockerfile
│       ├── entrypoint.sh
│       ├── gunicorn.conf
│       └── gunicorn.sh
├── docker-compose.yml
├── docker_django_demo
│   ├── __init__.py
│   ├── celery.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── env.tpl
├── manage.py
├── requirements.txt
```

除了Django的项目文件外，主要增加了`compose` 配置文件目录和`docker-compose.yml` 配置文件。

- `compose`目录主要存放各组件的dockerfile文件和启动脚本。
- `docker-compose.yml` 是docker-compose的编排配置文件。

## 编写Dockerfile 及启动初始化脚本

在docker-compose中，容器的启动有两种方法，一种是直接使用公共的镜像来启动容器，另一种是通过我们自己编写的Dockerfile。因为我们要安装额外的工具包和初始化相关配置，web和celery组件我们使用自定义的Dockerfile方式。

web容器的`compose/web/Dockerfile`:

```
FROM python:3.6
ENV PYTHONUNBUFFERED 1

RUN mkdir /code
WORKDIR /code

COPY ./requirements.txt /code/
RUN pip install --no-cache-dir -r requirements.txt \
    && rm -rf requirements.txt

COPY . /code/
COPY ./compose/web/*.sh /code/
RUN sed -i 's/\r//' gunicorn.sh \
    && chmod +x gunicorn.sh \
    && sed -i 's/\r//' entrypoint.sh \
    && chmod +x entrypoint.sh

ENTRYPOINT ["/bin/bash", "entrypoint.sh"]
```

web容器的其他文件：

- `compose/web/entrypoint.sh` web容器的启动脚本，执行一些初始化或检测逻辑。
- `compose/web/gunicorn.conf` gunicorn配置文件。
- `compose/web/gunicorn.sh` gunicorn的启动脚本。

celery的Dockerfile:

```
FROM python:3.6
ENV PYTHONUNBUFFERED 1

RUN mkdir /code
WORKDIR /code

COPY ./requirements.txt /code/
COPY ./compose/celery/*.sh /code/
RUN pip install --no-cache-dir -r requirements.txt \
    && rm -rf requirements.txt && sh init_env.sh

COPY . /code/
COPY ./compose/celery/*.sh /code/
RUN sed -i 's/\r//' celery.sh \
    && chmod +x celery.sh \
    && sed -i 's/\r//' celery-beat.sh \
    && chmod +x celery-beat.sh
```

celery的其他文件：

- `compose/celery/celery.sh` celery的启动脚本。
- `compose/celery/celery-beat.sh` celery-beat的启动脚本。

## 编写Compose启动配置文件

docker-compose 配置如下：

```
version: '2'

services:
  redis:
    image: redis
    ports:
      - "6379:6379"

  db:
    restart: always
    image: mysql:5.7.19
    # command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    volumes:
      - ./compose/mysql/:/etc/mysql/conf.d
      - ./db:/var/lib/mysql
    # for test
    ports:
      - "127.0.0.1:3307:3306"
      # (HOST:CONTAINER)
    env_file:
      - .env

  web:
    # restart: always
    build:
      context: .
      dockerfile: ./compose/web/Dockerfile
    command: sh gunicorn.sh # ["/bin/bash", "gunicorn.sh"]
    ports:
      - "8080:8002"
      # (HOST:CONTAINER)
    volumes:
      - ./logs:/var/logs/
      - ./collect_static:/code/collect_static
      - ./static:/code/static
      - ./templates:/code/templates
      - ./uploads:/code/uploads
    env_file: .env
    depends_on:
      - redis
      - db

  nginx:
    restart: always
    image: nginx:1.13.0
    volumes:
      - ./compose/nginx:/etc/nginx/conf.d/
      - ./staticfiles:/code/staticfiles
      - ./logs:/var/log/nginx
    ports:
      - "80:80"
      # (HOST:CONTAINER)
    depends_on:
      - web

  celery:
    build:
      context: .
      dockerfile: ./compose/celery/Dockerfile
    command: sh celery.sh
    volumes:
      - ./logs:/var/logs/
      - ./uploads:/code/uploads
    depends_on:
      - redis
      - db
    env_file: .env

  celery-beat:
    build:
      context: .
      dockerfile: ./compose/celery/Dockerfile
    command: sh celery-beat.sh
    volumes:
      - ./logs:/var/logs/
    depends_on:
      - redis
      - db
    env_file: .env
```

celery 的worker 和beat这里我们使用同一个镜像Dockerfile, 按照一个镜像一个进程的原则，启动两个容器来分别跑worker和beat进程。

## 编译测试

编写好配置文件之后，编译镜像测试运行：

```
docker-compose build 
docker-compose up  # 前台运行 
docker-compose up -d  # 无误后可后台运行
```

`docker-compose ps` 可以看到启动好的容器：

```
$ docker-compose ps    
             Name                           Command               State            Ports          
--------------------------------------------------------------------------------------------------
dockerdjangodemo_celery-beat_1   sh celery-beat.sh                Up                              
dockerdjangodemo_celery_1        sh celery.sh                     Up                              
dockerdjangodemo_db_1            docker-entrypoint.sh mysqld      Up      127.0.0.1:3307->3306/tcp
dockerdjangodemo_nginx_1         nginx -g daemon off;             Up      0.0.0.0:80->80/tcp      
dockerdjangodemo_redis_1         docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp  
dockerdjangodemo_web_1           /bin/bash entrypoint.sh sh ...   Up      0.0.0.0:8080->8002/tcp
```

映射端口可根据自己的实际情况调整。

## 问题

下面说下在构建过程中的几个需要注意的问题。

**mysql 编码问题**

docker 提供的mysql镜像，默认编码为`latin1`, 在保存中文时会显示乱码。官方提供了一种修改编码方式的方法，在启动脚本后指定编码格式，文档可见[这里](https://link.juejin.im/?target=https%3A%2F%2Fhub.docker.com%2F_%2Fmysql)。mysql容器5.7.19版本可直接在docker-compose.yml 中的command后跟上参数`--character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci`。这种方式，只是修改server端的编码。可直接使用配置文件覆盖的方式，指定所有的编码格式。

配置如下：

```
[mysqld]
default-storage-engine=INNODB
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
init-connect='SET NAMES utf8mb4'
init_connect='SET collation_connection = utf8mb4_general_ci'
skip-character-set-client-handshake  # 跳过客户端的编码配置，客户端直接使用服务端的编码配置
bind-address = 0.0.0.0
```

注：mysql 5.7.19 配置文件方式成功，5.7.4、5.7.17 均失败，可做参考。

**web等mysql启动完成后再继续**

mysql 容器在启动起来之前是无法接受数据库链接的，在web启动初始化时，若数据库还没有启动好会导致web容器启动失败直接退出。我们可以增加在web容器启动时增加检测脚本，数据库连通之后，再继续。

脚本如下：

```
#!/usr/bin/env bash
set -o errexit
set -o pipefail

echo $MYSQL_PASSWORD
echo $MYSQL_DATABASE
echo $MYSQL_HOST
echo $MYSQL_USER
echo $MYSQL_PORT

function mysql_ready(){
python << END
import sys
import pymysql
try:
    conn = pymysql.connect(host="db", port=3306, user="root", passwd="$MYSQL_ROOT_PASSWORD", db='$MYSQL_DATABASE', charset='utf8')
except pymysql.err.OperationalError:
    sys.exit(-1)
sys.exit(0)
END
}

until mysql_ready; do
  >&2 echo "MySQL is unavailable - sleeping"
  sleep 1
done

>&2 echo "MySQL is up - continuing..."
```

## 总结

到此，使用docker来部署django技术栈服务就完成了，完整的项目代码，大家可参阅 [docker-django-demo](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fpylixm%2Fdocker-django-demo%2Ftree%2Fdocker-compose-production3)。

文章开始说了，该部署方式不适合大型网站的线上生产服务，耦合度太高，不好维护等存在着许多问题。但是，部署自己的sideproject或者测试环境，在硬件资源有限的情况的下还是非常不错的。除了减少环境部署搭建的麻烦外，迁移起来也是很方便的。

[demo](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fpylixm%2Fdocker-django-demo%2Ftree%2Fdockerfile-for-mac)项目中也有些开发环境下如何使用docker的案例，但是个人一直认为docker更适合部署，在开发环境方便不如直接搭建来的灵活方便。欢迎大家留言，共同讨论docker在开发和部署上的使用心得。

## 参考

- [cookiecutter-django](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fpydanny%2Fcookiecutter-django)
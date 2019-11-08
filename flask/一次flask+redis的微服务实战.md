# 一次flask+redis的微服务实战

# 0x00 背景

先说一下需求的场景，策划部门想做一个垂直领域的社区产品，类似与知乎那种，但受益人群是金融行业的从业人员。产品中有一个搜索问题的需求，搜索的问题去题库中进行模糊匹配，终端支持H5/APP/小程序，甚至是微信的自动回复，用过ItChat的人都知道这个强大的工具包，不过最近好像有封号的动作了。做搜索，肯定就会用到分词，当前开源的分词库jieba是做的比较好的了，分析了一下技术栈，就决定用flask来实现一个微服务，供其他的服务模块使用。

本人会着重讲一下环境的搭建，各种配置的坑。

# 0x01 升级操作系统

操作系统：CentOS 6.x
CentOS 7的玩家手动跳过该环节。首先6.x的版本，默认装的python的版本是2.6的版本，这个版本后面用到的一些软件支持的不是很好，所以我们首先要把python升级到2.7的版本。先创建一个update.sh脚本，然后把下面内容复制进去。

```shell
#!/usr/bin/env bash
#安装依赖
yum install openssl openssl-devel zlib-devel gcc -y
# apt-get install libssl-dev
# apt-get install openssl openssl-devel
# 下载源码
wget http://www.python.org/ftp/python/2.7.12/Python-2.7.12.tgz
tar -zxvf Python-2.7.12.tgz
cd Python-2.7.12
mkdir /usr/local/python2.7.12
# 开启zlib编译选项
# sed -i '467c zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz' Module/Setup
sed '467s/^#//g' Module/Setup

./configure --prefix=/usr/local/python2.7.12
make
make install
if [ $? -eq 0 ];then
     echo "Python2.7.12升级完成"
else
     echo "Python2.7.12升级失败，查看报错信息手动安装"
fi
cd
mv /usr/bin/python /usr/bin/python2.6.6
ln -s /usr/local/python2.7.12/bin/python2.7 /usr/bin/python

sed -i '1s/python/python2.6/g' /usr/bin/yum
wget --no-check-certificate https://bootstrap.pypa.io/get-pip.py
python get-pip.py
if [ $? -eq 0 ];then
     echo "pip升级完成"
else
     echo "pip安装失败，查看报错信息手动安装"
fi
rm -rf /usr/bin/pip
ln -s /usr/local/python2.7.12/bin/pip2.7 /usr/bin/pip
```

然后把这个脚本加上可执行的权限，并且运行。

```shell
chmod u+x update.sh
./update.sh
```

# 0x02 创建python虚拟空间

这一步完成之后，我们就开始隔离一个虚拟的空间来运行这个微服务，以后我们所有的程序都在隔离的虚拟空间中来运行，有点docker的概念，这样不管是用python2.x还是3.x都可以同时兼容了。

```python
pip install virtualenv
cd /home/test/web
virtualenv ENV
cd bin
source active
```

至此就启动了这个docker，接下来就是在这个docker中安装我们requirement.txt中所需要的包了。
我们用的包有如下：

```python
pip install flask
pip install jieba
pip install pymysql
pip install redis
pip install virtualenv
pip install uwsgi
pip install xlrd
```

到此为止就可以在ENV的环境中，测试app的功能了，现在python myapp.py试一下各接口功能。

# 0x03 安装nginx

尽管测试成功了，但是线上是不能这么运行的，因为flask只是一个web框架，并不是一个web server的容器，flask自带的werkzeug只能用于开发环境，不能用于生产环境，对于web服务器，我们选择更专业的uWSGI，并且配合nginx作为反向代理。
首先，我们安装nginx

```shell
yum -y install nginx
```

默认的nginx的监听端口是80，如果部署在云服务器上，80端口极大的可能被占用了，我们要去修改这个配置文件。

```shell
cd /etc/nginx
vim conf.d
```

查看一下配置端口的配置文件在哪里，我们看到是default.conf这个文件中有端口号的配置，我们把所有端口号都改成我们想要的端口比如8100。

在 /etc/nginx/ngixn.conf 文件的 http 部分添加一条 include 内容，即最后一行

```conf
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include /etc/nginx/sites-enabled/*;
}
```

创建配置文件 vim /etc/nginx/sites-enabled/hello.conf

```conf
server {
    # Running port
    listen 8100;
    # 服务器ip 或者域名
    server_name 0.0.0.0;

    # Proxying connections to application servers
    location / {
        include            uwsgi_params;
        uwsgi_pass         127.0.0.1:5000;//和 uWSGI 配置文件中的 ip端口一致
    }
}
```

然后看系统中是否有nginx的service脚本

```shell
vim /etc/init.d/nginx
```

如果没有该脚本，在该脚本中写如下内容：

```shell
#!/bin/bash
# chkconfig: - 85 15
nginx=/usr/sbin/nginx
conf=/etc/nginx/nginx.conf

case $1 in
    start)
        echo -n "Starting Nginx"
        $nginx -c $conf
        echo " done"
    ;;

    stop)
        echo -n "Stopping Nginx"
        $nginx -s stop
        echo " done"
    ;;

    test)
        $nginx -t -c $conf
    ;;

    reload)
        echo -n "Reloading Nginx"
        $nginx -s reload
        echo " done"
    ;;

    restart)
        $0 stop
        $0 start
    ;;

    show)
        ps -aux|grep nginx
    ;;

    *)
        echo -n "Usage: $0 {start|restart|reload|stop|test|show}"
    ;;

esac
```

给该文件加上可执行权限

```shell
chmod u+x nginx
service nginx start
```

Nginx 在 /var/log/nginx 目录下有 access.log 和 error.log 两个日志文件，如果有问题可以查看这2个日志文件。

# 0x04 配置uWSGI

接下来配置uWSGI的启动文件，创建 uwsgi 配置文件

```shell
vim /etc/uwsgi/apps-enabled/hello.ini
[uwsgi]
// 开启主线程
master = true
// 项目目录
base = /home/test/web
// 移动到项目目录 cd
chdir = %(base)
// 本地的ip和端口
socket = 127.0.0.1:5000
// Python 虚拟环境目录
home = %(base)/ENV
// 程序启动文件
wsgi-file = hello.py
// 项目中引用 flask 实例的变量名
callable = app
// 处理器数
processes = 2
// 线程数
threads = 4
// 获取uwsgi统计信息的服务地址
stats = 127.0.0.1:9191
```

保存配置文件，通过 uwsgi -i /etc/uwsgi/apps-enabled/hello.ini，来启动 uwsgi。
注意在env的环境下，uwsgi安装的目录不会在/usr/bin中，而是在ENV的bin中，所以
在后面的/etc/init.d/uwsgi中的路径要注意。
创建uWSGI的日志文件

```shell
vim /var/log/uwsgi/hello.log
```

创建uWSGI的service文件

```shell
vim /etc/init.d/uwsgi
#!/bin/bash
# chkconfig: - 85 15
uwsgi=/home/test/web/ENV/bin/uwsgi
hello_conf=/etc/uwsgi/apps-enabled/hello.ini

case $1 in
    start)
        echo -n "Starting uWsgi"
        nohup $uwsgi -i $hello_conf >/var/log/uwsgi/hello.log 2>&1 &
        echo " done"
    ;;

    stop)
        echo -n "Stopping uWsgi"
        killall -9 uwsgi
        echo " done"
    ;;

    restart)
        $0 stop
        $0 start
    ;;

    show)
        ps -ef|grep uwsgi
    ;;

    *)
        echo -n "Usage: $0 {start|restart|stop|show}"
    ;;

esac
```

添加可执行属性

```shell
sudo chmod +x /etc/init.d/uwsgi
service uwsgi start
```

uWSGI 在 /var/log/uwsgi 目录下有项目对应日志，有问题可以查看。

# 0x05 redis的安装

首先安装redis的编译工具gcc和gcc-c++

```shell
yum install gcc gcc-c++
```

然后从redis的官网上下载redis并且安装

```shell
wget http://download.redis.io/releases/redis-3.2.5.tar.gz
tar -zxvf /soft/redis-3.2.5.tar.gz
cd redis-3.2.5
make & make install
```

然后配置redis.conf

```conf
bind 0.0.0.0
# 3.x版本，将此属性关闭
protected-mode no
```

添加redis的service服务

```shell
vim /etc/init.d/redis
#!/bin/sh
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

然后启动服务

```shell
chmod u+x redis
service redis start
```

# 0x06 python中引入redis作为缓存

我们知道redis是k-v的数据缓存机制，当缓存数据为string, 或是hash类型的时候，都有原生的函数支持，具体可以参考redis的API文档。
但是当我们缓存我们自定义的实体类的时候，有点力不从心，这时候可以用pickle库来做序列化，先把实体类转换成string类型存储在redis中，取出的时候，再做反序列化的操作，具体代码如下：

```python
#coding=utf-8

import redis
import pickle

class Redis:
    @staticmethod
    def connect(host='localhost', port=6379, db=0):
        r = redis.StrictRedis(host, port, db)
        return r

    # 将内存数据二进制通过序列号转为文本流，再存入redis
    @staticmethod
    def set_data(r, key, data, ex=None):
        r.set(pickle.dumps(key), pickle.dumps(data), ex)

    # 将文本流从redis中读取并反序列化，返回
    @staticmethod
    def get_data(r, key):
        data = r.get(pickle.dumps(key))
        if data is None:
            return None

        return pickle.loads(data)
```

# 0x07 总结

到此为止，一个flask的app的部署就基本完成了，中间的环节比较琐碎，但互联网上的资料也很多，基本都可以解决，祝大家玩的开心！
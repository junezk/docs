# 部署Django REST Framework服务(Nginx + uWSGI + Django)

### 初衷

在探索了DRF之后（两篇：[Django REST Framework Quickstart 项目解读 - 简书](https://www.jianshu.com/p/6c9204bda63f) 和 [Django REST Framework 实现业务 api 并自动文档化 - 简书](https://www.jianshu.com/p/4e7d1b04caf2) ） ，是时候上线到一个正式的服务上了，也为了后面更好地实现一些想法。

### 准备服务器

一开始想到的还是阿里云主机，之前也用过一段时间，但是说实话，价格贵了点。因为现在只是实现一些自己的小想法，所以选择了性价比稍微高一些的国外vps。根据大家的推荐，选择了vultr的5刀版本，已经足够使用了，支持支付宝付款，所以特别方便。

但是这个过程还是有些问题，比如说节点的IP可能会被墙，解决的方法是，不断地试着换实例，一般能换到访问正常的节点。但是这种墙法还是比较暴力的，使用了tcp截断，你会发现能ping通，但是不能建立tcp连接。

VPS启动ok之后，建立一个新用户，避免总是使用root账号去登录或者去操作，并在新用户上加入sshkey，方便今后ssh连接。之后就可以绑定域名了，使用vps的另一个好处是，不需要备案。

### 安装nginx

直接安装便可，我看版本也不低。

```
apt-get install nginx
```

主配置文件：/etc/nginx/nginx.conf

启动nginx服务：

```
sudo service nginx start
```

默认情况下，使用域名测试访问nginx是否启动正常便可。

### 安装python3+virtualenv环境

因为我选择了ubuntu18.04的系统，好像默认就预装了python3.6，所以这个可以跳过了。

安装virtualenv：

```
sudo apt-get install python3-pip
pip3 install virtualenv
```

高版本的ubuntu可能会把virtualenv安装到$HOME/.local/bin/下面，所以需要把路径添加到PATH里。

### 建立django运行的虚拟环境并安装django等

```
virtualenv -p /usr/bin/python3 env3
pip install djangorestframework django pymysql coreapi pygments markdown
git clone https://github.com/roubo/rouboApi.git
```

### 安装mysql

```
sudo apt-get install mysql-server
## 查看默认用户名密码
sudo cat /etc/mysql/debian.cnf 
## 登录
mysql -u xxxx -p yyyy
```

修改密码：

```
mysql> use mysql;
Database changed

mysql> update user set plugin='mysql_native_password' where user='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> update user set authentication_string=password('xxx') where user='root';
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

### 建立业务相关的数据库

```
create database rouboinfo default charset utf8 collate utf8_general_ci;
```

### django项目数据库初始化

```
python manage.py makemigrations
python manage.py migrate
```

### 测试Django端服务

启动django调试服务

```
python manage.py runserver
```

测试接口，本地访问ok（django服务本身不需要对外网提供访问服务，最终通过nginx分发）

```
➜  ~ curl -i 'http://127.0.0.1:8000/rouboapi/v1/report/'
HTTP/1.1 400 Bad Request
Date: Thu, 06 Sep 2018 03:42:23 GMT
Server: WSGIServer/0.2 CPython/3.6.5
Content-Type: application/json
Vary: Accept
Allow: GET, HEAD, OPTIONS
X-Frame-Options: SAMEORIGIN
Content-Length: 124

{"report_type":["This field is required."],"device_id":["This field is required."],"ip_address":["This field is required."]}%
```

### 桥接nginx和django

现在nginx和django的服务都已经ok，接下来就是怎么打通两者，我们之前提到过uWSGI。这里我们来了解下WSGI协议的事情。

#### WSGI协议

WSGI（Web Server GateWay Interface），这个是一套协议，用于连接web server 和 web application。他的出现也是人们对Web的server和application解耦的需求。举个例子，比如我们要完成两个api接口，我们可以很作地选择两个web application框架，比如Django和Flask，一个api分别使用一种框架，而对客户端来说，可以完全无感。WSGI协议在这里就起到了桥接作用，可以实现多Server对单application，或者当server对多application。

#### uWSGI应用

uWSGI是一个应用，他可做为两种角色：

- 当在 Nginx + uWSGI + Django 场景下，他是中间件。
- 当在 uWSGI + Django 的场景下， 他是Web Server。

当为中间件角色时，需要用到其uwsgi协议，Nginx一般通过该协议与uWSGI连接。这里的uwsgi协议的作用，我们可以简单地理解为，如何高效地传递request和response便好。而uWSGI中间件的作用，我们也可以简单地理解为，作为一个网关屏蔽各web application之间的协议、开发语言等差异，并且高效地管理着各种并发、负载均衡、日志等。

那为什么要选择 Nginx + uWSGI + Django 这种方式呢? 其实是一个物尽其用的理由。nginx有如下优点：

> 1、安全，客户端对Web服务器的访问需要先经过反向代理服务器。这样可以防止外部程序对Web服务器的直接攻击。
>  2、负载均衡，反向代理服务器可以根据Web服务器的负载情况，动态地把HTTP请求交给不同的Web服务器来处理，前提是要有多个Web服务器。
>  3、提升Web服务器的IO性能。一个HTTP请求的数据，从客户端传输给服务器，是需要时间的，例如N秒，如果直接传给Web服务器，Web服务器就需要让一个进程阻塞N秒，来接收IO，这样会降低Web服务器的性能。如果使用反向代理服务器，先让反向代理服务器接收完整个HTTP请求，再把请求发给Web服务器，就能提升Web服务器的性能。还有一些静态文件的请求，可以直接交给反向代理来处理，不需要经过Web服务器。

#### 安装和配置uWSGI和Nginx

注意我还是将应用安装在虚拟环境中。

首先在非虚拟环境中安装全局依赖的包：

```
sudo apt-get install python3.6-dev build-essential
```

在虚拟环境中安装uWSGI

```
pip install uwsgi
```

测试uWSGI + django是否运行正常，在项目路径下运行并测试8080端口是否正常：

```
uwsgi --http :8080 --module rouboinfo.wsgi
```

如果这一切正常，那就可以开始配置nginx接入uWSGI了。

增加nginx配置，rouboapi.conf:

```
# the upstream component nginx needs to connect to
upstream django {
    server unix:///data/django/rouboApi/rouboapi.scok; # for a file socket
    #server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      80;
    # the domain name it will serve for
    server_name app.airoubo.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    #location /media  {
    #    alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
    #}

    #location /static {
    #    alias /path/to/your/mysite/static; # your Django project's static files - amend as required
    #}

    # Finally, send all non-media requests to the Django server.
    location /roubo {
        uwsgi_pass  django;
        include     /etc/nginx/uwsgi_params; # the uwsgi_params file you installed
    }
}
```

使用配置文件运行uWSGI：

```
# mysite_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = /data/django/rouboApi
# Django's wsgi file
module          = rouboinfo.wsgi
# the virtualenv (full path)
home            = /data/django/env3

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 10
# the socket (use the full path to be safe
socket          = /data/django/rouboApi/rouboapi.scok
# ... with appropriate permissions - may be needed
chmod-socket    = 666
# clear environment on exit
vacuum          = true
# 在虚拟环境下执行
uwsgi --ini rouboapi.ini
```

至此，我们发现接口访问已经正常，nginx <-> uWSGI <-> django 的通路已经ok了。但是我们来看下Django REST Framework 的文档页面的效果：

![效果图](https:////upload-images.jianshu.io/upload_images/1354011-202cddc51eeb4e57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

发现所有样式都丢失了。。这是因为我们的nginx配置里并没有将静态文件暴露给客户端：

```
    #location /static {
    #    alias /path/to/your/mysite/static; # your Django project's static files - amend as required
    #}
```

### Django的静态文件收集和Nginx的静态文件配置

上面说的文档页面的样式丢失问题，我们可以看下nginx的error log：

```
2018/09/06 07:41:27 [error] 14754#14754: *23 open() "/usr/share/nginx/html/static/rest_framework/js/default.js" failed (2: No such file or directory), client: 116.231.148.176, server: app.airoubo.com, request: "GET /static/rest_framework/js/default.js HTTP/1.1", host: "app.airoubo.com", referrer: "http://app.airoubo.com/roubo/rouboapi/v1/report/"
```

也就是rest下的js文件是404。

#### 收集Django的静态文件到指定目录

Settings.py中加入收集路径：

```
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

在项目下执行：

```
python manage.py collectstatic
```

执行后在项目目录下可以看到static文件夹：

```
static
|-- admin
|   |-- css
|   |-- fonts
|   |-- img
|   `-- js
`-- rest_framework
    |-- css
    |-- docs
    |-- fonts
    |-- img
    `-- js
```

配置nginx，放开静态文件，修改nginx配置文件中的static如下：

```
location /static {
        alias /data/django/rouboApi/static; 
}
```

重启nginx，查看效果，完工：

![查看效果](https:////upload-images.jianshu.io/upload_images/1354011-c1e003ec2aba4fab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

### 完成

后端部署完成，后面可以愉快地添加新接口新服务了。

作者：萝卜日志

链接：https://www.jianshu.com/p/d50fbb7cd5df

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
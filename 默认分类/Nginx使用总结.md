# Nginx使用总结

Nginx安装
-------

### 安装

    tar zxvf nginx-1.2.9.tar.gz   #解压nginx
    cd nginx-1.2.9   #进入目录
    ./configure --prefix=/opt/soft/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module   #配置安装模块
    make install   #安装
    复制代码

`--prefix`：指定安装目录，默认的安装目录是/usr/local/nginx；

`--with-http_ssl_module`：安装https服务模块

### 启动

    /opt/soft/nginx/sbin/nginx
    /opt/soft/nginx/sbin/nginx -s stop   # fast shutdown
    /opt/soft/nginx/sbin/nginx -s quit   # graceful shutdown
    /opt/soft/nginx/sbin/nginx -s reload   # reloading the configuration file
    /opt/soft/nginx/sbin/nginx -s reopen   # reopening the log files
    复制代码

设置开机自启动：

    echo "/opt/soft/nginx/sbin/nginx -c /opt/soft/nginx/conf/nginx.conf" >> /etc/rc.local
    复制代码

Nignx配置虚拟主机、反向代理、负载均衡
---------------------

### 虚拟主机

_主要配置server模块的 listen 和 server_name_

#### 基于域名

    server {
    	listen 80;
    	server_name test.a.com;
    	location / {
    		proxy_pass http://192.168.0.1;   #反向代理到其他站点
    	}
    }
    server {
    	listen 80;
    	server_name test.b.com;
    	location / {
    		proxy_pass http://192.168.0.2;   #反向代理到其他站点
    	}
    }
    复制代码

**注意：** 配置文件下载服务器

    server {
    	listen       80;
    	server_name  file.download.com;
    	charset utf-8;
    	location ~ ^/(.*)$ {
    		add_header Content-Disposition "attachment; filename=$1";   #设置header
    		alias "C:/Robot_Download/$1";   #文件的本地位置
    	}
    }
    
    复制代码

#### 基于端口

    server {
    	listen 80;
    	server_name localhost;
    	alias /data/html/index.html;   #也可使用root、location等方式指向静态资源
    }
    server {
    	listen 81;
    	server_name localhost;
    	root /data/html/index.html;   #也可使用alias、location等方式指向静态资源
    }
    复制代码

#### 基于ip

    server {
    	listen 100.100.100.100:80;
    	server_name localhost;
    	location / {
    		alias /data/html/index.html;   #也可使用alias、root等方式指向静态资源
    	}
    }
    server {
    	listen 100.100.100.101:80;
    	server_name localhost;
    	location / {
    		alias /data/html/index.html;   #也可使用alias、root等方式指向静态资源
    	}
    }
    复制代码

### 反向代理

_主要配置location模块的 proxy_pass_

    server {
    	listen 80;
    	server_name test.b.com;
    	location / {
    		proxy_pass http://192.168.0.2;   #反向代理到其他应用服务器或web服务器
    	}
    }
    复制代码

### 负载均衡

_主要配置upstream和location模块的proxy_pass_

    upstream tomcat_server_pool{
    	ip_hash;
    	server 127.0.0.1:8090 weight=10;   #设置访问权重，权重越高越容易被访问
    	server 127.0.0.1:8100 weight=10;
    	server 127.0.0.1:8110 weight=7;
    }
    server {
    	listen 80;
    	server_name test.b.com;
    	location / {
    		proxy_pass http://tomcat_server_pool;   #反向代理到其他服务器集合
    	}
    }
    复制代码

\*\*ip\_hash：\*\*使用ip\_hash策略的负载均衡解决session问题。每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可较好地解决session的问题。

location映射规则
------------

### alias和root的区别

    location  /svn/ {
    	root /data/ftp/;
    	autoindex on;
    }
    复制代码

访问`127.0.0.1/svn/a.jpg`：则会进入到 **/data/ftp/svn/a.jpg**

    location  /svn/ {
    	alias /data/ftp/;
    	autoindex on;
    }
    复制代码

访问`127.0.0.1/svn/a.jpg`：则会进入到 **/data/ftp/a.jpg**

### proxy_pass的url是否存在 / 的区别

_注意：alias和root后的url都是要加/的_

    location  /proxy/ {
    	proxy_pass http://127.0.0.1/;
    }
    复制代码

访问：`127.0.0.1/proxy/a.jpg`：则会请求到：[http://127.0.0.1/a.jpg](https://link.juejin.im?target=http%3A%2F%2F127.0.0.1%2Fa.jpg)

    location  /proxy/ {
    	proxy_pass http://127.0.0.1;
    }
    复制代码

访问：`127.0.0.1/proxy/a.jpg`：则会请求到：[http://127.0.0.1/proxy/a.jpg](https://link.juejin.im?target=http%3A%2F%2F127.0.0.1%2Fproxy%2Fa.jpg)

### “location /xxx/” 与“location ^~ /xxx/”区别

    location = / {   #表示匹配访问根目录
    	root   html;   #当前安装目录下的html，/html则表示服务器根目录下的html
    	index  index.html index.htm;
    }
    location /svn/ {   #表示匹配ip:port/svn/
    	root /data/;
    	autoindex on;
    }
    }
    location ^~ /svn/ {   #表示只要含有svn/就会被匹配
    	root /data/;
    	autoindex on;
    }
    复制代码

“location /xxx/”表示匹配`ip:port/xxx`，需注意：

> 1.  能匹配到 [test.com/](https://link.juejin.im?target=http%3A%2F%2Ftest.com%2F)**xxx**/home.jpg；
> 2.  不能匹配到 [test.com/folder/](https://link.juejin.im?target=http%3A%2F%2Ftest.com%2Ffolder%2F)**xxx**/home.jpg；
> 3.  如果需要匹配到后者，应改为：`location /folder/xxx/`

**科普：**

一般默认都有`location = /（精确匹配）`，而还有一种是 `location /（模糊匹配）`。

两者的区别是：模糊匹配就算匹配到也会一直匹配下去，而精确匹配不会。

例： 如上例中，把`location = /`换成`location /`，那么请求 `112.74.55.239/svn/`：

1.  先会匹配 `/`，请求的物理路径变成了：**/usr/local/nginx/html**
2.  继续匹配`/svn/`，实际访问的物理路径变成了： **/usr/local/nginx/html/data/svn/**

### rewrite

    location ~ \.php${
    	rewirte "^/php/(.*)$" http://localhost:8090/$1
    }
    复制代码

*   将`localhost/php/test.php`重定向到`localhost:8090/test.php`。如果正则表达式（regex）匹配到了请求的URI（request URI），这个URI会被后面的replacement替换
*   如果正则表达式（regex）里包含“}” or “;”字符，需要用单引号或者双引号把正则表达式引起来
*   如果replacement字符串里有新的request参数，那么之前的参数会附加到其后面，如果要避免这种情况，那就在replacement字符串后面加上“？”，eg： rewrite ^/users/(.*)$ /show?user=$1? last;=

**可选的flag参数如下：**

1.  last

*   结束当前的请求处理，用替换后的URI重新匹配location；
*   可理解为重写（rewrite）后，发起了一个新请求，进入server模块，匹配location；
*   如果重新匹配循环的次数超过10次，nginx会返回500错误；
*   返回302 http状态码 ；
*   浏览器地址栏显示重定向后的url

2.  break

*   结束当前的请求处理，使用当前资源，不在执行location里余下的语句；
*   返回302 http状态码 ；
*   浏览器地址栏显示重定向后的url

3.  redirect

*   临时跳转，返回302 http状态码；
*   浏览器地址栏显示重定向后的url

4.  permanent

*   永久跳转，返回301 http状态码；
*   浏览器地址栏显示重定向后的url

常用配置
----

### 支持后端获取真实客户端IP，而不是该代理的IP

    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header REMOTE-HOST $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    复制代码

### 解决request.getScheme()获取不到真实协议

    proxy_set_header X-Forwarded-Proto $scheme;
    复制代码

### websocket配置

    map $http_upgrade $connection_upgrade {
    default upgrade;
    	'' close;
    }
    复制代码
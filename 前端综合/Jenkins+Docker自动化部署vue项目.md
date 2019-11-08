# [手把手系列之] Jenkins+Docker自动化部署vue项目

## 介绍

本系列主要介绍如何自动化部署我们的前端和后端项目

本篇主要介绍如何自动化部署vue项目

下篇讲如何自动化部署nodejs项目

我这里是mac os,如果你用linux就不会碰到很多下面碰到的问题了。

## 相关应用

- [jenkins](https://jenkins.io/zh/)
- [docker](https://docs.docker.com/)

## 整个流程

首先我们把我们的电脑想象成一台服务器，当然如果你有服务器，也可以直接在服务器上，执行以下操作

我这里方便起见，就直接把我的电脑（mac）当做一台服务器，事实证明这里面很多坑！！！

注意区分以下本地和服务器的区别，其实我们这里都是我们的电脑

1. 我们在服务器（这里是我们的mac）上安装docker
2. 我们在服务器（这里是我们的mac）上用docker安装了jenkins
3. 我们本地新建一个vue项目，并推到github
4. 我们本地访问服务器上的jenkins，即打开网址http://localhost:8081 (如果是服务器这里localhost换成你服务器的ip)
5. 我们用jenkins拉了github的项目到jenkins的安装目录里的`workspace/`目录
6. jenkins使用`执行shell`跑了命令

```
npm install
npm run build
```

这里我们需要给jenkins安装nodejs插件

1. 我们用jenkins ssh连接到我们的服务器

这里我们需要安装Publish Over SSH插件，并配置好服务器

1. 在服务器（这里是我们的mac）上执行以下操作

把我们的vue项目打包成镜像vuenginxapp

用这个镜像创建一个容器vuenginxapp

## 安装docker

请参考 [www.runoob.com/docker/maco…](https://www.runoob.com/docker/macos-docker-install.html)

我这里是mac, 安装成功之后，在状态栏有docker图标

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1bc8ef7076ab5) 

这时命令行输入`docker -v`

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1bc9f718860f9) 

则说明安装成功

## docker安装jenkins

题外话：你也可以不通过docker安装，可以参考我的另一篇文章[Mac安装jenkins](https://juejin.im/post/5d52e1865188255d691bdf59)

### 搜索jenkins镜像

[hub.docker.com/](https://hub.docker.com/) 是docker存放image(镜像)的地方，界面如下：

![img](Jenkins+Docker自动化部署vue项目.assets/16e1bcbf3a6ade18)

我们搜索一下要安装的jenkins镜像

![img](Jenkins+Docker自动化部署vue项目.assets/16e1bcccefd2be5f)

**注意: 虽然第一个是docker官方维护的版本，但它很长时间没有更新了，是一个过时的版本。所以这里我们要安装第二个，这个是jenkins维护的**

ps：我们也可以通过命令行搜索,命令是`docker search jenkins`

### docker安装jenkins

新建一个目录jenkins_home，如我这里是

![img](Jenkins+Docker自动化部署vue项目.assets/16e1bd951db0989d)

执行命令

```
docker run --name jenkins_node -d -v /Users/isaacho/Documents/application/docker/jenkins_home:/var/jenkins_home -p 8081:8080 -p 50000:50000 jenkins/jenkins:lts
```

**注意： 这里不要直接复制上面的命令，需要把/Users/isaacho/Documents/application/docker/jenkins_home替换成你新建的目录。同样下面如果出现/Users/isaacho/Documents/application/docker/jenkins_home都需要换成你的目录**

等待安装过程。。。，如果能打开`http://localhost:8081`则说明安装成功

同时我们可以看到/Users/isaacho/Documents/application/docker/jenkins_home目录下生成了很多文件

命令解释：

--name jenkins_node 表示你的容器名字叫jenkins_node

-d 表示在后台运行

-v /Users/isaacho/Documents/application/docker/jenkins_home:/var/jenkins_home 表示把我们新建的jenkins_home目录映射到容器的/var/jenkins_home目录

-p 就是端口映射

### 配置jenkins

浏览器打开`http://localhost:8081`,界面如下： 

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1be1166a1e24a)  

我们通过以下命令获取管理员密码

```
cat /Users/isaacho/Documents/application/docker/jenkins_home/secrets/initialAdminPassword
```

点击继续  

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1bf7f6f03d3bf) 

选择推荐的插件，安装插件，然后等待。。。

### 创建管理员用户

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1c4cb6b28c9a4) 

点击保存并完成

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1c4d16669764e) 

点击开始使用jenkins

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1c4e82f463458) 

### 安装ssh插件

点击系统管理

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1c51d27b6711a) 

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1c5252ef5a3fa) 

搜索`Publish Over SSH`

### 配置ssh

系统管理=》configure system =》拉到最下面

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1d805c5996025) 

点击 Test Configuration,显示success则说明配置成功，点击保存

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1d80b5a6e5f3d) 

如果不成功，说明我们的电脑没有开启远程登录

解决方案:

选择系统偏好设置->选择共享->点击远程登录

上面的hostname即你这里的IP地址

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1fd2e797a35c4) 

### 安装node插件

node插件在跑node项目时需要使用，这里先安装好

点击系统管理

![img](Jenkins+Docker自动化部署vue项目.assets/16e1c51d27b6711a-1572786257770)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c5252ef5a3fa-1572786257908)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c53201ee0c16)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c53946f7935d)

等待安装好

### 选择安装nodejs版本

![img](Jenkins+Docker自动化部署vue项目.assets/16e1c547f8e09e84)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c564d7574495)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c56f940e840d)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c5791ff1f995)

## 准备一个vue项目

项目已经上传到[github](https://github.com/repototest/vueclidemoapp)

### vue cli创建vue项目

我们用vue-cli3直接创建一个vue项目

**注意： 这里需要你的node版本大于8.9 或更高版本**

```
vue create vueclidemoapp
```

### 改写

修改HelloWorld.vue组件，改写页面，并加入axios发送一个请求

```
import axios from 'axios';
axios.get('/api', {
  params: {},
}).then((res) => {
  console.log(res);
});

...
```

界面如下

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1c46ea0b5ef96) 

### 创建nginx配置文件

关于nginx配置请看[前端想要了解的Nginx](https://juejin.im/post/5cae9de95188251ae2324ec3)

在项目根目录下新建nginx.conf

```conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    gzip  on;
    gzip_types text/plain application/javascript text/css;

    # 虚拟主机server块
    server {
        # 端口
        listen   80;
        # 匹配请求中的host值
        server_name  localhost;
        
        # 监听请求路径
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }

    }

    include /etc/nginx/conf.d/*.conf;
}
```

### 创建Dockerfile

在项目根目录下新建Dockerfile

```
FROM nginx

EXPOSE 80
```

此时你的目录结构是这样

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1c597d572638f) 

### 把这个项目上传到github

这里我就不演示了，我的这个项目的github地址是https://github.com/repototest/vueclidemoapp

## jenkins部署vue项目

### 新建任务

![img](Jenkins+Docker自动化部署vue项目.assets/16e1c5be35feccbe)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c5c753840c11)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c5d16fc805a2)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c5df8a482a3c)



![img](Jenkins+Docker自动化部署vue项目.assets/16e215a048679398)

第一个shell命令是build我们的vue项目，会在项目根目录下生成dist目录

```
echo $PATH
node -v
npm -v
npm install
npm run build
```

第二个shell命令是用来构建docker镜像和跑docker服务的

它首先会ssh连接到我们的服务器

```
docker stop vuenginxapp || true \
 && docker rm vuenginxapp || true \
 && cd /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp  \
 && docker build  -t vuenginxapp  . \
 && docker run -d -p 8083:80 --name vuenginxapp -v /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp/dist:/usr/share/nginx/html -v /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp/nginx.conf:/etc/nginx/nginx.conf vuenginxapp
```

`docker stop vuenginxapp`用于停止vuenginxapp容器

`sudo docker rm vuenginxapp`用于删除vuenginxapp容器

`cd /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp`进入我们的项目目录，这里需要改成你们自己的目录，即`(yourpath)/jenkins_home/workspace/vueclidemoapp`, yourpath就是上面我们新建jenkins_home目录时的目录

`docker build -t vuenginxapp .` 表示build镜像，名称为vuenginxapp

`docker run -d -p 8083:80 --name vuenginxapp -v /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp/dist:/usr/share/nginx/html -v /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp/nginx.conf:/etc/nginx/nginx.conf vuenginxapp` 表示已我们上面的vuenginxapp为镜像生成容器vuenginxapp

详细解释 `-d` 在后台运行

`-p 8083:80` 把容器的80端口映射程我们访问的8083端口，注意80端口是nginx默认的端口号

`--name vuenginxapp` 容器名称为vuenginxapp

`-v /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp/dist:/usr/share/nginx/html` 把我们生成的dist目录挂载到容器的/usr/share/nginx/html，/usr/share/nginx/html是nginx默认的http目录

`-v /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp/nginx.conf:/etc/nginx/nginx.conf` 把我们的nginx.conf挂载到容器的/etc/nginx/nginx.conf，/etc/nginx/nginx.conf是nginx默认的配置文件

`vuenginxapp`是我们的镜像名称

#### 可能遇到的问题

- docker不是一个命令 出现这个问题的原因是，我们用ssh连接mac时，命令行只能使用/usr/bin里的命令，但我们的docker命令是放在/usr/local/bin里的，所以找不到，解决方案是用/usr/local/bin/docker 代替上面的docker

即

```
/usr/local/bin/docker stop vuenginxapp || true \
 && /usr/local/bin/docker rm vuenginxapp || true \
 && cd  /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp  \
 && /usr/local/bin/docker build  -t vuenginxapp  . \
 && /usr/local/bin/docker run -d -p 8083:80 --name vuenginxapp -v  /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp/dist:/usr/share/nginx/html -v  /Users/isaacho/Documents/application/docker/jenkins_home/workspace/vueclidemoapp/nginx.conf:/etc/nginx/nginx.conf vuenginxapp
复制代码
```

### 构建项目

点击立即构建

 ![img](Jenkins+Docker自动化部署vue项目.assets/16e1c64984ee0007) 

等待部署。。。

点击这个小圆可以看控制台输出

![img](Jenkins+Docker自动化部署vue项目.assets/16e1c65a64e6a233)



![img](Jenkins+Docker自动化部署vue项目.assets/16e1c65ea2e79980)

可以看到我们的项目被拉到`(yourpath)/jenkins_home/workspace`下了

![img](Jenkins+Docker自动化部署vue项目.assets/16e1c630b47a8fa2)

访问http://localhost:8083/

![img](Jenkins+Docker自动化部署vue项目.assets/16e1fd9231b9090f)

至此我们成功部署了我们的vue项目

#### 可能遇到的问题

- Get [registry-1.docker.io/v2/:](https://registry-1.docker.io/v2/:) net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)


# frp + nginx 配置多人共用的http 内网穿透服务

## 一、 前言

**frp** 是一个用**Go**语言开发的，可用于**内网穿透**的高性能的反向代理应用，支持 tcp, udp 、 http 和 https。可将一个部署在本机的web服务映射到外网。

本文主要讲如何基于**frp** \+ **nginx** 配置http 内网穿透服务，承载**多人**同时使用，**从而支持微信公众号，微信小程序的本地开发调试**

所需资源：

*   一台公网服务器或者VPS（本人用的 腾讯云的主机）

*   一个指向到此台公网服务器的域名（本文以**msh.com** 为例）


本文涉及的环境

*   centos7.2

*   nginx 1.10.1

*   frp 0.22.0

*   go 1.11.4

*   Windows 10


## 二 、 frp 原理

（请仔细阅读原理，在不理解原理的情况下上手配置容易出错，且很难定位原因。这都是本人所经历的惨痛教训）

以本人搭建的**frp**内网穿透服务为例：

**第一步：** 配置无误的情况下，frp服务端和frp客户端先后启动，建立**通信隧道**，其中：

*   frp服务端监听http **7071**端口（此端口可自定义），接收**此端口下**所有外网用户请求

*   frp客户端代理本地想要暴露给外网的web服务端口，本文以**8585** , **8686** 端口为例


**第二步：** 通过配置**nginx**反向代理，将指向本台公网服务器的**dev.msh.com** 下的子域名，映射到服务器的**7071**端口，也就是frp监听的那个端口。 外网用户访问**dev.msh.com**下的子域名，例如 ：

*   **a.dev.msh.com**

*   **b.dev.msh.com**


等同于访问**msh.com：7071**，会 **触发** frp服务端和客户端的互动，从而http请求由frp服务端传递到frp客户端

**第三步：** frp客户端收到http请求后，基于自定义配置，则做如下处理：

*   监听到http请求中的域名为 **a.dev.msh.com**，则将请求转发到我本地的**8585**web服务端口

*   监听到http请求中的域名为 **b.dev.msh.com**，则将请求转发到我本地的**8686**web服务端口


**第四步：** 本地的web服务收到http请求后，对请求做处理，并完成响应

**第五步：** frp客户端将响应结果回传给frp的服务端。服务端最终将响应回传给外网用户

**第六步：** 最终的实测效果为：

*   访问 **a.dev.msh.com**，等同于访问我本地的**localhost:8585**

*   访问 **b.dev.msh.com**，等同于访问我本地的**localhost:8686**


## 三 、 准备工作

### **3.1 在域名解析后台配置子域名**

本文以**msh.com** 为例：

登录域名的解析后台，在**msh.com**下增加两条A记录： **dev** , ***.dev**,记录值为部署frp服务端的公网服务器的ip。

代表**dev.msh.com**下的所有的子域名，会全部指向此台公网服务器。

### **3.2 关于 go语言环境**

因为本文采用的是**绿色安装**，所以不需要配置go语言环境。多谢 Tylerrrkd 指正

## 四、服务端配置

### **4.1 frp服务端安装配置**

**下载解压**

```bash
# 下载 
wget https://github.com/fatedier/frp/releases/download/v0.22.0/frp_0.22.0_linux_amd64.tar.gz 
# 解压 
tar -zxvf frp_0.22.0_linux_amd64.tar.gz
```

**修改配置文件**

解压后进入解压目录，找到**frps.ini**文件, 做如下配置 。配置说明请参见各项对应的注释

```
[common] 
# frp监听的端口，用作服务端和客户端通信 
bind_port = 7000 

# 服务端通过此端口接监听和接收公网用户的http请求 
vhost_http_port = 7071 

# frp提供了一个控制台，可以通过这个端口访问到控制台。可查看frp当前有多少代理连接以及对应的状态 
dashboard_port = 7500 

# 服务端的subdomain_host需要和客户端配置文件中的subdomain、local_port配合使用， 
# 可通过{subdomain}.{subdomain_host} 的域名格式来访问自己本地的 web 服务。 
# 假如服务端的subdomain_host为dev.msh.com，客户端某个配置组中的 
# subdomain为a,local_port为8585， 
# 则： 
# 访问 a.dev.msh.com ，等同于访问本地的localhost:8585 
subdomain_host = dev.msh.com
```

**启动frp服务端**

```
./frps -c frps.ini
```

### **4.2 nginx反向代理配置**

修改 **nginx.conf** 文件

```
 # frp的接收http请求的反向代理 
 server { 
 listen 80; 
 server_name *.dev.msh.com dev.msh.com; 
 
 location / {
 # 7071端口即为frp监听的http端口 
 proxy_pass http://127.0.0.1:7071; 
 proxy_set_header Host $host:80; 
 proxy_set_header X-Real-IP $remote_addr; 
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
 
 proxy_set_header Upgrade $http_upgrade; 
 proxy_set_header Connection "upgrade"; 
 
 proxy_connect_timeout 7d; 
 proxy_send_timeout 7d; 
 proxy_read_timeout 7d; 
 } 
 # 防止爬虫抓取 
 if ($http_user_agent ~* "360Spider|JikeSpider|Spider|spider|bot|Bot|2345Explorer|curl|wget|webZIP|qihoobot|Baiduspider|Googlebot|Googlebot-Mobile|Googlebot-Image|Mediapartners-Google|Adsbot-Google|Feedfetcher-Google|Yahoo! Slurp|Yahoo! Slurp China|YoudaoBot|Sosospider|Sogou spider|Sogou web spider|MSNBot|ia_archiver|Tomato Bot|NSPlayer|bingbot") 
 {
 return 403; 
 }
}
```

**让nginx重新加载配置文件**

```
/usr/local/nginx/sbin/nginx -s reload
```

### **4.3 开启防火墙端口**

```
# 开启防火墙端口 7000端口和7071端口即为上面配置的bind_port和vhost_http_port端口
firewall-cmd --zone=public --add-port=7000/tcp --permanent 
firewall-cmd --zone=public --add-port=7071/tcp --permanent 

# 开启后重启防火墙，使得刚刚的修改生效 
firewall-cmd --reload
```

## 五、 客户端安装配置

### **下载客户端**

去github上面下载最新版的 windows客户端 github.com/fatedier/fr… ，找到 **frp\_0.23.1\_windows_amd64.zip**，点击下载即可

（Mac用户请下载Mac版本的客户端）

### **解压后，编辑 frpc.ini 文件**

```
[common] 
# 部署frp服务端的公网服务器的ip 
server_addr = 132.232.64.79 
# 和服务端的bind_port保持一致 
server_port = 7000 

# 代理服务一 ，[]内的代理服务名称在全局范围内确保唯一，每个人的每个代理服务不能重名， 
# 否则会影响正常使用。 
[http-a] 
type = http 
# local_port代表你想要暴露给外网的本地web服务端口 
local_port = 8585 
# subdomain 在全局范围内要确保唯一，每个代理服务的subdomain不能重名，否则会影响正常使用。 
# 客户端的subdomain需和服务端的subdomain_host配合使用 
subdomain = a 

# 代理服务二 ，各项配置说明请参考配置组一 
[http-b] 
type = http 
local_port = 8686 
subdomain = b
```

### **启动 客户端**

在frp解压目录下右键打开 **powershell** 或者**cmd**，执行如下命令

```
./frpc.exe -c .rpc.ini
```

如果窗口提示 **『start proxy success』** ，则代表frp服务端和frp客户端的**通信隧道**建立成功

### **测试访问**

在浏览器里面访问 **http://a.dev.msh.com**、测试本地的web服务是否已经暴露给外网
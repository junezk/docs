# Harbor私有镜像仓库无坑搭建

## 一、介绍

Docker容器应用的开发和运行路不开可靠的镜像管理，虽然Docker官方也提供了公共的镜像仓库，但是从安全和效率等方面考虑，部署我们私有环境的Registry也是非常必要的。 Harbor是由`VMware`公司开源的企业级的Docker Registry管理项目，它包括权限管理(RBAC)、LDAP、日志审核、管理界面、自我注册、镜像复制和中文支持等功能。

## 二、组件

用于部署 Harbor 的 Docker Compose 模板位于 /Deployer/docker-compose.yml. 打开这个模板文件，会发现 Harbor 由 5 个容器组成：

- proxy：由 Nginx 服务器构成的反向代理。
- registry：由 Docker 官方的开源 registry 镜像构成的容器实例。
- ui：即架构中的 core services, 构成此容器的代码是 Harbor 项目的主体。
- mysql：由官方 MySql 镜像构成的数据库容器。
- log: 运行着 rsyslogd 的容器，通过 log-driver 的形式收集其他容器的日志。

这几个容器通过 **Docker link** 的形式连接在一起，这样，在容器之间可以通过容器名字互相访问。对终端用户而言，只需要暴露 proxy (即 Nginx)的服务端口。

## 三、工作原理

假设我们将 Harbor 部署在主机名为 registry.abcdocker.com 的虚机上。用户通过 docker login 命令向这个 Harbor 服务发起登录请求:docker login registry.abcdocker.com当用户输入所需信息并点击回车后，Docker 客户端会向地址“registry.abcdocker.com/v2/” 发出 HTTP GET 请求。

Harbor 的各个容器会通过以下步骤处理：

### (1)Docker login

(a) 首先，这个请求会由监听 80 端口的 proxy 容器接收到。根据预先设置的匹配规则，容器中的 Nginx会将请求转发给后端的 registry 容器；

(b) 在 registry 容器一方，由于配置了基于 token 的认证，registry 会返回错误代码 401，提示 Docker客户端访问 token 服务绑定的 URL。在 Harbor 中，这个 URL 指向 Core Services；

(c) Docker 客户端在接到这个错误代码后，会向token服务的URL发出请求，并根据HTTP协议的BasicAuthentication 规范，将用户名密码组合并编码，放在请求头部(header)；

(d)类似地，这个请求通过 80 端口发到 proxy 容器后，Nginx 会根据规则把请求转发给 ui 容器，ui 容器监听 token 服务网址的处理程序接收到请求后，会将请求头解码，得到用户名、密码；

(e) 在得到用户名、密码后，ui 容器中的代码会查询数据库，将用户名、密码与 mysql 容器中的数据进行比对(注:ui 容器还支持 LDAP 的认证方式，在那种情况下 ui 会试图和外部 LDAP 服务进行通信并校验用户名/密码)。比对成功，ui 容器会返回表示成功的状态码， 并用密钥生成 token，放在响应体中返回给 Docker 客户端。这个过程中组件间的交互过程如下图所示

![img](Harbor私有镜像仓库无坑搭建.assets/16daa1888323d6e3)

**至此，一次 docker login 成功地完成了，Docker 客户端会把步骤(c)中编码后的用户名密码保存在本地的隐藏文件中。**

### (2) Docker Push

用户登录成功后用 docker push 命令向 Harbor 推送一个 Docker 镜像：docker push registry.abcdocker.com/library/hello-world

(a) 首先，docker 客户端会重复 login 的过程，首先发送请求到 registry,之后得到 token 服务的地址；

(b) 之后，Docker 客户端在访问ui容器上的token服务时会提供额外信息，指明它要申请一个对imagelibrary/hello-world 进行 push 操作的 token；

(c) token 服务在经过 Nginx 转发得到这个请求后，会访问数据库核实当前用户是否有权限对该 image进行 push。如果有权限，它会把 image 的信息以及 push 动作进行编码，并用私钥签名，生成 token返回给 Docker 客户端；

(d) 得到 token 之后 Docker 客户端会把 token 放在请求头部，向 registry 发出请求，试图开始推送image。 Registry 收到请求后会用公钥解码 token 并进行核对，一切成功后，image 的传输就开始了。我们省去 proxy 转发的步骤，下图描述了这个过程中各组件的通信

![img](Harbor私有镜像仓库无坑搭建.assets/16daa18883a2fa5e)

## 四、安装环境

| IP地址        | docker版本 | docker-compose版本 | 内核    | Centos |
| ------------- | ---------- | ------------------ | ------- | ------ |
| 192.168.1.197 | 17.03      | 1.22               | 4.17.12 | 7.4    |

**注意：**Harbor的所有服务组件都是在Docker中部署的，所以官方安装使用Docker-compose快速部署，所以我们需要安装Docker、Docker-compose. 由于Harbor是基于Docker Registry V2版本，所以就要求Docker版本不小于1.10.0，Docker-compose版本不小于1.6.0

## 五、环境准备

```
# 设置主机名
hostnamectl set-hostname harbor

# 时间同步
yum -y install ntp
 systemctl enable ntpd
 systemctl start ntpd
 ntpdate -u cn.pool.ntp.org
 hwclock --systohc
 timedatectl set-timezone Asia/Shanghai
 
# 关闭swap分区
vim /etc/fstab  #永久关闭swap分区
swap was on /dev/sda11 during installation
UUID=0a55fdb5-a9d8-4215-80f7-f42f75644f69 none  swap    sw      0       0
#注释掉SWAP分区项，即可
或者
```

echo "vm.swappiness = 0">> /etc/sysctl.conf
sysctl -p
swapoff -a && swapon -a

```
# 关闭防火墙 Selinux
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config
 
# 升级内核
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml -y&&
sed -i s/saved/0/g /etc/default/grub&&
grub2-mkconfig -o /boot/grub2/grub.cfg && reboot
```

## 六、安装Docker、Docker-compose

### 下载Docker安装包

```
wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm
wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm
复制代码
➜ yum install docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm -y
➜ yum install docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm -y
```

``

```
设置开机启动并启动docker
systemctl enable docker 
systemctl start docker 

替换docker相关配置
sed -i '/ExecStart=\/usr\/bin\/dockerd/i\ExecStartPost=\/sbin/iptables -I FORWARD -s 0.0.0.0\/0 -d 0.0.0.0\/0 -j ACCEPT' /usr/lib/systemd/system/docker.service
sed -i '/dockerd/s/$/ \-\-storage\-driver\=overlay2 --insecure-registry 192.168.60.26/g' /usr/lib/systemd/system/docker.service复制代码
注意：
1.Docker默认从官方拉取镜像,并且从1.12版本之后，默认私有仓库是使用https来进行连接,我们需要将使用私服的主机上docker启动文件进行修改，在启动命令添加--insecure-registry 192.168.1.197
Docker-compose Install
```

Github地址：[github.com/docker/comp…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fdocker%2Fcompose%2Freleases)

### Docker Compose简介

> Docker Compose是Docker编排服务的最后一块，前面提到的Machine可以让用户在其它平台快速安装Docker，Swarm可以让Docker容器在集群中高效运转，而Compose可以让用户在集群中部署分布式应用。简单的说，Docker Compose属于一个“应用层”的服务，用户可以定义哪个容器组运行哪个应用，它支持动态改变应用，并在需要时扩展。

### 安装Docker-compose

```
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

$ docker-compose --version
docker-compose version 1.22.0, build f46880fe
```

Docker-compose常用命令

```
docker-compose up -d               ###后台启动，如果容器不存在根据镜像自动创建

docker-compose down   -v         ###停止容器并删除容器

docker-compose start                 ###启动容器，容器不存在就无法启动，不会自动创建镜像

docker-compose stop                 ###停止容器
```

### 安装Harbor

Harbor官方地址
[github.com/vmware/harb…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fvmware%2Fharbor%2Freleases)

### Harbor 2种安装方式

```
1.离线安装
$ wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.5.0.tgz
$ tar xf harbor-online-installer-v1.5.0.tgz

2.在线安装
$ wget https://storage.googleapis.com/harbor-releases/harbor-online-installer-v1.5.2.tgz
$ tar xf harbor-online-installer-v1.5.2.tgz

 重点：下载那个823M的安装
```

### 配置Harbor

解压之后，配置文件名称harbor.cfg，该文件就是Harbor的配置文件

编辑harbor.cfg
实际上我们只需要修改Hostname字段

```
[root@harbor harbor]# head  harbor.cfg 
## Configuration file of Harbor

#This attribute is for migrator to detect the version of the .cfg file, DO NOT MODIFY!
_version = 1.5.0
#The IP address or hostname to access admin UI and registry service.
#DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname = 192.168.1.197
```

### harbor.cfg相关配置解释

```
[root@master harbor]# grep -Ev "^$|^[#;]" harbor.cfg 
# Harbor版本
_version = 1.5.0

# hostname 设置访问地址，可以使用IP/域名，不可以设置127.0.0.1 && localhost
hostname = 192.168.60.24

# 访问协议,默认是http,也可以设置https,如果设置https,则nginx ssl需要设置on
ui_url_protocol = http

# Job 最大进程数
max_job_workers = 50 

# 是否创建证书,创建证书将会在下面的路径下生成
customize_crt = on
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key

# 私钥存储路径
secretkey_path = /data

# 设置日志大小
admiral_url = NA
log_rotate_count = 50
log_rotate_size = 200M

# 是否使用代理
http_proxy =
https_proxy =
no_proxy = 127.0.0.1,localhost,ui

# 邮箱设置，发送重置密码邮件时使用
email_identity = 
email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false
email_insecure = false

# 安装Harbor后，管理员UI登陆的密码,默认是Harbor12345
harbor_admin_password = Harbor12345

# 认证方式，这里支持多种认证方式，比如LDAP、数据库认证。默认是db_auth
auth_mode = db_auth

# LDAP认证时配置项
ldap_url = ldaps://ldap.mydomain.com
ldap_basedn = ou=people,dc=mydomain,dc=com
ldap_uid = uid 
ldap_scope = 2 
ldap_timeout = 5
ldap_verify_cert = true
ldap_group_basedn = ou=group,dc=mydomain,dc=com
ldap_group_filter = objectclass=group
ldap_group_gid = cn
ldap_group_scope = 2

#是否开启自动注册
self_registration = on

# Token有效时间，默认30分钟
token_expiration = 30

# 用户创建项目权限控制，默认是everyone(所有人),也可以设置为adminonly(管理员)
project_creation_restriction = everyone

# Mysql数据库root用户默认密码root123，根据实际时使用来进行修改
db_host = mysql
db_password = root123
db_port = 3306
db_user = root

# Redis配置
redis_url = redis:6379
clair_db_host = postgres
clair_db_password = password
clair_db_port = 5432
clair_db_username = postgres
clair_db = postgres
uaa_endpoint = uaa.mydomain.org
uaa_clientid = id
uaa_clientsecret = secret
uaa_verify_cert = true
uaa_ca_cert = /path/to/ca.pem
registry_storage_provider_name = filesystem
registry_storage_provider_config = 
```

本次我们使用离线安装，安装包在上面

```
tar xf harbor-offline-installer-v1.5.0.tgz
./install
```

![img](Harbor私有镜像仓库无坑搭建.assets/16daa18884b1e92f)

![img](Harbor私有镜像仓库无坑搭建.assets/16daa1888553b610)

## 七、访问Harbor

访问：192.168.1.197

默认端口为80，需要修改端口请修改docker-compose.yaml对应服务的映射

![img](Harbor私有镜像仓库无坑搭建.assets/16daa18885762653)

## 八、客户端配置免https

```
[root@localhost ~]# echo '{ "insecure-registries":["172.16.1.146:8888"] }' > /etc/docker/daemon.json
 [root@localhost ~]# cat /etc/docker/daemon.json
{ "insecure-registries":["172.16.1.146:8888"] } 
restart docker 
[root@localhost ~]# service docker restart
```

如果不配置，客户端使用时候会报错： Error response from daemon: Get https:// 172.16.1.146:5000/v1/_ping: http: server gave HTTP response to HTTPS client

```
[root@localhost harbor]# docker login 172.16.1.146:8888
Username (admin): admin
Password: 
Login Succeeded
```

## 九、管理镜像

### 打标签并且上传

```
docekr tag 镜像名称:标签 你的IP:端口/项目名称/镜像名称:标签 
docker push 你的IP:端口/项目名称/镜像名称:标签复制代码
[root@localhost harbor]#  docker tag registry:2.6.2 172.16.1.146:8888/wondertek/registry:2.6.2
[root@localhost harbor]#  docker push  172.16.1.146:8888/wondertek/registry:2.6.2
The push refers to a repository
 [172.16.1.146:8888/wondertek/registry]
9113493eaae1: Layer already exists 
621c2399d41a: Layer already exists 
59e80739ed3f: Layer already exists 
febf19f93653: Layer already exists 
e53f74215d12: Layer already exists 
2.6.2: digest: sha256:feb40d14cd33e646b9985e2d6754ed66616fedb840226c4d917ef53d616dcd6c size: 1364
```

### 删除本地镜像 重新下载

```
[root@localhost harbor]# docker rmi 172.16.1.146:8888/wondertek/registry:2.6.2
Untagged: 172.16.1.146:8888/wondertek/registry:2.6.2
Untagged: 172.16.1.146:8888/wondertek/registry@sha256:feb40d14cd33e646b9985e2d6754ed66616fedb840226c4d917ef53d616dcd6c

[root@localhost harbor]# docker pull 172.16.1.146:8888/wondertek/registry:2.6.2
Trying to pull repository 172.16.1.146:8888/wondertek/registry ... 
2.6.2: Pulling from 172.16.1.146:8888/wondertek/registry
Digest: sha256:feb40d14cd33e646b9985e2d6754ed66616fedb840226c4d917ef53d616dcd6c
Status: Downloaded newer image for 172.16.1.146:8888/wondertek/registry:2.6.2
```
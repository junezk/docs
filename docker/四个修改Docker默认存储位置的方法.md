# [四个修改Docker默认存储位置的方法](https://www.cnblogs.com/whmbky/p/7843580.html)



**我使用方法二、修改镜像和容器的存放路径**

指定镜像和容器存放路径的参数是--graph=/var/lib/docker，我们只需要修改配置文件指定启动参数即可。

Docker 的配置文件可以设置大部分的后台进程参数，在各个操作系统中的存放位置不一致，在 Ubuntu 中的位置是：/etc/default/docker，在 CentOS 中的位置是：/etc/sysconfig/docker。

**如果是 Ubuntu 则添加下面这行（因为 Ubuntu 默认没开启 selinux）：**

OPTIONS=--graph="/root/data/docker" -H fd://# 或者DOCKER_OPTS="-g /root/data/docker"

最后重新启动，Docker 的路径就改成 /root/data/docker 了。

**如果是centos7：**

修改docker.service文件，使用-g参数指定存储位置

vi /usr/lib/systemd/system/docker.service  

ExecStart=/usr/bin/dockerd --graph /new-path/docker 

 // reload配置文件 

systemctl daemon-reload 

 // 重启docker 

systemctl restart docker.service

//查看 Docker Root Dir: /var/lib/docker是否改成设定的目录/new-path/docker 

docker info


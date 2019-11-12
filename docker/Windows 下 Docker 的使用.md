# Windows 下Docker的使用

## 基本操作

- docker pull image-name:tag  拉取镜像
- docker images 显示已下载镜像
- docker rmi 镜像id|镜像名  删除镜像
- docker ps -a 查看容器
- docker rm 容器id  删除容器
- docker run 运行容器
- docker stop 结束运行容器
- docker start 重新运行容器

## 示例

- docker run -p 27017:27017 -v <LocalDirectoryPath>:/data/db --name docker_mongodb -d mongo

  在上面的命令中，几个命令参数的详细解释如下：

  -p  端口定义

  -v 数据映射，将本机目录映射到容器中

  --name 给容器起名字

  -d 以守护进程方式运行

## Docker debian镜像加速命令：

```
#更新apt-get源 使用163的源
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak \
    echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >/etc/apt/sources.list && \
    echo "deb http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
```

Docker ubuntu镜像加速命令：

```
FROM ubuntu:latest
RUN  sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list \
&& apt-get clean \
&& apt-get update
```



## pip install 使用豆瓣源加速

```
 pip install -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com xxxx
```


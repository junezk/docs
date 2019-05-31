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
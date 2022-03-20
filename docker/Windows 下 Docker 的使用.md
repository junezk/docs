# Windows 下Docker的使用



## 基本操作

- docker pull image-name:tag  拉取镜像
- docker search 镜像名  查询镜像
- docker images 显示已下载镜像
- docker rmi 镜像id|镜像名  删除镜像
- docker ps -a 查看容器
- docker rm 容器id  删除容器
- docker run 运行容器
- docker stop 结束运行容器
- docker start  重新运行容器
- docker inspect 容器id   查看容器详细信息
  - docker inspect -f {{.State.Pid}} 容器id   查看容器pid
- 进入容器
  - docker attach 容器名/id   进入容器，当多个窗口同时使用该命令进入该容器时，所有的窗口都会同步显示。如果有一个窗口阻塞了，那么其他窗口也无法再进行操作。
  - 使用SSH进入Docker容器
  - 使用nsenter进入Docker容器  需要主机安装`nsenter`。
    - nsenter --target [容器pid] --mount --uts --ipc --net --pid
  - docker exec -it [容器id] /bin/bash   进入容器，开启终端
- 导入、导出镜像
  - docker save -o 文件名 镜像名    例如：`docker save > nginx.tar nginx:latest`
  - docker load -i 文件名   例如：`docker load < nginx.tar `
- 导入、导出容器
  - docker export -o 文件名 容器名
  - docker import 文件名 镜像名  例如：`docker import nginx-test.tar nginx:imp`
- docker logs 容器名   查看容器运行日志

两种方式的区别：导入导出镜像可以保留镜像的分层信息，导入导出容器将丢失这些信息。因此，备份镜像、迁移镜像使用上面的方法，启动容器后，容器内容有修改变化需要备份，使用下面的方法。

## 示例

- docker run -p 27017:27017 -v <LocalDirectoryPath>:/data/db --name docker_mongodb -d mongo

  在上面的命令中，几个命令参数的详细解释如下：

  -p  端口定义

  -v 数据映射，将本机目录映射到容器中

  --name 给容器起名字

  -d 以守护进程方式运行


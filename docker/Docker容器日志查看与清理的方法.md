# [Docker容器日志查看与清理的方法](http://www.west.cn/docs/51325.html)

## 1. 问题

docker容器日志导致主机磁盘空间满了。`docker logs -f container_name`噼里啪啦一大堆，很占用空间，不用的日志可以清理掉了。 

## 2. 解决方法

### 2.1 找出Docker容器日志

在linux上，容器日志一般存放在`/var/lib/docker/containers/container_id/`下面，查看各个日志文件大小的脚本docker_log_size.sh，内容如下：

```bash
#!/bin/sh
echo "======== docker containers logs file size ========"  
logs=$(find /var/lib/docker/containers/ -name *-json.log)  
for log in $logs  
        do  
             ls -lh $log   
        done  
# chmod +x docker_log_size.sh
# ./docker_log_size.sh
```



### 2.2 清理Docker容器日志（治标）

如果docker容器正在运行，那么使用`rm -rf`方式删除日志后，通过`df -h`会发现磁盘空间并没有释放。原因是在Linux或者Unix系统中，通过`rm -rf`或者文件管理器删除文件，将会从文件系统的目录结构上解除链接（unlink）。如果文件是被打开的（有一个进程正在使用），那么进程将仍然可以读取该文件，磁盘空间也一直被占用。正确姿势是`cat /dev/null > *-json.log`，当然你也可以通过`rm -rf`删除后重启docker。接下来，提供一个日志清理脚本`clean_docker_log.sh`，内容如下：

```bash
#!/bin/sh 
echo "======== start clean docker containers logs ========"  
logs=$(find /var/lib/docker/containers/ -name *-json.log)  
for log in $logs  
        do  
                echo "clean logs : $log"  
                cat /dev/null > $log  
        done  

echo "======== end clean docker containers logs ========"  
# chmod +x clean_docker_log.sh
# ./clean_docker_log.sh
```

但是，这样清理之后，随着时间的推移，容器日志会像杂草一样，卷土重来。

### 2.3 设置Docker容器日志大小（治本）

- 设置一个容器服务的日志大小上限

上述方法，日志文件迟早又会涨回来。要从根本上解决问题，需要限制容器服务的日志大小上限。这个通过配置容器docker-compose的max-size选项来实现

```yaml
nginx: 
  image: nginx:1.12.1 
  restart: always 
  logging: 
    driver: “json-file” 
    options: 
      max-size: “5g” 
```

重启nginx容器之后，其日志文件的大小就被限制在5GB，再也不用担心了。

- 全局设置

新建/etc/docker/daemon.json，若有就不用新建了。添加log-dirver和log-opts参数，样例如下：

```json
# vim /etc/docker/daemon.json
{
  "registry-mirrors": ["http://f613ce8f.m.daocloud.io"],
  "log-driver":"json-file",
  "log-opts": {"max-size":"500m", "max-file":"3"}
}
```

max-size=500m，意味着一个容器日志大小上限是500M， 
max-file=3，意味着一个容器有三个日志，分别是id+.json、id+1.json、id+2.json。

```
// 重启docker守护进程
# systemctl daemon-reload
# systemctl restart docker
```

> 注意：设置的日志大小，只对新建的容器有效。  

## 3. 参考文章

https://blog.csdn.net/xunzhaoyao/article/details/72959917 
https://www.cnblogs.com/testzcy/p/7904829.html
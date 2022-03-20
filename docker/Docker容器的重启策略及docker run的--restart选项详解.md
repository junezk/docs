# Docker容器的重启策略及docker run的--restart选项详解

## 1.  Docker容器的重启策略

Docker容器的重启策略是面向生产环境的一个启动策略，在开发过程中可以忽略该策略。

Docker容器的重启都是由Docker守护进程完成的，因此与守护进程息息相关。

Docker容器的重启策略如下：

- no，默认策略，在容器退出时不重启容器
- on-failure，在容器非正常退出时（退出状态非0），才会重启容器
- on-failure:3，在容器非正常退出时重启容器，最多重启3次
- always，在容器退出时总是重启容器
- unless-stopped，在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器

## 2. Docker容器的退出状态码

docker run的退出状态码如下：

- 0，表示正常退出
- 非0，表示异常退出（退出状态码采用chroot标准）
- 125，Docker守护进程本身的错误
- 126，容器启动后，要执行的默认命令无法调用
- 127，容器启动后，要执行的默认命令不存在
- 其他命令状态码，容器启动后正常执行命令，退出命令时该命令的返回状态码作为容器的退出状态码

## 3. docker run的--restart选项

通过--restart选项，可以设置容器的重启策略，以决定在容器退出时Docker守护进程是否重启刚刚退出的容器。

--restart选项通常只用于detached模式的容器。

--restart选项不能与--rm选项同时使用。显然，--restart选项适用于detached模式的容器，而--rm选项适用于foreground模式的容器。

在docker ps查看容器时，对于使用了--restart选项的容器，其可能的状态只有Up或Restarting两种状态。

示例：

```
docker run -d --restart=always bba-208
docker run -d --restart=on-failure:10 bba-208
```


补充：

查看容器重启次数

```
docker inspect -f "{{ .RestartCount }}" bba-208
```

查看容器最后一次的启动时间
```
docker inspect -f "{{ .State.StartedAt }}" bba-208
```

参考链接：
https://docs.docker.com/engine/reference/run/
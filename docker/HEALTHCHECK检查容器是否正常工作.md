# HEALTHCHECK检查容器是否正常工作

HEALTHCHECK指令有两种形式：

HEALTHCHECK [[OPTIONS](https://www.centos.bz/tag/options/)] CMD [command](https://www.centos.bz/tag/command/) [通过在容器内运行一个命令来检查容器健康情况]

HEALTHCHECK NONE [禁用从base镜像继承的任何healthcheck]

HEALTHCHECK指令告诉[Docker](https://www.centos.bz/tag/docker/)如何测试一个容器来检查它是否工作正常。这个可以用来检测如web [server](https://www.centos.bz/tag/server/)陷入了死循环且已经无法处理新的连接了，即使server进程仍然在运行。
当一个容器设置了healthcheck之后，除了正常的状态，它多了一个health状态。这个状态初始为starting。当健康检查通过后，它变成了healthy(不管之前是什么状态)。当连续出现几次失败后，就变成unhealthy。
在CMD之前的选项有：

–interval=DURATION [默认30s]

–timeout=DURATION [默认30s]

–retries=N [默认3]

当容器启动之后，首先等待interval秒然后进行健康检查，然后等这次检查完成后再等interval秒之后继续再次检查，如此循环。
如果有一个检查所花的时间超过了timeout秒，那么就认为这次检查失败了。
如果连续retries次失败，就认为此容器状态为unhealthy。
[Dockerfile](https://www.centos.bz/tag/dockerfile/)中只能使用一个HEALTHCHECK指令，如果设置了多次，就取最后一个。
这个命令的CMD之后的命令可以是一个[shell](https://www.centos.bz/category/shell/)命令[如HEALTHCHECK CMD /bin/check-running]或者exec数组[像Dockerfile命令[ENTRYPOINT](https://www.centos.bz/tag/entrypoint/)使用的数组]。
命令退出状态表示容器的健康状态。可能的值为：

0: success – 容器是健康的。

1: unhealthy – 容器工作不正常。

2: reserved – 不要使用这个退出代码

例如，隔5分钟检查一次容器确保web server能够在3秒内正常输出主页：

1. HEALTHCHECK --interval=5m --timeout=3s \
2.   CMD curl -f http://localhost/ || exit 1

为了方便调查失败原因，检查命令的输出(UTF-8编码)会写到health状态中，并且可以通过docker inspect查看。输出应该尽量短，不大于4096字节。
HEALTHCHECK功能是在Docker 1.12添加的。
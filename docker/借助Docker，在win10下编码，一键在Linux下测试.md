# 借助Docker，在win10下编码，一键在Linux下测试

此前在公司实习时，日常的开发、工作按规定都必须使用Windows操作系统，但是项目实际的测试、上线环境都是基于Linux的，所以每次只能在本地编写某一功能的代码后通过“跳板机”将项目代码传送到服务器上进行测试。出现问题也只能在服务器上修改，运行通过后再将修改后的代码传送到本地然后继续下一阶段的工作。由于我自己不太喜欢直接在服务器上使用vim编辑代码，也受不了每次都要通过"跳板机"将代码传来传去的繁复操作，本着生命在于折腾的真理，开启了这次的折腾之旅。

为了既可以愉快地在Windows下编码、交流、玩耍，又可以一键在Linux下快速部署、测试代码，本文通过Docker for Windows在Windows系统中布置与服务器上一致的Linux测试环境，不用再将我们的代码拷贝到远程服务器上，只占用较小的资源就可以一键在本地以相同的运行环境测试我们的代码。

### 效果演示

1. 一键运行项目：

![img](借助Docker，在win10下编码，一键在Linux下测试.assets/16abffa1c283d641)

1. 模拟修改项目后的一键重新部署、测试：

![img](借助Docker，在win10下编码，一键在Linux下测试.assets/16abffa48cfa2d5e)

### 1. Docker for Windows的安装与设置

> 下载地址：[Download Docker Desktop](https://link.juejin.im/?target=https%3A%2F%2Fwww.docker.com%2Fproducts%2Fdocker-hub)
>
> 安装前提：64bit Windows 10 Pro and Microsoft Hyper-V

#### 1.1 安装步骤：

1. 开启Hyper-V功能：
   - 控制面板 -> 程序和功能 -> 启动或关闭Windows功能 -> 勾选"Hype-V" -> 重启
2. 安装Docker；
3. 设置国内镜像仓库地址：
   - 启动Docker后右击Docker的运行图标选择"setting"进入设置界面；
   - 选择设置面板左侧的"Daemon"选项，在"Registry mirrors"中填入国内的镜像仓库地址（推荐使用阿里云的，在我这边测试的速度较快，但是可能需要注册账号）；
   - 点击"Apply"；
4. 设置共享目录：
   - 进入Docker的设置面板，选择"Shared Drives"选项；
   - 勾选想要共享的盘符，点击"Apply"，可能需要输入Windows账户的密码；

#### 1.2 可能遇到的问题：

在Docker的安装和设置过程中，最可能遇到问题的地方可能是“设置共享目录”的时候，可能会出现勾选了盘符并点击"Apply"后选项又被取消的问题。由于挂载Windows的盘符失败会在加入"-v"选项启动Docker容器时会出现卡死的情况。查看Docker的日志文件(C:\ProgramData\DockerDesktop\service.txt)会发现如下错误：

```
[Error] Unable to mount C drive: C:\Program Files\Docker\Docker\Resources\bin\docker.exe: Error response from daemon: OCI runtime create failed: container_linux.go:344: starting container process caused "exec: \"/usr/bin/nsenter1\": stat /usr/bin/nsenter1: no such file or directory": unknown.
```

​	网络上关于该问题的解决方案主要分为以下几种：

- [修改本地安全策略](https://juejin.im/post/5af985e5f265da0b9265b695)
- [新建用户](https://link.juejin.im/?target=https%3A%2F%2Fblogs.msdn.microsoft.com%2Fstevelasker%2F2016%2F06%2F14%2Fconfiguring-docker-for-windows-volumes%2F)
- [安装其他版本或重装Docker](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fdocker%2Ffor-win%2Fissues%2F690)

 我尝试了第1、3个方法，第1个方法对我的Docker不起作用，然后使用第3中方法重新安装后就可以了，比较奇怪。

------

### 2. 正式开始

#### 2.1 创建工作目录

首先需要设置Windows与docker的Linux测试环境的共享目录为：`D:\pc_share\`，而该目录下的目录树如下所示：

```
.
|-- apps					# 放置我们的项目
|   |-- test_demo1				# 示例项目1
|   |   |-- app.log				
|   |   `-- app.py				
|   `-- test_demo2				# 示例项目2
|       |-- app.log
|       `-- app.py				
`-- etc						# 放置我们的配置文件
    |-- DockerFile			        # 镜像的配置文件，用于构建docker镜像
    |-- requirements.txt		        # 用于记录、安装python项目的依赖库
    `-- supervisor				# 放置示例项目的运行配置文件	
        |-- test_demo1.conf	        # 记录了示例项目1的启动信息，通过软件supervisor启动、监控
        `-- test_demo2.conf		# 记录了示例项目2的启动信息，通过软件supervisor启动、监控
```

以上就是我们这次实验的所有文件的目录树示意图，将相关项目都放在"D:\pc_share\apps"下，为了方便演示，在这里创建了两个基于tornado的Web项目`test_demo1`，`test_demo1`，每个项目中只包含了一个启动文件`app.py`：

```
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello, from test_demo_1")

def make_app():
    return tornado.web.Application([
        (r"/", MainHandler),				# 路由
    ])

if __name__ == "__main__":
    app = make_app()
    app.listen(8888)					# 监听8888端口，示例项目2的监听改为另一个端口，否则有一个将由于端口占用无法成功启动
    tornado.ioloop.IOLoop.current().start()		# 开启Web服务
```

如上所示是一段十分简单的Web程序，该程序监听'8888'端口，如果访问"http:\\localhost:8888"就可以得到回复：**"Hello, from test_demo_1"**。

#### 2.2 使用Docker创建程序运行环境

由章节2.1可知，运行上述项目至少需要三个环境：`操作系统`、`python`、`tornado框架`。为了与项目上线的环境保持一致，这里假设三个环境条件分别为：`Centos7`、`python2.7`、`tornado5.1.1`。所以我们需要使用Dockerfile配置相应的运行环境，不熟悉Dockerfile的同学可以浏览下教程：[Dockerfile教程](https://link.juejin.im/?target=https%3A%2F%2Fyeasy.gitbooks.io%2Fdocker_practice%2Fcontent%2Fimage%2Fbuild.html)。

为了更方便地修改运行环境，Dockerfile主要分为两部分：`配置下层运行环境`、`配置上层运行环境`。

##### 2.2.1 配置下层运行环境

![img](https://user-gold-cdn.xitu.io/2019/5/16/16abff67c8eacf22?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如上图所示为底层运行环境的配置过程示意图，对应的Dockerfile的部分为：

```dockerfile
FROM centos:7 as centos_python2				# 基于镜像："centos:7"
MAINTAINER mileskong <xiangqian_kong@126.com>		# 作者信息

# 环境变量，反斜杠表示换行
ENV PYPI_REPO=https://mirrors.aliyun.com/pypi/simple/ \
	PYTHONIOENCODING=UTF-8 \
	SHARE_PATH=/mnt/share				# docker容器的挂载点，对应Windows中的"D:\pc_share\"路径（启动容器时可以设置共享路径与容器中的挂载点）

# 为Centos7安装必要的软件，这里只安装了三个常用的软件，可以按需修改
RUN set -ex \
	&& yum -y install epel-release wget net-tools \
	&& yum clean all				# 清空缓存，精简镜像

# 安装pip软件（这里省略了python2.7的安装，因为Centos7中自带了python2.7.5，如需安装其他版本的同学在此之前加入python的安装过程）
ENV PIP_VERSION 19.1.1
RUN set -ex; \
	\
	wget -O get-pip.py 'https://bootstrap.pypa.io/get-pip.py'; \
	\
	python get-pip.py \
		--disable-pip-version-check \
		--no-cache-dir \
		"pip==$PIP_VERSION" \
	; \
	pip --version; \
	\
	find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' +; \
	rm -f get-pip.py
```

通过以上Dockerfile中的内容，我们已经完成了一个镜像的配置，该镜像中主要包含了：`Centos7`、`python2.7`、`pip`。

将以上内容保存为：`D:\pc_share\etc\DockerFile`，然后在Windows的命令行软件`cmd`中键入如下命令通过Dockerfile构建镜像centos_python2:1.0（centos_python2为镜像名，1.0为tag，也即版本号）：

```bash
cd D:\pc_share\etc			# 进入配置文件的目录

# docker的镜像构建命令，
#   -t [镜像名:tag]：设置镜像名与tag；
#	-f [文件名]：指定Dockerfile；
#	--target=[阶段名]：用于多阶段构建；
docker build -t centos_python2:1.0 -f DockerFile --target=centos_python2 .
```

通过上述命令可以构建出包含`Centos7`、`python2.7`、`pip`的镜像：`centos_python2:1.0`。在`cmd`中键入`docker images`命令可以得到如下信息：

![img](https://user-gold-cdn.xitu.io/2019/5/16/16abff6f2a435b45?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如上图中红框中即为对应的镜像，其大小仅为237Mb，如果使用`alpine`代替`Centos7`甚至可以将镜像控制在30Mb以内。

##### 2.2.2 配置上层运行环境

![img](借助Docker，在win10下编码，一键在Linux下测试.assets/16abff777a285bef)

下层运行环境提供了一个较为通用的基于Centos的python开发平台，是所有python项目运行的通用基础，在后续的开发、测试过程中基本不需要修改。但是，不同的python项目可能会面临着不同的问题，主要可以分为以下几项：

- 依赖不同的第三方库；
- 不同的项目的启动参数及启动方式不相同；
- 同一个项目的配置文件可能会经常修改；

由于docker镜像是只读的并且构建过程较为耗时，在开发过程中我们会经常修改项目文件、配置文件、依赖库等信息。我们不可能每修改一次就重新构建一个新的镜像。为了解决这一问题，我们将经常变动的过程放置到另一个镜像中，也即`上层运行环境`。

为了解决上述第一个问题，我们在开发python项目时常用`requestments.txt`文件记录并控制项目所依赖python第三方库，通过以下命令安装`requestments.txt`中记录的第三方库：

```
pip install -r /&{path}/requirements.txt
```

本文示例项目的第三方库配置文件放置在：`D:\pc_share\etc\requestments.txt`：

```
tornado==5.1.1				# 依赖的Web框架，版本号为5.1.1
supervisor==4.0.2			# 使用supervisor监管python项目的运行
```

当项目依赖的第三方库发生变化时直接修改`requestments.txt`文件并基于`下层运行环境`重新构建`上层运行环境`的镜像就可以了，减少了不必要的操作。

为了解决第二、三个问题，这里使用了`supervisor`管理python项目，关于其详细的教程可以参见[“淡水网志”的博客：supervisor使用教程](https://link.juejin.im/?target=https%3A%2F%2Fwww.restran.net%2F2015%2F10%2F04%2Fsupervisord-tutorial%2F)。按照`supervisor`的格式为python项目编写启动配置文件并将其放置在`supervisor`监控的文件夹中，`supervisor`就可以按照我们的配置文件很方便地对项目进行启动、监控。例如示例项目1的启动配置文件放在：`D:\pc_share\etc\supervisor\test_demo1.conf`：

```
[program:test_demo1]							# 项目名称，唯一
directory=/mnt/share/apps/test_demo1					# 项目所在路径，因为是在docker的Linux中执行，所以需要填项目在Linux中的路径，这里假设将Windows中的共享路径挂载到"/mnt/share/"下
command=python app.py							# 项目的启动命令
stdout_logfile=/mnt/share/apps/test_demo1/app.log		        # 项目的日志文件
priority=1								# 优先级
numprocs=1								# 进程数
autostart=true								# 自动启动
autorestart=true							# 自动重启
startretries=5								# 启动尝试次数
```

以上就是示例项目1的启动配置文件，修改`supervisor`的监控路径为`/mnt/share/etc/supervisor/`，因为docker可以将Windows的共享目录`D:\pc_share\`挂载到`/mnt/share/`下，所以当`supervisor`启动时就会在`/mnt/share/etc/supervisor/`（也即Windows中的`D:\pc_share\etc\supervisor`）下寻找项目的配置文件，并按照项目的配置文件启动对应的项目。

```
FROM centos_python2:1.0 as centos_python2_supervisor		# 上层运行环境的镜像基于下层运行环境的镜像："centos_python2:1.0"

ENV SUPERVISOR_PATH=$SHARE_PATH/etc/supervisor		        # 放置项目的启动配置文件的路径，下面会配置supervisor的设置文件将其读取配置文件的目录改为"$SUPERVISOR_PATH"所指向的目录

# 通过pip按照requirements.txt记录的依赖库列表安装python的第三方库
COPY requirements.txt /tmp/					# 将 D:\pc_share\etc\ 目录下的requirements.txt复制到镜像的/tmp/中
RUN set -ex \
	&& pip install -r /tmp/requirements.txt -i $PYPI_REPO \
	&& rm -rf ~/.cache/pip/*

# 配置supervisor的设置文件"/etc/supervisord.conf"
RUN	set -ex \
	&& echo_supervisord_conf > /etc/supervisord.conf \
	&& mkdir /etc/supervisord.d/ \
	&& echo "[include]" >> /etc/supervisord.conf \
	&& echo "files = $SUPERVISOR_PATH/*.conf" >> /etc/supervisord.conf \	# 监控"$SUPERVISOR_PATH"目录中的项目启动配置文件
	&& sed -i '/nodaemon/s/false/true/' /etc/supervisord.conf		# 【十分重要】将supervisord.conf中的nodaemon变量改由false改为true，也即将supervisor改为前台运行，后面作详细解释

EXPOSE 8888 8889											# 声明暴露的端口

CMD ["supervisord", "-c", "/etc/supervisord.conf"]		# 容器（不是镜像）启动时执行的命令
```

如上所示为`上层运行环境`的镜像配置文件。首先，该镜像基于刚刚构建的`下层运行环境`镜像：`centos_python2:1.0`，在此基础上，使用pip工具依照`requirements.txt`中记录的依赖项为项目安装运行时必要的python第三方库。然后，生成、配置`supervisor`的设置文件`/etc/supervisord.conf`，将`supervisor`读取项目的启动配置文件的目录改为`$SUPERVISOR_PATH/`，其中`$SUPERVISOR_PATH`为前文设置的环境变量，记录了Windows中项目的启动配置文件在docker镜像中的挂载点。则通过`$SUPERVISOR_PATH`读取的配置文件实际就是Windows中`D:\pc_share\etc\supervisor\`下的配置文件。

最后，`CMD`后的内容表示容器启动时会被执行的语句，通常可以用来启动项目。由于我们使用了`supervisor`来监控项目，所以文件中直接开启了`supervisor`的服务。

`supervisor`的设置过程中有一行命令为`sed -i '/nodaemon/s/false/true/' /etc/supervisord.conf`，这是我在不了解docker运行机制的情况下踩过的坑。Docker镜像使用run命令运行起来后就会产生一个容器，可以简单地把容器当作一个进程，进程在执行完它的指令后就会退出、停止。所以当我们的指令为启动`supervisor`时，而`supervisor`默认是后台运行的，所以我们的容器启动时执行命令启动`supervisor`：`supervisord -c /etc/supervisord.conf`，这是`supervisor`进入了后台运行，该命令执行结束，docker容器也认为它完成了自己的任务，顺利退出、停止运行。造成的结果就是每次启动docker容器都会立刻停止运行，无法使容器按我们的想法持续的运行。为了解决这个问题，我们可以将`supervisor`改为在前台运行，也即修改设置文件`/etc/supervisord.conf`中的"nodaemon"为"true"。这时再启动容器，`supervisor`会一直占据着前台，使容器不能"完成"自己的指令，我们的容器就可以保持着持续的运行状态。

#### 2.3 构建镜像并运行

通过2.2节我们完成了docker镜像的配置文件Dockerfile的编写，在这一份Dockerfile中，为了方便修改、加速镜像的构建过程，我们将其分为两个部分，分别对应着下层运行环境镜像与上层运行环境镜像。为了区分，根据docker镜像的命名规范，我们将前者命名为：centos_python2:1.0，后者命名为：centos_python2_supervisor:1.0。

综上所述，整个Dockerfile的内容为（D:\pc_share\etc\DockerFile）：

```
FROM centos:7 as centos_python2
MAINTAINER mileskong <xiangqian_kong@126.com>

ENV PYPI_REPO=https://mirrors.aliyun.com/pypi/simple/ \
	PYTHONIOENCODING=UTF-8 \
	SHARE_PATH=/mnt/share

# install software
RUN set -ex \
	&& yum -y install epel-release wget net-tools \
	&& yum clean all

# install pip
ENV PIP_VERSION 19.1.1
RUN set -ex; \
	\
	wget -O get-pip.py 'https://bootstrap.pypa.io/get-pip.py'; \
	\
	python get-pip.py \
		--disable-pip-version-check \
		--no-cache-dir \
		"pip==$PIP_VERSION" \
	; \
	pip --version; \
	\
	find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' +; \
	rm -f get-pip.py


FROM centos_python2:1.0 as centos_python2_supervisor

ENV SUPERVISOR_PATH=$SHARE_PATH/etc/supervisor

# install python libs by pip
COPY requirements.txt /tmp/
RUN set -ex \
	&& pip install -r /tmp/requirements.txt -i $PYPI_REPO \
	&& rm -rf ~/.cache/pip/*

# build supervisord.conf 
RUN	set -ex \
	&& echo_supervisord_conf > /etc/supervisord.conf \
	&& mkdir /etc/supervisord.d/ \
	&& echo "[include]" >> /etc/supervisord.conf \
	&& echo "files = $SUPERVISOR_PATH/*.conf" >> /etc/supervisord.conf \
	&& sed -i '/nodaemon/s/false/true/' /etc/supervisord.conf

EXPOSE 8888 8889

CMD ["supervisord", "-c", "/etc/supervisord.conf"]
```

下面则是根据Dockerfile构建镜像并运行的步骤：

```
cd D:\pc_share\etc			# 进入配置文件的目录
docker build -t centos_python2:1.0 -f DockerFile --target=centos_python2 .			        # 构建下层运行环境镜像："centos_python2:1.0"
docker build -t centos_python2_supervisor:1.0 -f DockerFile --target=centos_python2_supervisor .	# 基于"centos_python2:1.0"，构建上层运行环境镜像："centos_python2_supervisor:1.0"
docker run -itd -v d:/pc_share/:/mnt/share -p 8888:8888 -p 8889:8889 ${image_id}			# 运行镜像"centos_python2:1.0"
```

上述命令首先构建了下层运行环境的镜像`centos_python2:1.0`，然后在此基础上构建了上层运行环境的镜像`centos_python2_supervisor:1.0`，最后通过docker的run命令启动镜像。其中比较重要的是run命令中的一些参数，下面作详细介绍：

```
docker run [options] [image_id]
# options:
	-v [宿主机的共享目录]:[docker容器中的挂载点]：将宿主机中的共享目录挂载到docker容器的某个挂载点下，如果挂载点不存在则启动容器时自动创建；
	-p [宿主机端口]:[docker容器端口]：将宿主机的端口映射到docker容器的端口上；
```

根据run的参数可以看出，我们在运行镜像时将Windows系统下的目录：`d:/pc_share/`挂载到docker容器中的`/mnt/share`下；将宿主机的8888端口映射到docker容器的8888端口，将8889映射到docker容器的8889端口。

通过上述步骤启动镜像`centos_python2_supervisor:1.0`后，通过命令`docker ps -a`可以查看通过该镜像开启的容器：

![img](https://user-gold-cdn.xitu.io/2019/5/16/16abff8592e82baa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

当我们在本地浏览器中分别输入"localhost:8888"和"localhost:8889"后就可以访问运行在docker中的示例项目了。

### 3. 一键启动

在第2章节中详细介绍了怎么使用docker自定义项目的运行环境，整个过程基本可以划分为以下几部分：

1. 编写Dockerfile，自定义项目的运行环境和启动配置；
2. 通过Dockerfile构建docker镜像；
3. 通过run命令启动docker，生成容器。

在开发过程中，前两个步骤是配置环境的过程，而第三个步骤是我们经常重复进行的，每当我们在Windows中修改了项目代码或者配置文件后，都需要重启docker容器验证代码的运行情况，而run命令的参数也较为复杂，所以频繁的重启容器也可能是比较烦人的。

由于我平常比较喜欢使用`vscode`作为开发的IDE工具，偶尔中发现了`vscode`也支持docker插件，十分方便，下面简要为大家介绍下如何通过`vscode`的docker插件实现“一键启动”。

#### 3.1 docker插件的使用方法

1. 在`vscode`的扩展插件商店中搜索"docker"并安装该插件，重启后会发现`vscode`的侧边栏多出了一个docker的图标：

![img](借助Docker，在win10下编码，一键在Linux下测试.assets/16abff94c362363f)

如上图中所示，这个插件可以直接显示出本机中存在的镜像、容器和仓库信息。由图可以看到我们之前创建的所有镜像。

1. 选择我们的镜像`centos_python2_supervisor:1.0`，右击并在选项列表中选择`Run`，会发现我们的容器列表中新增了一个运行的容器：

![img](借助Docker，在win10下编码，一键在Linux下测试.assets/16abff97aa5a3d49)

1. 如果我们修改了代码后需要重新测试的话，选择容器右击选择`Restart Container`，就可以重启我们的容器，重新启动我们的项目：

![img](借助Docker，在win10下编码，一键在Linux下测试.assets/16abff9af13c87dc)

所以，当我们通过编写Dockerfile、构建镜像后，基本的运行环境就搭建成功了。之后每次需要测试修改后的代码时只需要在docker的插件界面中选中对应容器选择重启就可以了。如果测试想要了解项目运行更详细的信息，右击选择`Attach Shell`就可以进入容器中通过熟悉的终端查看相关日志：

![img](借助Docker，在win10下编码，一键在Linux下测试.assets/16abff9e50c7a086)


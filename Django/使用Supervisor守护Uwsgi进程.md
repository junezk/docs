## 使用Supervisor守护Uwsgi进程

## 1.了解Supervisor

Supervisor是一款运行在类Unix系统（如Linux、MacOS系统）上的进程管理软件，基于Python开发。通过它我们可以很方便的管理服务器上的各种程序的进程，包括今天要讲的用supervisor管理uwsgi进程。有了它，我们再也不用担心由于uwsgi进程莫名被"死掉"而引起的Django网站打不开的情况了。

由于我也是第一次用它，所以折腾了老半天才搞好，这期间遇到了各种报错，于是就各种Google，汇总了有关Supervisor的各种操作，以及一些常见的错误，现将Supervisor的安装配置方法，以及管理命令总结如下，方便日后查看。

## 2.安装Supervisor

由于Supervisor是基于Python开发，那么最简便的安装方法就是通过pip安装。截止目前，pypi上最新的supervisor版本是2019年4月18号发布的4.0.2版本。现在网上大多数的非官方supervisor教程是在它不支持python3的时候发布的，因此大多都提到了安装supervisor时的python版本问题。

但是4.0以后的版本都开始支持python3了，更确切的说是支持python3.4以及更高的版本。对于python2，supervisor目前摒弃了对python2.4、2.5、2.6支持，仅支持python2.7了。以下是英文原话：


>Support for Python 3 has been added. On Python 3, Supervisor requires Python 3.4 or later.
>Support for Python 2.4, 2.5, and 2.6 has been dropped. On Python 2, Supervisor now requires Python 2.7.


好了，直接pip安装

```
pip install supervisor
```

## 3.配置Supervisor

### 3.1生成配置文件

安装完成之后我们可以使用下面的命令来查看supervisor的一个默认配置模板：

```
echo_supervisord_conf
```

然而上面命令输出的结果仅是一个模板，我们需要使用下面的命令在`/etc`目录下生成配置文件：

```
echo_supervisord_conf > /etc/supervisord.conf
```

之后我们可以使用`vim`查看该文件的内容

```
vim /etc/supervisord.conf
```

### 3.2修改配置文件

我们可以看到，该配置文件内容一大堆，但是我们目前仅需要知道并改文件末尾的这个部分就可以了，这是默认写法

```
;[include]
;files = relative/directory/*.ini
```

分号`;`后面的是注释内容，我们首先把它去掉，然后将files的值改为：

```
[include]
files = /etc/supervisor/*.conf
```

`include`可以理解为包含，它的意思是将`/etc/supervisor/`目录下的，所有以`.conf`结尾的文件包含进来，`/etc/`下的`supervisor/`文件夹默认是不存在的，我们需要手动创建：

```
cd /etc
mkdir supervisor
```

以`.conf`结尾的文件文件也是默认不存的，也需要我们根据需求自己创建，一个conf文件代表的是一个程序，假如我需要supervisor管理uwsgi程序，那么就在`/etc/supervisor/`目录下创建一个`uwsgi.conf`文件，在里面写入配置信息，如果需要管理其他的xx程序，就重新创建一个`xx.conf`文件。

好，现在我们就创建一个`uwsgi.conf`文件:

```
vim /etc/supervisor/uwsgi.conf
```

将下面内容复制到文件中

```
[program:uwsgi]
command=/usr/local/bin/uwsgi --ini /home/mysite_uwsgi/mysite.ini
user=root
autorestart=true
autostart=true
startretries=3
redirect_stderr=true
startsecs=5
stdout_logfile=/var/log/django/supervisor.log
stopasgroup=true
killasgroup=true
priority=999
```

这里几个命令我注意说明一下：

- command：需要托管给supervisor执行的命令，这里是uwsgi的启动命令，这里需要按自己的情况更改
- user：执行命令的用户，这里填root，你可以填其他的，只要有权限即可
- autorestart：uwsgi关闭后是否自动重启
- autostart：是否随supervisor启动而启动
- startretries：启动失败自动重试次数，默认是 3
- redirect_stderr：把 stderr 重定向到 stdout，默认 false
- startsecs：启动 5 秒后没有异常退出，就当作已经正常启动了
- stdout_logfile：日志文件存放目录
- stopasgroup：是否干掉程序的所有进程(包括子进程)
- killasgroup：作用同上
- priority：程序启动优先级，若有多个进程管理时使用，默认-1

好了，目前来说，只配置这几个参数完全够用了。更多配置命令详解，我们在最下面给出。现在保存，退出。

## 4.启动Supervisor

在启动之前我们还需要完成两件事情：

### 4.1修改uwsgi配置

之前我们在uwsgi的配置文件中设置了一个日志文件保存位置的参数：

```
daemonize = /home/mysite_uwsgi/mysite.log
```

但是这个参数的配置与Supervisor的日志输出配置是有冲突的，需要注释掉它，因为在前面加个`#`：

```
#daemonize = /home/mysite_uwsgi/mysite.log
```

### 4.2干掉uwsgi进程

在使用Supervisor启动uwsgi之前，我们需要先把uwsgi正在运行的进程都干掉，如果你之前没有运行uwsgi,这一步可以跳过：

```
ps -aux | grep uwsgi |awk '{print $2}'|xargs kill -9
```

### 4.3启动Supervisor

```
supervisord -c /etc/supervisord.conf
```

### 4.3查看运行状态

上一步执行成功之后，如果没有报错那么就是启动成功了，然后我们执行下面的命令查看uwsgi启动状态

```
supervisorctl status
```

如果显示：

```
uwsgi  RUNNING   pid 20986, uptime 0:31:33
```

那么代表的就是启动成功，可以看到这个结果有四列，uwsgi代表的是进程名称，RUNNING代表的进程启动状态，pid代表的是进程id，uptime代表的是进程运行时间。

## 5.管理Supervisor

启动成功之后，我们还需要对Supervisor进程管理。其实supervisor包含两个东西，一个是服务端(Server)，我们使用的命令是supervisord；一个是客户端(Client)，我们使用的命令是supervisorctl。服务端面向程序，负责进程的运行，客户端面向用户，负责进程的管理。现在我们要对supervisor进行管理，因此需要用supervisorctl命令，下面是常用的命令：

```
#查看所有进程的运行状态
supervisorctl status

#查看某一进程的运行状态
supervisorctl status　进程名

#启动某一进程
supervisorctl start 进程名

#启动所有进程
supervisorctl start all

#停止某一进程
supervisorctl stop 进程名

#停止所有进程
supervisorctl stop all

#重启某一进程
supervisorctl restart 进程名

#重启所有进程
supervisorctl restart all

#新增进程后，更新进程(不影响其他进程运行)
supervisorctl update

#新增进程后，重启所有进程
supervisorctl reload
```

## 6.测试Supervisor

我们如何测试Supervisor会守护uwsgi进程的功能呢？这里我给大家简单做个测试，我们手动将所有uwsgi关掉

```
ps -aux | grep uwsgi |awk '{print $2}'|xargs kill -9
```

在没有使用Supervisor之前，现在网站应该访问不了了，因为所有的uwsgi进程都已经被干掉了，但是我们发现我们的网站依然能够正常运行。这是因为Supervisor检测到uwsgi进程关闭了，然后把它重启了。

如果我们在关掉uwsgi进程之后，紧接着用`supervisorctl status`查看进程运次状态，你会得到如下结果：

```
uwsgi      STARTING
```

这说明uwsgi正在被Supervisor启动，稍后我们再次查看，可得到如下结果：

```
uwsgi      RUNNING   pid 24083, uptime 0:04:56
```

此时uwsgi进程已经被重启成功了。怎么样，NB吧！

## 7.常见问题

### 7.1已有Supervisor在运行

如果你在启动Supervisor的时候遇到下面的问题，说明已经有Supervisor程序在运行了

```
Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.
For help, use /home/django/bin/supervisord -h
```

解决办法就是干掉他们：

```
sudo pkill supervisord
```

### 7.2找不到进程

如果你遇到了下面的错误

```
ERROR (no such process)
```

那就是进程名称写错了。

### 7.3进程运行出错

如果遇到下面的进程状态

```
uwsgi     FATAL     Exited too quickly (process log may have details)
```

请可能是没有进行4.1中所提到的修改uwsgi的配置的操作。

### 7.4找不到sock

如果在使用supervisorctl 的时候遇到了下面的错误提示，那说明你掉进了Supervisor官方给你挖的一个坑。

```
unix:///tmp/supervisor.sock no such file
```

打开配置文件

```
vim /etc/supervisord.conf
```

使vim显示行号：

```
:set number
```

我们可以看到，在第16、28、32、57行中:supervisord.pid 以及 supervisor.sock 是放在 /tmp 目录下，但是 /tmp 目录是存放临时文件，里面的文件是会被 Linux 系统删除的，一旦这些文件丢失，就无法再通过 supervisorctl 来执行 restart 和 stop 命令了，将只会得到 unix:///tmp/supervisor.sock 不存在的错误 。解决办法如下：

1. 修改配置文件的内容，这里把所有的 `/tmp` 路径改掉，需要改的地方有第16、28、32、57行：

```
/tmp/supervisor.sock` 改成 `/var/run/supervisor.sock`， `/tmp/supervisord.log` 改成 `/var/log/supervisor.log`， `/tmp/supervisord.pid` 改成 `/var/run/supervisor.pid
```

1. 修改权限

```
sudo chmod 777 /var/run  
sudo chmod 777 /var/log
```

如果没改，启动报如下错

```
IOError: [Errno 13] Permission denied: '/var/log/supervisord.log'
```

1. 创建 `supervisor.sock`

```
sudo touch /var/run/supervisor.sock
sudo chmod 777 /var/run/supervisor.sock
```

1. 停止所有进程

```
supervisorctl stop all
```

1. 停止Supervisord

```
sudo pkill supervisord
```

1. 启动Supervisord

```
supervisord -c /etc/supervisord.conf
```

### 7.5Unlinking错误

如果遇到了如下错误提醒：

```
Unlinking stale socket /var/run/supervisor.sock
```

执行

```
sudo unlink /var/run/supervisor.sock
```

## 8.常用命令

```
#查看所有进程ID
ps -aux | grep supervisord

#杀掉全部进程
sudo pkill supervisord

# 开启supervisord
supervisord -c /etc/supervisord.conf
```

## 9.参考资料

- [Supervisor官方文档](http://supervisord.org/)
- [supervisor 管理进程简明教程](https://www.jianshu.com/p/bf2b3f4dec73)
- [解决unix:///tmp/supervisor.sock no such file的问题](https://zanjs.com/2018/04/09/supervisord/解决unix-tmp-supervisor-sock-no-such-file的问题/)

## 10.配置文件详解

```ini
[unix_http_server]            
file=/tmp/supervisor.sock   ; socket文件的路径，supervisorctl用XML_RPC和supervisord通信就是通过它进行
                              的。如果不设置的话，supervisorctl也就不能用了  
                              不设置的话，默认为none。 非必须设置        
;chmod=0700                 ; 这个简单，就是修改上面的那个socket文件的权限为0700
                              不设置的话，默认为0700。 非必须设置
;chown=nobody:nogroup       ; 这个一样，修改上面的那个socket文件的属组为user.group
                              不设置的话，默认为启动supervisord进程的用户及属组。非必须设置
;username=user              ; 使用supervisorctl连接的时候，认证的用户
                               不设置的话，默认为不需要用户。 非必须设置
;password=123               ; 和上面的用户名对应的密码，可以直接使用明码，也可以使用SHA加密
                              如：{SHA}82ab876d1387bfafe46cc1c8a2ef074eae50cb1d
                              默认不设置。。。非必须设置

;[inet_http_server]         ; 侦听在TCP上的socket，Web Server和远程的supervisorctl都要用到他
                              不设置的话，默认为不开启。非必须设置
;port=127.0.0.1:9001        ; 这个是侦听的IP和端口，侦听所有IP用 :9001或*:9001。
                              这个必须设置，只要上面的[inet_http_server]开启了，就必须设置它
;username=user              ; 这个和上面的uinx_http_server一个样。非必须设置
;password=123               ; 这个也一个样。非必须设置

[supervisord]                ;这个主要是定义supervisord这个服务端进程的一些参数的
                              这个必须设置，不设置，supervisor就不用干活了
logfile=/tmp/supervisord.log ; 这个是supervisord这个主进程的日志路径，注意和子进程的日志不搭嘎。
                               默认路径$CWD/supervisord.log，$CWD是当前目录。。非必须设置
logfile_maxbytes=50MB        ; 这个是上面那个日志文件的最大的大小，当超过50M的时候，会生成一个新的日 
                               志文件。当设置为0时，表示不限制文件大小
                               默认值是50M，非必须设置。              
logfile_backups=10           ; 日志文件保持的数量，上面的日志文件大于50M时，就会生成一个新文件。文件
                               数量大于10时，最初的老文件被新文件覆盖，文件数量将保持为10
                               当设置为0时，表示不限制文件的数量。
                               默认情况下为10。。。非必须设置
loglevel=info                ; 日志级别，有critical, error, warn, info, debug, trace, or blather等
                               默认为info。。。非必须设置项
pidfile=/tmp/supervisord.pid ; supervisord的pid文件路径。
                               默认为$CWD/supervisord.pid。。。非必须设置
nodaemon=false               ; 如果是true，supervisord进程将在前台运行
                               默认为false，也就是后台以守护进程运行。。。非必须设置
minfds=1024                  ; 这个是最少系统空闲的文件描述符，低于这个值supervisor将不会启动。
                               系统的文件描述符在这里设置cat /proc/sys/fs/file-max
                               默认情况下为1024。。。非必须设置
minprocs=200                 ; 最小可用的进程描述符，低于这个值supervisor也将不会正常启动。
                              ulimit  -u这个命令，可以查看linux下面用户的最大进程数
                              默认为200。。。非必须设置
;umask=022                   ; 进程创建文件的掩码
                               默认为022。。非必须设置项
;user=chrism                 ; 这个参数可以设置一个非root用户，当我们以root用户启动supervisord之后。
                               我这里面设置的这个用户，也可以对supervisord进行管理
                               默认情况是不设置。。。非必须设置项
;identifier=supervisor       ; 这个参数是supervisord的标识符，主要是给XML_RPC用的。当你有多个
                               supervisor的时候，而且想调用XML_RPC统一管理，就需要为每个
                               supervisor设置不同的标识符了
                               默认是supervisord。。。非必需设置
;directory=/tmp              ; 这个参数是当supervisord作为守护进程运行的时候，设置这个参数的话，启动
                               supervisord进程之前，会先切换到这个目录
                               默认不设置。。。非必须设置
;nocleanup=true              ; 这个参数当为false的时候，会在supervisord进程启动的时候，把以前子进程
                               产生的日志文件(路径为AUTO的情况下)清除掉。有时候咱们想要看历史日志，当 
                               然不想日志被清除了。所以可以设置为true
                               默认是false，有调试需求的同学可以设置为true。。。非必须设置
;childlogdir=/tmp            ; 当子进程日志路径为AUTO的时候，子进程日志文件的存放路径。
                               默认路径是这个东西，执行下面的这个命令看看就OK了，处理的东西就默认路径
                               python -c "import tempfile;print tempfile.gettempdir()"
                               非必须设置
;environment=KEY="value"     ; 这个是用来设置环境变量的，supervisord在linux中启动默认继承了linux的
                               环境变量，在这里可以设置supervisord进程特有的其他环境变量。
                               supervisord启动子进程时，子进程会拷贝父进程的内存空间内容。 所以设置的
                               这些环境变量也会被子进程继承。
                               小例子：environment=name="haha",age="hehe"
                               默认为不设置。。。非必须设置
;strip_ansi=false            ; 这个选项如果设置为true，会清除子进程日志中的所有ANSI 序列。什么是ANSI
                               序列呢？就是我们的\n,\t这些东西。
                               默认为false。。。非必须设置

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]    ;这个选项是给XML_RPC用的，当然你如果想使用supervisord或者web server 这 
                              个选项必须要开启的
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface 

[supervisorctl]              ;这个主要是针对supervisorctl的一些配置  
serverurl=unix:///tmp/supervisor.sock ; 这个是supervisorctl本地连接supervisord的时候，本地UNIX socket
                                        路径，注意这个是和前面的[unix_http_server]对应的
                                        默认值就是unix:///tmp/supervisor.sock。。非必须设置
;serverurl=http://127.0.0.1:9001 ; 这个是supervisorctl远程连接supervisord的时候，用到的TCP socket路径
                                   注意这个和前面的[inet_http_server]对应
                                   默认就是http://127.0.0.1:9001。。。非必须项

;username=chris              ; 用户名
                               默认空。。非必须设置
;password=123                ; 密码
                              默认空。。非必须设置
;prompt=mysupervisor         ; 输入用户名密码时候的提示符
                               默认supervisor。。非必须设置
;history_file=~/.sc_history  ; 这个参数和shell中的history类似，我们可以用上下键来查找前面执行过的命令
                               默认是no file的。。所以我们想要有这种功能，必须指定一个文件。。。非
                               必须设置

; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

;[program:theprogramname]      ;这个就是咱们要管理的子进程了，":"后面的是名字，最好别乱写和实际进程
                                有点关联最好。这样的program我们可以设置一个或多个，一个program就是
                                要被管理的一个进程
;command=/bin/cat              ; 这个就是我们的要启动进程的命令路径了，可以带参数
                                例子：/home/test.py -a 'hehe'
                                有一点需要注意的是，我们的command只能是那种在终端运行的进程，不能是
                                守护进程。这个想想也知道了，比如说command=service httpd start。
                                httpd这个进程被linux的service管理了，我们的supervisor再去启动这个命令
                                这已经不是严格意义的子进程了。
                                这个是个必须设置的项
;process_name=%(program_name)s ; 这个是进程名，如果我们下面的numprocs参数为1的话，就不用管这个参数
                                 了，它默认值%(program_name)s也就是上面的那个program冒号后面的名字，
                                 但是如果numprocs为多个的话，那就不能这么干了。想想也知道，不可能每个
                                 进程都用同一个进程名吧。


;numprocs=1                    ; 启动进程的数目。当不为1时，就是进程池的概念，注意process_name的设置
                                 默认为1    。。非必须设置
;directory=/tmp                ; 进程运行前，会前切换到这个目录
                                 默认不设置。。。非必须设置
;umask=022                     ; 进程掩码，默认none，非必须
;priority=999                  ; 子进程启动关闭优先级，优先级低的，最先启动，关闭的时候最后关闭
                                 默认值为999 。。非必须设置
;autostart=true                ; 如果是true的话，子进程将在supervisord启动后被自动启动
                                 默认就是true   。。非必须设置
;autorestart=unexpected        ; 这个是设置子进程挂掉后自动重启的情况，有三个选项，false,unexpected
                                 和true。如果为false的时候，无论什么情况下，都不会被重新启动，
                                 如果为unexpected，只有当进程的退出码不在下面的exitcodes里面定义的退 
                                 出码的时候，才会被自动重启。当为true的时候，只要子进程挂掉，将会被无
                                 条件的重启
;startsecs=1                   ; 这个选项是子进程启动多少秒之后，此时状态如果是running，则我们认为启
                                 动成功了
                                 默认值为1 。。非必须设置
;startretries=3                ; 当进程启动失败后，最大尝试启动的次数。。当超过3次后，supervisor将把
                                 此进程的状态置为FAIL
                                 默认值为3 。。非必须设置
;exitcodes=0,2                 ; 注意和上面的的autorestart=unexpected对应。。exitcodes里面的定义的
                                 退出码是expected的。
;stopsignal=QUIT               ; 进程停止信号，可以为TERM, HUP, INT, QUIT, KILL, USR1, or USR2等信号
                                  默认为TERM 。。当用设定的信号去干掉进程，退出码会被认为是expected
                                  非必须设置
;stopwaitsecs=10               ; 这个是当我们向子进程发送stopsignal信号后，到系统返回信息
                                 给supervisord，所等待的最大时间。 超过这个时间，supervisord会向该
                                 子进程发送一个强制kill的信号。
                                 默认为10秒。。非必须设置
;stopasgroup=false             ; 这个东西主要用于，supervisord管理的子进程，这个子进程本身还有
                                 子进程。那么我们如果仅仅干掉supervisord的子进程的话，子进程的子进程
                                 有可能会变成孤儿进程。所以咱们可以设置可个选项，把整个该子进程的
                                 整个进程组都干掉。 设置为true的话，一般killasgroup也会被设置为true。
                                 需要注意的是，该选项发送的是stop信号
                                 默认为false。。非必须设置。。
;killasgroup=false             ; 这个和上面的stopasgroup类似，不过发送的是kill信号
;user=chrism                   ; 如果supervisord是root启动，我们在这里设置这个非root用户，可以用来
                                 管理该program
                                 默认不设置。。。非必须设置项
;redirect_stderr=true          ; 如果为true，则stderr的日志会被写入stdout日志文件中
                                 默认为false，非必须设置
;stdout_logfile=/a/path        ; 子进程的stdout的日志路径，可以指定路径，AUTO，none等三个选项。
                                 设置为none的话，将没有日志产生。设置为AUTO的话，将随机找一个地方
                                 生成日志文件，而且当supervisord重新启动的时候，以前的日志文件会被
                                 清空。当 redirect_stderr=true的时候，sterr也会写进这个日志文件
;stdout_logfile_maxbytes=1MB   ; 日志文件最大大小，和[supervisord]中定义的一样。默认为50
;stdout_logfile_backups=10     ; 和[supervisord]定义的一样。默认10
;stdout_capture_maxbytes=1MB   ; 这个东西是设定capture管道的大小，当值不为0的时候，子进程可以从stdout
                                 发送信息，而supervisor可以根据信息，发送相应的event。
                                 默认为0，为0的时候表达关闭管道。。。非必须项
;stdout_events_enabled=false   ; 当设置为ture的时候，当子进程由stdout向文件描述符中写日志的时候，将
                                 触发supervisord发送PROCESS_LOG_STDOUT类型的event
                                 默认为false。。。非必须设置
;stderr_logfile=/a/path        ; 这个东西是设置stderr写的日志路径，当redirect_stderr=true。这个就不用
                                 设置了，设置了也是白搭。因为它会被写入stdout_logfile的同一个文件中
                                 默认为AUTO，也就是随便找个地存，supervisord重启被清空。。非必须设置
;stderr_logfile_maxbytes=1MB   ; 这个出现好几次了，就不重复了
;stderr_logfile_backups=10     ; 这个也是
;stderr_capture_maxbytes=1MB   ; 这个一样，和stdout_capture一样。 默认为0，关闭状态
;stderr_events_enabled=false   ; 这个也是一样，默认为false
;environment=A="1",B="2"       ; 这个是该子进程的环境变量，和别的子进程是不共享的
;serverurl=AUTO                ; 

; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.

;[eventlistener:theeventlistenername] ;这个东西其实和program的地位是一样的，也是suopervisor启动的子进
                                       程，不过它干的活是订阅supervisord发送的event。他的名字就叫
                                       listener了。我们可以在listener里面做一系列处理，比如报警等等
                                       楼主这两天干的活，就是弄的这玩意
;command=/bin/eventlistener    ; 这个和上面的program一样，表示listener的可执行文件的路径
;process_name=%(program_name)s ; 这个也一样，进程名，当下面的numprocs为多个的时候，才需要。否则默认就
                                 OK了
;numprocs=1                    ; 相同的listener启动的个数
;events=EVENT                  ; event事件的类型，也就是说，只有写在这个地方的事件类型。才会被发送


;buffer_size=10                ; 这个是event队列缓存大小，单位不太清楚，楼主猜测应该是个吧。当buffer
                                 超过10的时候，最旧的event将会被清除，并把新的event放进去。
                                 默认值为10。。非必须选项
;directory=/tmp                ; 进程执行前，会切换到这个目录下执行
                                 默认为不切换。。。非必须
;umask=022                     ; 淹没，默认为none，不说了
;priority=-1                   ; 启动优先级，默认-1，也不扯了
;autostart=true                ; 是否随supervisord启动一起启动，默认true
;autorestart=unexpected        ; 是否自动重启，和program一个样，分true,false,unexpected等，注意
                                  unexpected和exitcodes的关系
;startsecs=1                   ; 也是一样，进程启动后跑了几秒钟，才被认定为成功启动，默认1
;startretries=3                ; 失败最大尝试次数，默认3
;exitcodes=0,2                 ; 期望或者说预料中的进程退出码，
;stopsignal=QUIT               ; 干掉进程的信号，默认为TERM，比如设置为QUIT，那么如果QUIT来干这个进程
                                 那么会被认为是正常维护，退出码也被认为是expected中的
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ;设置普通用户，可以用来管理该listener进程。
                                默认为空。。非必须设置
;redirect_stderr=true          ; 为true的话，stderr的log会并入stdout的log里面
                                默认为false。。。非必须设置
;stdout_logfile=/a/path        ; 这个不说了，好几遍了
;stdout_logfile_maxbytes=1MB   ; 这个也是
;stdout_logfile_backups=10     ; 这个也是
;stdout_events_enabled=false   ; 这个其实是错的，listener是不能发送event
;stderr_logfile=/a/path        ; 这个也是
;stderr_logfile_maxbytes=1MB   ; 这个也是
;stderr_logfile_backups        ; 这个不说了
;stderr_events_enabled=false   ; 这个也是错的，listener不能发送event
;environment=A="1",B="2"       ; 这个是该子进程的环境变量
                                 默认为空。。。非必须设置
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.

;[group:thegroupname]  ;这个东西就是给programs分组，划分到组里面的program。我们就不用一个一个去操作了
                         我们可以对组名进行统一的操作。 注意：program被划分到组里面之后，就相当于原来
                         的配置从supervisor的配置文件里消失了。。。supervisor只会对组进行管理，而不再
                         会对组里面的单个program进行管理了
;programs=progname1,progname2  ; 组成员，用逗号分开
                                 这个是个必须的设置项
;priority=999                  ; 优先级，相对于组和组之间说的
                                 默认999。。非必须选项

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

;[include]                         ;这个东西挺有用的，当我们要管理的进程很多的时候，写在一个文件里面
                                    就有点大了。我们可以把配置信息写到多个文件中，然后include过来
;files = relative/directory/*.ini
```
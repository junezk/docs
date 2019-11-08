# 使用 gunicorn 搭建一个高并发的 web 服务

### 

用过 Python web 框架的朋友都知道，Flask 自带的 wsgi 性能低下，不支持高并发。只适合你开发调试的时候用，所以在线上一般都使用 Nginx + gunicorn 才能获得更强的性能和更高的安全性!

gunicorn 是一个 python Wsgi http server，只支持在 Unix 系统上运行，下面我们来熟悉一下以 gunicorn 的配置与使用。

### 一、gunicorn 的安装

注意 gunicorn 不能在 windows 环境下使用

> pip install gunicorn

比如有以下一个 app 的 py 文件

```
from flask import Flask  
  
app = Flask(__name__)  

  
@app.route('/')  
def index():  
    return 'hello world!' 

if __name__ == '__main__':  
    app.run()
```

那么我们在 flask 的项目的目录下如何使用 gunicorn 来启动呢? 命令如下：

```
gunicorn -w 4 -b 0.0.0.0:8000 app:app
```

其中： 第一个 app 指的是 app.py 文件； 第二个指的是 flask 应用的名字，app = Flask(*name*)

### 二、gunicorn 的参数详解

通过 gunicorn -h 我们可以看到 gunicorn 有非常多的配置项，我将他们的参数配置项都列出来了，供大家参考：

```
-c CONFIG  : CONFIG,配置文件的路径，通过配置文件启动；生产环境使用；

-b ADDRESS : ADDRESS，ip加端口，绑定运行的主机；

-w INT,  --workers INT用于处理工作进程的数量，为正整数，默认为1；

-k STRTING, --worker-class STRTING要使用的工作模式，默认为sync异步，可以下载eventlet和gevent并指定
--threads INT：处理请求的工作线程数，使用指定数量的线程运行每个worker。为正整数，默认为1。

--worker-connections INT：最大客户端并发数量，默认情况下这个值为1000。

--backlog int：未决连接的最大数量，即等待服务的客户的数量。默认2048个，一般不修改；

-p FILE, --pid FILE：设置pid文件的文件名，如果不设置将不会创建pid文件

--access-logfile FILE  ： 要写入的访问日志目录

--access-logformat STRING：要写入的访问日志格式

--error-logfile FILE, --log-file FILE ：  要写入错误日志的文件目录。

--log-level LEVEL  ：  错误日志输出等级。

--limit-request-line INT ： HTTP请求头的行数的最大大小，此参数用于限制HTTP请求行的允许大小，默认情况下，这个值为4094。值是0~8190的数字。

--limit-request-fields INT ： 限制HTTP请求中请求头字段的数量。此字段用于限制请求头字段的数量以防止DDOS攻击，默认情况下，这个值为100，这个值不能超过32768

--limit-request-field-size INT ： 限制HTTP请求中请求头的大小，默认情况下这个值为8190字节。值是一个整数或者0，当该值为0时，表示将对请求头大小不做限制

-t INT, --timeout INT：超过这么多秒后工作将被杀掉，并重新启动。一般设定为30秒；

--daemon：是否以守护进程启动，默认false；

--chdir ：在加载应用程序之前切换目录；

--graceful-timeout INT：默认情况下，这个值为30，在超时(从接收到重启信号开始)之后仍然活着的工作将被强行杀死；一般使用默认；

--keep-alive INT：在keep-alive连接上等待请求的秒数，默认情况下值为2。一般设定在1~5秒之间。

--reload：默认为False。此设置用于开发，每当应用程序发生更改时，都会导致工作重新启动。

--spew：打印服务器执行过的每一条语句，默认False。此选择为原子性的，即要么全部打印，要么全部不打印；

--check-config  ：显示现在的配置，默认值为False，即显示。

-e ENV,  --env ENV： 设置环境变量；
```

是不是觉得gunicorn 的参数很多？在部署的时候输入这么大一串命令，就算是对参数很熟悉的人，也有可能会输错！那么，有没有更方便的方法呢？当然，那就是用以配置文件的方式启动！

### 三、以配置文件的方式启动

配置文件的方式启动，一般这个配置文件名是 gunicorn.conf.py。 代码如下:

```
# 并行工作进程数
workers = 4

# 指定每个工作者的线程数
threads = 2

# 端口 5000
bind = '0.0.0.0:5000'

# 设置守护进程,将进程交给supervisor管理
daemon = 'false'

# 工作模式协程
worker_class = 'gevent'

# 设置最大并发量
worker_connections = 2000

# 设置进程文件目录
pidfile = '/var/run/gunicorn.pid'

# 设置访问日志和错误信息日志路径
accesslog = "log/access.log"
errorlog = "log/debug.log"
loglevel = "debug"

# 设置日志记录水平
loglevel = 'warning'
```

代码里面的注释都很详细了，但是还是有几点需要说明的：

#### 1.日志

配置文件中指定了日志文件的输出目录，需要注意的是：

- 需要log目录存在，如果不存在，启动会报错
- accesslog 是访问日志，可以通过 access_log_format 设置访问日志格式
- loglevel 用于控制 errorlog 的信息级别

不过更建议使用 logging 模块来管理日志

#### 2. workers

- worker_class 是指开启的每个工作进程的模式类型，默认为 sync 模式，这个使用 gevent 模式，gevent 是 python 的一个高并发库
- workers 是并行工作进程数 ，在上述配置文件中，取的是服务器的 CPU 的数量。需要注意的是，这个数字不是越大越好，因为我们还要注意部署机器的性能，不能无限制多开。一般是根据服务器的 CPU核心数来确定的！

```
workers = multiprocessing * cpu_count() * 2 + 1
```

配置文件搞定之后，启动命令就很简单了。启动 gunicorn 命令如下

```
gunicorn -c gunicorn.conf app:app
```

感谢您的关注！
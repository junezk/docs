

# Django的生产部署：Nginx + Gunicorn + Supervisor

## 1、新建virtual环境，安装django，安装各种代码包，部署代码。

## 2、在虚拟环境中安装gunicorn：

```shell
pip install gunicorn 
```

安装好了以后，在虚拟环境中使用Gunicorn来测试运行django：

```shell
gunicorn hello.wsgi:application --bind 0.0.0.0:8001
```

上面的命令是简单的测试，检查软件是否正常安装。

在正式的生产环境下使用Gunicorn，还需要增加一些配置。我们把这些配置文件写成一个bash脚本，保存为bin/gunicorn_start

```bash
#!/bin/bash
 
NAME="tk" # Name of the application
PROJECTDIR=/www/tk
DJANGODIR=/www/tk/web-tk # Django project directory
SOCKFILE=/www/tk/run/gunicorn.sock # we will communicte using this unix socket
USER=ubuntu # the user to run as
GROUP=ubuntu # the group to run as
NUM_WORKERS=5 # how many worker processes should Gunicorn spawn
DJANGO_SETTINGS_MODULE=tk.settings # which settings file should Django use
DJANGO_WSGI_MODULE=tk.wsgi # WSGI module name
 
echo "Starting $NAME as `ubuntu`"
 
# Activate the virtual environment
cd $PROJECTDIR
pipenv shell
cd $DJANGODIR
export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
export PYTHONPATH=$DJANGODIR:$PYTHONPATH
 
# Create the run directory if it doesn't exist
RUNDIR=$(dirname $SOCKFILE)
test -d $RUNDIR || mkdir -p $RUNDIR
 
# Start your Django Unicorn
# Programs meant to be run under supervisor should not daemonize themselves (do not use --daemon)
exec gunicorn ${DJANGO_WSGI_MODULE}:application \
--name $NAME \
--workers $NUM_WORKERS \
--user=$USER --group=$GROUP \
--bind=unix:$SOCKFILE \
--log-level=debug \
--log-file=-
```

## 3、安装Supervisor

Superviosr是一个进程监管的工具。简而言之，Superviosr可以保证你的程序在服务器开机时自动启动以及程序意外终止时重新启动。通过下面的命令即可安装：

```
sudo aptitude install supervisor
```

Superviosr通过配置文件来设置被监管的程序。一般配置文件都放置在`/etc/supervisor/conf.d`路径下面。此处我们创建一个名为`yourapp.conf`的配置文件，内容如下：

```
[program:tk]
command = /www/tk/bin/gunicorn_start ; Command to start app
user = ubuntu ; User to run as
stdout_logfile = /www/tk/logs/gunicorn_supervisor.log ; Where to write log messages
redirect_stderr = true ; Save stderr in the same log
environment=LANG=en_US.UTF-8,LC_ALL=en_US.UTF-8 ; Set UTF-8 as default encoding
```

日志文件需要我们手动创建一下：

```
hello@django:~$ mkdir -p /webapps/hello_django/logs/
hello@django:~$ touch /webapps/hello_django/logs/gunicorn_supervisor.log
```

设置好上面的文件以后，我们可以通过`supervisorctl`工具来启用这些设置了：

```
$ sudo supervisorctl reread 
hello: available 
$ sudo supervisorctl update 
hello: added process group
```

现在你可以start，stop或者restart你的进程了

```
$ sudo supervisorctl status hello hello RUNNING pid 18020, uptime 0:00:50 
$ sudo supervisorctl stop hello 
hello: stopped 
$ sudo supervisorctl start hello 
hello: started 
$ sudo supervisorctl restart hello 
hello: stopped 
hello: started
```

## 4、安装配置Nginx

每个Nginx的虚拟server都由`/etc/nginx/sites-available`路径下的一个配置文件来表示。而将其链接到的`/etc/nginx/sites-enabled`路径下则可以启用对应的站点。

为我们的Django应用创建一个配置文件`/etc/nginx/sites-available/hello`. 文件内容如下：

```bash
upstream hello_app_server {
    # fail_timeout=0 means we always retry an upstream even if it failed
    # to return a good HTTP response (in case the Unicorn master nukes a
    # single worker for timing out).
    server unix:/webapps/hello_django/run/gunicorn.sock fail_timeout=0;
}
 
server {
 
    listen 80;
    server_name example.com;

    client_max_body_size 4G;

    access_log /webapps/hello_django/logs/nginx-access.log;
    error_log /webapps/hello_django/logs/nginx-error.log;

    location /static/ {
        alias /webapps/hello_django/static/;
    }

    location /media/ {
        alias /webapps/hello_django/media/;
    }

    location / {
        # an HTTP header important enough to have its own Wikipedia entry:
        # http://en.wikipedia.org/wiki/X-Forwarded-For
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # enable this if and only if you use HTTPS, this helps Rack
        # set the proper protocol for doing redirects:
        # proxy_set_header X-Forwarded-Proto https;

        # pass the Host: header from the client right along so redirects
        # can be set properly within the Rack application
        proxy_set_header Host $http_host;

        # we don't want nginx trying to do something clever with
        # redirects, we set the Host: header above already.
        proxy_redirect off;

        # set "proxy_buffering off" *only* for Rainbows! when doing
        # Comet/long-poll stuff. It's also safe to set if you're
        # using only serving fast clients with Unicorn + nginx.
        # Otherwise you _want_ nginx to buffer responses to slow
        # clients, really.
        # proxy_buffering off;

        # Try to serve static files from nginx, no point in making an
        # *application* server like Unicorn/Rainbows! serve static files.
        if (!-f $request_filename) {
            proxy_pass http://hello_app_server;
            break;
        }
    }

    # Error pages
    error_page 500 502 503 504 /500.html;
    location = /500.html {
    	root /webapps/hello_django/static/;
    }
}
```

将这个文件链接到`site-enabled`文件夹下：

```
$ sudo ln -s /etc/nginx/sites-available/hello /etc/nginx/sites-enabled/hello
```

然后重启nginx

```
$ sudo service nginx restart
```

至此配置就全部完成了，最终项目的整个结构应该如下所示：

```
/webapps/hello_django/ 
├── bin <= Directory created by virtualenv 
│ ├── activate <= Environment activation script
│ ├── django-admin.py 
│ ├── gunicorn 
│ ├── gunicorn_django 
│ ├── gunicorn_start <= Script to start application with Gunicorn 
│ └── python 
├── hello <= Django project directory, add this to PYTHONPATH 
│ ├── manage.py 
│ ├── project_application_1 
│ ├── project_application_2 
│ └── hello <= Project settings directory 
│ ├── __init__.py 
│ ├── settings.py <= hello.settings - settings module Gunicorn will use 
│ ├── urls.py 
│ └── wsgi.py <= hello.wsgi - WSGI module Gunicorn will use 
├── include 
│ └── python2.7 -> /usr/include/python2.7 
├── lib 
│ └── python2.7 
├── lib64 -> /webapps/hello_django/lib 
├── logs <= Application logs directory 
│ ├── gunicorn_supervisor.log 
│ ├── nginx-access.log 
│ └── nginx-error.log 
├── media <= User uploaded files folder 
├── run 
│ └── gunicorn.sock 
└── static <= Collect and serve static files from here
```


# 使用 Fabric 自动化部署 Django 项目

作者：HelloGitHub-**追梦人物**

> 文中涉及的示例代码，已同步更新到 [HelloGitHub-Team 仓库](https://github.com/HelloGitHub-Team/HelloDjango-blog-tutorial)

在上一篇教程中，我们通过手工方式将代码部署到了服务器。整个过程涉及到十几条命令，输了 N 个字符。一旦我们本地的代码有更新，整个过程又得重复来一遍，这将变得非常繁琐。

使用 Fabric 可以在服务器中自动执行命令。因为整个代码部署过程都是相同的，只要我们用 Fabric 写好部署脚本，以后就可以通过运行脚本自动完成部署了。

首先在本地安装 Fabric：

```
$ pipenv install fabric --dev
```

因为 Fabric 只需在本地使用，因此使用 `--dev` 选项，让 Pipenv 将 Fabric 依赖写到 dev-packages 配置下，线上环境就不会安装 Fabric。

## 部署过程回顾

在写 Fabric 脚本之前，先来回顾一下当我们在本地开发环境下更新了代码后，在服务器上的整个部署过程。

1. 远程连接服务器。
2. 进入项目根目录，从远程仓库拉取最新的代码。
3. 如果项目引入了新的依赖，需要执行 `pipenv install --deploy --ignore-pipfile` 安装最新依赖。
4. 如果修改或新增了项目静态文件，需要执行 `pipenv run python manage.py collectstatic` 收集静态文件。
5. 如果数据库发生了变化，需要执行 `pipenv run python manage.py migrate` 迁移数据库。
6. 重启 Nginx 和 Gunicorn 使改动生效。

整个过程就是这样，把每一步操作翻译成 Fabric 对应的脚本代码，这样一个自动化部署脚本就完成了。

## 完善项目配置

### 分离 settings 文件

为了安全，线上环境我们将 debug 改为了 False，但开发环境要改为 True，改来改去将很麻烦。此外，django 的 SECRET_KEY 是很私密的配置，django 的很多安全机制都依赖它，如果不慎泄露，网站将面临巨大安全风险，像我们现在这样直接写在配置文件中，万一不小心公开了源代码，SECRET_KEY 就会直接泄露，好的实践是将这个值写入环境变量，通过从环境变量取这个值。

解决以上问题的一个方案就是拆分 settings.py 文件，不同环境对应不同的 settings 文件，django 在启动时会从环境变量中读取 `DJANGO_SETTINGS_MODULE` 的值，以这个值指定的文件作为应用的最终配置。

我们来把 settings.py 拆分，首先在 blogproject 目录下新建一个 Python 包，名为 settings，然后创建一个 common.py，用于存放通用配置，local.py 存放开发环境的配置，production.py 存放线上环境的配置：

```
blogproject\
	settings\
		__init__.py
		local.py
		production.py
	settings.py
```

将 settings.py 文件中的内容全部复制到 common.py 里，并将 `SECRET_KEY`、`DEBUG`、`ALLOWED_HOSTS` 这些配置移到 local.py 和 production.py 中（common.py 中这些项可以删除）。

开发环境的配置 local.py 内容如下：

```
from .common import *

SECRET_KEY = 'development-secret-key'
DEBUG = True
ALLOWED_HOSTS = ['*']
```

线上环境的配置：

```
from .common import *

SECRET_KEY = os.environ['DJANGO_SECRET_KEY']
DEBUG = False
ALLOWED_HOSTS = ['hellodjango-blog-tutorial.zmrenwu.com']
```

注意这里我们在顶部使用 `from .common import *` 将全部配置从 common.py 导入，然后根据环境的不同，在下面进行配置覆盖。

线上环境和开发环境不同的是，为了安全，DEBUG 模式被关闭，SECRET_KEY 从环境变量获取，ALLOWED_HOSTS 设置了允许的 HTTP HOSTS（具体作用见后面的讲解）。

以上操作完成后，**一定记得删除 settings.py**。

现在我们有了两套配置，一套是 local.py，一套是 production.py，那么启动项目时，django 怎么知道我们使用了哪套配置呢？答案是在运行 manage.py 脚本时，django 默认帮我们指定了。在使用 python manage.py 执行命令时，django 可以接收一个 --settings-module 的参数，用于指定执行命令时，项目使用的配置文件，如果参数未显示指定，django 会从环境变量 DJANGO_SETTINGS_MODULE 里获取。看到 manage.py 的源码：

```python
def main():
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'blogproject.settings')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)
```

可以看到这个 main 函数，第一行的 setdefault 为我们设置了环境变量 `DJANGO_SETTINGS_MODULE` 的值，这句代码的作用是，如果当前环境中 `DJANGO_SETTINGS_MODULE` 的值没有被设置，就将其设置为 `blogproject.settings`，所以我们使用 `python manage.py` 执行命令时，django 默认为我们使用了 settings.py 这个配置。

所以我们可以通过设置环境变量，来指定 django 使用的配置文件。

对于 manage.py，通常在开发环境下执行，因此将这里的 `DJANGO_SETTINGS_MODULE` 的值改为 `blogproject.settings.local`，这样运行开发服务器时 django 会加载 blogproject/settings/local.py 这个配置文件。

另外看到 wsgi.py 文件中，这个文件中有一个 application，是在线上环境时 Gunicorn 加载运行的，将这里面的 `DJANGO_SETTINGS_MODULE` 改为 `blogproject.settings.production`

这样，在使用 manage.py 执行命令时，加载的是 local.py 的设置，而使用 gunicorn 运行项目时，使用的是 production.py 的设置。

### 修改 BASE_DIR 配置项

还有需要注意的一点，看到存放通用配置的 common.py 文件，里面有一个配置项为：

```
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
```

这个 `BASE_DIR` 指向项目根目录，其获取方式为根据所在的配置文件向上回溯，找到项目根目录。因为此前的目录结构为 HelloDjango-blog-tutorial/blogproject/settings.py，因此向上回溯 2 层就到达项目根目录。而现在目录结构变为 HelloDjango-blog-tutorial/blogproject/settings/common.py，需向上回溯 3 层才到达项目根目录，因此需将 `BASE_DIR` 进行一个简单修改，修改如下：

```
BASE_DIR = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
```

即再在外面包一层 os.path.dirname，再向上回退一层，到达项目根目录。

### 设置 Supervisor 环境变量

此外，由于线上环境配置中的 secret_key 从环境变量获取，因此我们改一下 supervisor 的配置，将环境变量导入，打开 supervisor 的配置文件 ~/etc/supervisor/conf.d/hellodjango-blog-tutorial.ini，添加环境变量的配置语句：

```
environment=DJANGO_SECRET_KEY=2pe8eih8oah2_2z1=7f84bzme7^bwuto7y&f(#@rgd9ux9mp-3
```

因为此前可能将代码传过公开的代码仓库，所以最好把线上使用的 SECRET_KEY换一下。这个网站可以自动生成 SECRET_KEY：[Django Secret Key Generator](https://www.miniwebtool.com/django-secret-key-generator/)。

保存配置，然后要执行 update 命令更新配置。

```
$ supervisorctl -c ~/etc/supervisord.conf update
```

## 编写 Fabric 脚本

一切准备工作均已就绪，现在就来使用 Fabric 编写自动部署脚本。

Fabric 脚本通常位于 fabfile.py 文件里，因此先在**项目根目录**下建一个 fabfile.py 文件。

根据上述过程编写的脚本代码如下：

```
from fabric import task
from invoke import Responder
from ._credentials import github_username, github_password


def _get_github_auth_responders():
    """
    返回 GitHub 用户名密码自动填充器
    """
    username_responder = Responder(
        pattern="Username for 'https://github.com':",
        response='{}\n'.format(github_username)
    )
    password_responder = Responder(
        pattern="Password for 'https://{}@github.com':".format(github_username),
        response='{}\n'.format(github_password)
    )
    return [username_responder, password_responder]


@task()
def deploy(c):
    supervisor_conf_path = '~/etc/'
    supervisor_program_name = 'hellodjango-blog-tutorial'

    project_root_path = '~/apps/HelloDjango-blog-tutorial/'

    # 先停止应用
    with c.cd(supervisor_conf_path):
        cmd = 'supervisorctl stop {}'.format(supervisor_program_name)
        c.run(cmd)

    # 进入项目根目录，从 Git 拉取最新代码
    with c.cd(project_root_path):
        cmd = 'git pull'
        responders = _get_github_auth_responders()
        c.run(cmd, watchers=responders)

    # 安装依赖，迁移数据库，收集静态文件
    with c.cd(project_root_path):
        c.run('pipenv install --deploy --ignore-pipfile')
        c.run('pipenv run python manage.py migrate')
        c.run('pipenv run python collectstatic --noinput')

    # 重新启动应用
    with c.cd(supervisor_conf_path):
        cmd = 'supervisorctl start {}'.format(supervisor_program_name)
        c.run(cmd)
```

来分析一下部署代码。

deploy 函数为部署过程的入口，加上 task 装饰器将其标注为一个 fabric 任务。

然后定义了一些项目相关的变量，主要是应用相关代码和配置所在服务器的路径。

deploy 函数被调用时会传入一个 c 参数，这个参数的值是 Fabric 在连接服务器时创建的 ssh 客户端实例，使用这个实例可以在服务器上运行相关命令。

接着就是执行一系列部署命令了，进入某个目录使用 ssh 客户端实例的 `cd` 方法，运行命令使用 `run` 方法。

需要注意的是，每次 ssh 客户端实例执行新的命令是无状态的，即每次都会在服务器根目录执行新的命令，而不是在上一次执行的命令所在目录，所以要在同一个目录下连续执行多条命令，需要使用 `with c.cd` 上下文管理器。

最后，如果服务器没有加入代码仓库的信任列表，运行 git pull 一般会要求输入密码。我们代码托管使用了 GitHub，所以写了一个 GitHub 账户密码响应器，一旦 Fabric 检测到需要输入 GitHub 账户密码，就会调用这个响应器，自动填写账户密码。

由于响应器从 `_credentials.py` 模块导入敏感信息，因此在 fabfile.py 同级目录新建一个 `_credentials.py`文件，写上 GitHub 的用户名和密码：

```
github_username = your-github-username
github_password = your-github-password
```

当然，这个文件包含账户密码等敏感信息，**所以一定记得将这个文件加入 .gitignore 文件**，将其排除在版本控制系统之外，别一不小心提交了公开仓库，导致个人 GitHub 账户泄露。

## 执行 Fabric 自动部署脚本

进入 fabfile.py 文件所在的目录，用 fab 命令运行这个脚本文件（将 server_ip 换为你线上服务器的 ip 地址）：

```
fab -H server_ip --prompt-for-login-password -p deploy
```

这时 Fabric 会自动检测到 fabfile.py 脚本中的 deploy 函数并运行，输入服务器登录密码后回车，然后你会看到命令行输出了一系列字符串，最后看到部署完毕的消息。

如果脚本运行中出错，检查一下命令行输出的错误信息，修复问题后重新运行脚本即可。以后当你在本地开发完相关功能后，只需要执行这一个脚本文件，就可以自动把最新代码部署到服务器了。

**『讲解开源项目系列』**——让对开源项目感兴趣的人不再畏惧、让开源项目的发起者不再孤单。跟着我们的文章，你会发现编程的乐趣、使用和发现参与开源项目如此简单。欢迎留言联系我们、加入我们，让更多人爱上开源、贡献开源～
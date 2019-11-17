# 拆分Django的settings.py配置文件，使线下开发和线上部署更方便

## 1.为什么要拆分

就像本文标题一样，对`Django`的配置文件`Settings.py`进行拆分的主要目的就是为了使线下开发和线上开发配合得更加方便。以我的个人网站(www.eastnotes.com)举例，在本地进行开发时的一些配置，比如数据库配置信息，和远程部署服务器上的配置是不一样的。另外，本地开发所单独需要加载的某些APP(比如`django-debug-toolbar`)，在部署服务器上就不需要加载。

在拆分之前，我的解决方案是将`djangoblog/settings.py`文件写入`.gitignore`配置文件中，这样在版本控制的时候，`Git`就会忽略对这个文件的跟踪，从而保持本地配置和远程配置的不一样。但这样做的缺点就是，每当我需要更改配置文件的时候，就需要改两次，非常麻烦。通过对配置文件的拆分，就能大大减轻这种不便，接下来看具体操作。

## 2.拆分后的效果

```bash
├── settings
│   ├── __init__.py
│   ├── base.py
│   ├── local.py
│   └── production.py
```

我将原始`settings.py`文件拆分成了3个文件，`base.py`存放在本地开发和线上部署所共同的配置信息，`local.py`存放本地开发所特有的数据库配置等信息，`production.py`存放线上部署所特有的数据库配置等信息。`__init__.py`是一个空文件，作用是使`seetings`文件夹成为一个包文件。以便使用`import`语句。

## 3.拆分步骤

### 3.1 创建文件

在原始`settings.py`配置文件的同级项目下创建一个`settings`文件夹，然后在该文件夹下创建四个`.py`文件，命名参考上面一段内容。注意，一定要创建一个空的`__init__.py`文件。否则后期进行导入操作的时候一定会遇到如下的报错：

```python
ModuleNotFoundError: No module named 'djangoblog.settings.local'; 'djangoblog.settings' is not a package
```

### 3.2 内容拆分

内容的拆分需要你从`settings.py`文件中进行筛选，将线上线下所共同需要的配置放入`base.py`中，将线下独有的配置放入到`local.py`中，将线上所独有的配置放入`production.py`中。

### 3.2 内容导入

要想使`local.py`和`production.py`都能使用到`base.py`文件中的配置，我们需要在这两个文件的顶部写入`import`语句：

```python
from .base import *
```

以上就完成了内容上面的拆分，但仅仅拆分完是不够的，此时`django`还不能自动识别并使用这些配置文件，我们还需要接下来的相关配置。

## 4.相关配置

### 4.1 修改manage.py

`manage.py`应该是我们初学`django`时就会使用到的一个工具文件，使用它我们可以启动项目，创建`APP`，创建超级用户等操作。当我们使用它启动项目的时候，它会自动加载文件中的一个配置：`DJANGO_SETTINGS_MODULE`，该配置指定了配置文件的所在位置，由于我们更改了配置文件，因此也一定要更改它才能正常启动`Django`，你需要将`djangoblog`换成你的项目名即可。

```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djangoblog.settings.local')
```

### 4.2 修改BASE_DIR

在`base.py`文件中有一个配置文件路径的选项，拆分之前的代码如下：

```python
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
```

这样设置的`BASE_DIR`的值是`/Users/reborn/PycharmProjects/djangoblog`，也就是说指向的是项目的根目录，但是拆分之后，该值所指向的是`/Users/reborn/PycharmProjects/djangoblog/djangoblog`，如果不修改，`django`会找不到使用了这个路径的文件，比如模板文件，会报出下面的错误：

```python
django.template.exceptions.TemplateDoesNotExist: base.html
```

因此，我们只需要将此目录指向上一级目录就可以了，使用`join`完成：

```python
BASE_DIR = os.path.join(os.path.dirname(os.path.dirname(os.path.abspath(__file__))), "..")
```

### 4.3 修改wsgi.py

修改这个文件的需求是我在部署时才发现的，在完成上面的步骤之后，我就通过`git`更新了服务器上的配置文件，重启`nginx`和`uwsgi`之后，网站依然打不开。使用`python manage.py runserver`之后，项目能正常启动，但是网站依然打不开。

于是我突然想到应该是`uwsgi`找不到我的配置文件，因为线上部署的话是通过`uwsgi`来启动我们的项目的。打开`djangoblog`下的`wsgi.py`文件后，不出我所料，这里面同样有一个有关`DJANGO_SETTINGS_MODULE`的配置，因此我将它指向我的`production.py`配置文件：

```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djangoblog.settings.production')
```

此时，再打开网站就没有问题了。

### 4.4 修改Pycharm

如果你用的是`Pycharm`进行`Django`开发的话需要进行此项修改。每次使用`Pycharm`运行项目的时候，它都会默认加载一个自带的配置，这个配置在`Pycharm`运行按钮的旁边，下拉菜单中有一个`Edit Configurations`，点击之后修改里面的`Environment variables`,目的也是让`Pycharm`能够找到我们的本地配置文件：

```python
PYTHONUNBUFFERED=1;DJANGO_SETTINGS_MODULE=djangoblog.settings.local
```

至此，对`Django`配置文件的拆分工作已经完成，你可以删除原来的`settings.py`文件了。如果你在配置的过程中有任何问题都可以在下方留言，或者关注我的【公众号：程序员向东】私聊我。等你哟~
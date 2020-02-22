# 博客将 Django 1.11+ 升级到 Django 2.2+ 遇到的问题及规避方法

之前就有人一直催我把博客的 Django 升级到 Django 2.0 以上，但是我一直懒得升级，因为我早就看过 Django 2.x 版本和 1.11.x 版本，其实没有太多的不同，所以没有找到需要升级的必要。但是，秉着向新技术看齐的态度，我还是花了些时间（大概半天）把博客依赖的 Django 升级到当前最新版的 Django 2.2.6 版本了，在升级的过程中也遇到了很多问题，这些问题也是之前考虑到的，所以基本都迎刃而解了，这篇文章就分享一下升级的过程。

## 重装所有依赖库

为了升级 django 到 2.0 以上的版本，需要配套升级其他的依赖组件，所以需要把当前项目下面的 requirements.txt 文件里面依赖的版本号全部删除，然后重新创建一个虚拟环境，安装所有依赖的最新版本，这样就可以做到在使用 django2.0 的时候其他依赖组件也能支持 django。

## PyMySQL 版本报错

### 报错现象

将 Django 升级到 2.2 以上版本之后，会发现如下报错：

```
  ...

  File "G:\envs\izone\lib\site-packages\django\db\backends\mysql\base.py", line 36, in <module>
    raise ImproperlyConfigured('mysqlclient 1.3.13 or newer is required; you have %s.' % Database.__version__)
django.core.exceptions.ImproperlyConfigured: mysqlclient 1.3.13 or newer is required; you have 0.9.3.
```

这个问题的意思是需要 mysqlclient 1.3.13 以上版本，但是当前的版本是 0.9.3。之所以会出现这个问题，是因为 django 默认的 MySQL 连接库是 mysqldb，所以它其实想要读取的是这个库的版本，但是由于这个库不支持 Python3 所以我们一般会改成 pymysql 库来连接，所以这个地方就读取到了 PyMySQL 的版本是 0.9.3，因此版本号的要求就自然无法得到满足了。

### 规避方法

方法一：将 Django 的版本降到 2.1.4 版本即可。

方法二：仍然使用 django 2.2+ 但是需要修改一些源码的代码。

首先需要修改报错的时候指定的文件（Django源码）：

```
# G:\envs\izone\lib\site-packages\django\db\backends\mysql\base.py
# 注释掉下面两行

if version < (1, 3, 7):
    raise ImproperlyConfigured('mysqlclient 1.3.7 or newer is required; you have %s.' % Database.__version__)
```

网上说注释掉上面两行之后还会出现编码格式问题（我暂时没有发现），需要修改如下文件：

```
# G:\envs\izone\Lib\site-packages\django\db\backends\mysql\operations.py
# 修改 146 行

if query is not None:
    query = query.decode(errors='replace')  # decode 改为 encode
return query
```

方法三：不要使用 PyMySQL 作为 MySQL 的连接库，而是使用官方推荐的 mysqlclient 作为依赖。因为这个库是支持 Python3 的，而且是官方推荐的库，可以避免出现问题。

由于修改了依赖，所以第一步应该卸载当前虚拟机环境中的 PyMySQL 库同时安装 mysqlclient 1.4.4，这个地方可能会出现问题，解决方式如下：

如果是 Windows 系统，直接使用 `pip install mysqlclient` 会报错，具体的报错还有原因可以自行上网搜索，这个时候应该选择下载 whl 文件的方式安装，具体的下载地址推荐 https://www.lfd.uci.edu/~gohlke/pythonlibs/#mysqlclient

如果是 linux 系统，也可能出现问题，大概是缺少 MySQL 依赖，这个需要根据具体的系统版本去网上搜索依赖自行试试。

当依赖更换完毕之后，还需要修改项目配置（因为之前的项目配置中将 PyMySQL 设置成了默认的连接库，所以要删除掉），具体是删除如下两行，我的项目代码已经删除：

```
# 更换默认的数据库连接    
import pymysql  

pymysql.install_as_MySQLdb()
```

## 关于 on_delete 参数的报错

### 报错现象

```
    class Article(models.Model):
  File "G:\Mycodes\izone\apps\blog\models.py", line 71, in Article
    author = models.ForeignKey(settings.AUTH_USER_MODEL, verbose_name='作者')
TypeError: __init__() missing 1 required positional argument: 'on_delete'
```

报错的原因其实一看就能知道，这个地方是说的关于一些模型在一对一关联和一对多关联（外键）的时候需要指定 on_delete 参数才行，多对多没有这个设置。因为在 Django 1.x 的版本中，这个参数是有默认值的，但是 Django 2.x 没有指定，所以需要显示设定一个值。

### 规避方法

把项目中所有模型中有外键的字段都添加这个属性，添加的时候需要根据实际情况设定值。

以下是主要的几个设置项的意义：

```
on_delete=models.CASCADE,     # 删除关联数据,与之关联也删除
on_delete=models.DO_NOTHING,  # 删除关联数据,什么也不做
on_delete=models.PROTECT,     # 删除关联数据,引发错误ProtectedError
on_delete=models.SET_NULL,    # 删除关联数据,与之关联的值设置为null（前提该字段需要设置为可空,一对一同理）
on_delete=models.SET_DEFAULT, # 删除关联数据,与之关联的值设置为默认值（前提FK字段需要设置默认值,一对一同理）
on_delete-models.SET(),      # 删除之后执行一个函数
```

比如我这里添加了如下几个字段：

```
# class Article(models.Model):
author = models.ForeignKey(settings.AUTH_USER_MODEL, verbose_name='作者',
    on_delete=models.PROTECT)  # 当删除有文章的作者时引发错误
category = models.ForeignKey(Category, verbose_name='文章分类', 
    on_delete=models.PROTECT) # 当删除有文章的分类时引发错误

# class ToolLink(models.Model):
category = models.ForeignKey(ToolCategory, verbose_name='网站分类', 
    blank=True, null=True, on_delete=models.SET_NULL) # 当工具分类删除后把分类设置为空

# class Comment(models.Model):
# 删除评论人或者父评论的时候同时删除评论
author = models.ForeignKey(settings.AUTH_USER_MODEL, related_name='%(class)s_related',
    verbose_name='评论人', on_delete=models.CASCADE) 
parent = models.ForeignKey('self', verbose_name='父评论', related_name='%(class)s_child_comments', blank=True,
    null=True, on_delete=models.CASCADE)
rep_to = models.ForeignKey('self', verbose_name='回复', related_name='%(class)s_rep_comments',
    blank=True, null=True, on_delete=models.CASCADE)

#class ArticleComment(Comment):
belong = models.ForeignKey(Article, related_name='article_comments', verbose_name='所属文章',
    on_delete=models.CASCADE) # 提醒的文章被删除则删除提醒

# 省略其他修改项，都是根据实际情况添加设置
```

## 关于 urls.py 中 include() 的报错

### 报错现象

```
  File "G:\envs\izone\lib\site-packages\django\urls\conf.py", line 39, in include
    'Specifying a namespace in include() without providing an app_name '
django.core.exceptions.ImproperlyConfigured: Specifying a namespace in include() without providing an app_name is not supported. Set the app_name attribute in the inc
luded module, or pass a 2-tuple containing the list of patterns and app_name instead.
```

报错分析，这个是 django 2.x 里面会出现的问题，当 urls.py 里面使用 include 的时候，当有 namespace 参数的时候，第一个参数需要添加成元组，其中第二个参数是 app_name。

### 规避方法

首先，Django 2.x 跟 Django 1.x 最明显的修改就是在路由的格式，可以使用 path 替换之前使用的 url，会显得更简单，下面是我修改的：

```
# blog app 的写法
path('', include(('blog.urls', 'blog'), namespace='blog')),  # blog

# restframework 的写法
urlpatterns.append(path('api/v1/',include((router.urls, router.root_view_name),namespace='api')))
```

## redis 存储时序列化报错

### 报错现象

```
PicklingError at /article/Jenkins-install/
Can't pickle <class 'markdown.util.PriorityItem'>: attribute lookup PriorityItem on markdown.util failed
```

报错原因：由于我在把文章 markdown 转化之后就直接存放到 redis 中了，但是存储的时候序列化失败了（这个比较奇怪，之前没有遇到过）。

### 规避方法

修改之后，我的代码如下：

```
ud = obj.update_date.strftime("%Y%m%d%H%M%S")
md_key = '{}_md_{}'.format(obj.id, ud)
cache_md = cache.get(md_key)
if cache_md:
    obj.body, obj.toc = cache_md
else:
    md = markdown.Markdown(extensions=[
        'markdown.extensions.extra',
        'markdown.extensions.codehilite',
        TocExtension(slugify=slugify),
    ])
    obj.body = md.convert(obj.body)
    obj.toc = md.toc
    # 关键在这个地方，不能直接把 md 存进去
    cache.set(md_key, (obj.body, obj.toc), 60 * 60 * 12)
```

## 模板错误

模板报错都是和版本升级有关，因为我的博客重写了一些模板，所以升级版本必须同步到最新的模板去修改才行

下面这个报错是 bootstrap_admin 的模板被我改写了，所以报错，更新模板即可。

```
    "or load this tag?" % (token.lineno, command)
django.template.exceptions.TemplateSyntaxError: Invalid block tag on line 14: 'datetime_widget_css_path'. Did you forget to register or load this tag?
```

## 升级MR

具体的修改MR可以看：https://github.com/Hopetree/izone/pull/76/files

后续补充MR：https://github.com/Hopetree/izone/pull/80/files
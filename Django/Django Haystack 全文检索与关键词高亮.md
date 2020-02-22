# Django Haystack 全文检索与关键词高亮

> 文中所涉及的示例代码，已同步更新到 [HelloGitHub-Team 仓库](https://github.com/HelloGitHub-Team/HelloDjango-blog-tutorial)

博客提供 RSS 订阅应该是标配，这样读者就可以通过一些聚合阅读工具订阅你的博客，时时查看是否有文章更新，而不必每次都跳转到博客上来查看。现在我们就来为博客添加 RSS 订阅功能。

在此之前我们使用了 Django 内置的一些方法实现了一个简单的搜索功能。但这个搜索功能实在过于简单，没有多大的实用性。对于一个搜索引擎来说，至少应该能够根据用户的搜索关键词对搜索结果进行排序以及高亮关键字。现在我们就来使用 django-haystack 实现这些特性。

## Django Haystack 简介

django-haystack 是一个专门提供搜索功能的 django 第三方应用，它支持 Solr、Elasticsearch、Whoosh、Xapian 等多种搜索引擎，上一版本的教程中我们使用 Whoosh 加 jieba 中文分词的方案，原因是为了简单，无需安装外部服务。但现在有了 docker，安装一个外部服务就是轻而易举的事情，所以这次我们采用更为强大的 elasticsearch 作为我们博客的搜索引擎，同时使用 elasticsearch 的中文分词插件 ik，来提升中文搜索的效果。

## 安装必要依赖

### 安装 django-haystack

django-haystack 安装非常简单，只需要执行 `pipenv install django-haystack` 即可。需要注意的是，目前 elasticsearch 有 2 系列和 5 系列两大版本，本来新项目的原则是尽可能采用新版本，但目前 django-haystack 在 pypi 上发布的稳定版只支持 elasticsearch2，master 分支下支持 elasticsearch5，因此处于稳定性考虑，我们暂时使用 elasticsearch2，后续如果 django-haystack 发布了支持 elasticsearch5 的pypi版本，我们会升级到 elasticsearch5，有了 docker，升级就是轻而易举的事情。

由于使用 elasticsearch 服务，haystack 连接 elasticsearch 需要 python 版本的 SDK 支持，因此还需要安装 elasticsearch python SDK，这里我们不要直接使用 pipenv 安装，而是手动编辑 Pipfile 文件，指定 SDK 的版本，否则 pipenv 默认会安装最新版。打开 Pipfile 文件，将依赖手动添加到 packages 板块下：

```
[packages]
django = "~=2.2"
elasticsearch = ">=2,<3"
```

### 安装 elasticsearch 2

接下来就是构建一个新的容器来运行 elasticsearch 服务，因此首先需要来编排容器镜像，回顾一下容器镜像的目录结构：

```
compose\
	local\
	production\
		django\
		nginx\
```

由于 elasticsearch 在线上环境和本地测试都要使用，我们把镜像编排在 production 目录下，新建一个 elasticsearch 目录，用来存放和 elasticsearch 相关的内容。Dockfile 内容如下：

```
FROM elasticsearch:2.4.6-alpine

COPY ./compose/production/elasticsearch/elasticsearch-analysis-ik-1.10.6.zip /usr/share/elasticsearch/plugins/
RUN cd /usr/share/elasticsearch/plugins/ && mkdir ik && unzip elasticsearch-analysis-ik-1.10.6.zip -d ik/
RUN rm /usr/share/elasticsearch/plugins/elasticsearch-analysis-ik-1.10.6.zip

USER root
COPY ./compose/production/elasticsearch/elasticsearch.yml /usr/share/elasticsearch/config/
RUN chown elasticsearch:elasticsearch /usr/share/elasticsearch/config/elasticsearch.yml

USER elasticsearch
```

这个镜像从 elasticsearch 的官方基础镜像 2.4.6 版本进行构建，接着我们把 ik 分词插件复制到 elasticsearch 安装插件的目录下，然后解压启用。

接着我们又把 elasticsearch.yml 配置文件复制到容器内，然后切换用户为 elasticsearch，因为我们将以 elasticsearch 用户和组运行 elasticsearch 服务。

elasticsearch.yml 配置文件内容很简单：

```
bootstrap.memory_lock: true
network.host: 0.0.0.0
```

其中 `bootstrap.memory_lock` 这个参数是为了提高 elasticsearch 的效率（涉及到 JVM 相关的优化，不做过多介绍）。`network.host` 指定服务启动的地址。

接着修改 docker compose 文件，我们先在本地启动，因此修改 local.yml 文件，加入 elasticsearch 服务：

```
version: '3'

volumes:
  database_local:
  esdata_local:

services:
  hellodjango_blog_tutorial_local:
	# 其它配置不变...
    depends_on:
      - elasticsearch_local

  elasticsearch_local:
    build:
      context: .
      dockerfile: ./compose/production/elasticsearch/Dockerfile
    image: elasticsearch_local
    container_name: elasticsearch_local
    volumes:
      - esdata_local:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nproc: 65536
      nofile:
        soft: 65536
        hard: 65536
```

主要是加入了 elasticsearch 服务，其中 `environment` 和 `ulimits` 的参数与 elasticksearch 服务调优有关，对于简单的博客搜索来说，调优的意义不是很大，因此这里不做过多介绍，感兴趣的可以参考 elasticksearch 的[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/master/setting-system-settings.html#ulimit)。

## 配置 Haystack

安装好 django haystack 后需要在项目的 settings.py 做一些简单的配置。

首先是把 django haystack 加入到 `INSTALLED_APPS` 设置里：

```
blogproject/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
	# 其它 app...
    'haystack',
    'blog',
    'comments',
]
```

然后加入如下配置项：

```
blogproject/common.py

# 搜索设置
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.elasticsearch2_backend.Elasticsearch2SearchEngine',
        'URL': '',
        'INDEX_NAME': 'hellodjango_blog_tutorial',
    },
}
HAYSTACK_SEARCH_RESULTS_PER_PAGE = 10
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
```

`HAYSTACK_CONNECTIONS` 的 `ENGINE` 指定了 django haystack 使用的搜索引擎，这里我们使用了 haystack 默认的 Elasticsearch2 搜索引擎。`PATH` 指定了索引文件需要存放的位置，我们设置为项目根目录 `BASE_DIR` 下的 whoosh_index 文件夹（在建立索引是会自动创建）。

`HAYSTACK_SEARCH_RESULTS_PER_PAGE` 指定如何对搜索结果分页，这里设置为每 10 项结果为一页。

`HAYSTACK_SIGNAL_PROCESSOR` 指定什么时候更新索引，这里我们使用 `haystack.signals.RealtimeSignalProcessor`，作用是每当有文章更新时就更新索引。由于博客文章更新不会太频繁，因此实时更新没有问题。

由于开发环境和线上环境，elasticsearch 服务的 url 地址是不同的，所以我们在 common 的配置中没有指定 url，在 local.py 设置文件指定之：

```
HAYSTACK_CONNECTIONS['default']['URL'] = 'http://elasticsearch_local:9200/'
```

## 处理数据

接下来就要告诉 django haystack 使用哪些数据建立索引以及如何存放索引。如果要对 blog 应用下的数据进行全文检索，做法是在 blog 应用下建立一个 search_indexes.py 文件，写上如下代码：

```
blog/search_indexes.py

from haystack import indexes
from .models import Post


class PostIndex(indexes.SearchIndex, indexes.Indexable):
    text = indexes.CharField(document=True, use_template=True)

    def get_model(self):
        return Post

    def index_queryset(self, using=None):
        return self.get_model().objects.all()
```

这是 django haystack 的规定。要相对某个 app 下的数据进行全文检索，就要在该 app 下创建一个 search_indexes.py 文件，然后创建一个 XXIndex 类（XX 为含有被检索数据的模型，如这里的 Post），并且继承 `SearchIndex` 和 `Indexable`。

为什么要创建索引？索引就像是一本书的目录，可以为读者提供更快速的导航与查找。在这里也是同样的道理，当数据量非常大的时候，若要从这些数据里找出所有的满足搜索条件的几乎是不太可能的，将会给服务器带来极大的负担。所以我们需要为指定的数据添加一个索引（目录），在这里是为 Post 创建一个索引，索引的实现细节是我们不需要关心的，我们只关心为哪些字段创建索引，如何指定。

每个索引里面必须有且只能有一个字段为 `document=True`，这代表 django haystack 和搜索引擎将使用此字段的内容作为索引进行检索(primary field)。注意，如果使用一个字段设置了`document=True`，则一般约定此字段名为`text`，这是在 `SearchIndex` 类里面一贯的命名，以防止后台混乱，当然名字你也可以随便改，不过不建议改。

并且，haystack 提供了 use_template=True 在 text 字段中，这样就允许我们使用数据模板去建立搜索引擎索引的文件，说得通俗点就是索引里面需要存放一些什么东西，例如 Post 的 title 字段，这样我们可以通过 title 内容来检索 Post 数据了。举个例子，假如你搜索 Python ，那么就可以检索出 title 中含有 Python 的Post了，怎么样是不是很简单？数据模板的路径为 templates/search/indexes/youapp/<model_name>_text.txt（例如 templates/search/indexes/blog/post_text.txt），其内容为：

```
templates/search/indexes/blog/post_text.txt

{{ object.title }}
{{ object.body }}
```

这个数据模板的作用是对 Post.title、Post.body 这两个字段建立索引，当检索的时候会对这两个字段做全文检索匹配，然后将匹配的结果排序后作为搜索结果返回。

## 配置 URL

接下来就是配置 URL，搜索的视图函数和 URL 模式 django haystack 都已经帮我们写好了，只需要项目的 urls.py 中包含它：

```
blogproject/urls.py

urlpatterns = [
	# 其它...
    path('search/', include('haystack.urls')),
]
```

另外在此之前我们也为自己写的搜索视图配置了 URL，把那个 URL 删掉，以免冲突：

```
blog/urls.py

# path('search/', views.search, name='search'),
```

## 修改搜索表单

修改一下搜索表单，让它提交数据到 django haystack 搜索视图对应的 URL：

```
<form role="search" method="get" id="searchform" action="{% url 'haystack_search' %}">
  <input type="search" name="q" placeholder="搜索" required>
  <button type="submit"><span class="ion-ios-search-strong"></span></button>
</form>
```

主要是把表单的 action 属性改为 {% url 'haystack_search' %}

## 创建搜索结果页面

`haystack_search` 视图函数会将搜索结果传递给模板 search/search.html，因此创建这个模板文件，对搜索结果进行渲染：

```
templates/search/search.html

{% extends 'base.html' %}
{% load highlight %}

{% block main %}
  {% if query %}
    {% for result in page.object_list %}
      <article class="post post-{{ result.object.pk }}">
        <header class="entry-header">
          <h1 class="entry-title">
            <a href="{{ result.object.get_absolute_url }}">{% highlight result.object.title with query %}</a>
          </h1>
          <div class="entry-meta">
                    <span class="post-category">
                        <a href="{% url 'blog:category' result.object.category.pk %}">
                            {{ result.object.category.name }}</a></span>
            <span class="post-date"><a href="#">
                            <time class="entry-date" datetime="{{ result.object.created_time }}">
                                {{ result.object.created_time }}</time></a></span>
            <span class="post-author"><a href="#">{{ result.object.author }}</a></span>
            <span class="comments-link">
                        <a href="{{ result.object.get_absolute_url }}#comment-area">
                            {{ result.object.comment_set.count }} 评论</a></span>
            <span class="views-count"><a
                    href="{{ result.object.get_absolute_url }}">{{ result.object.views }} 阅读</a></span>
          </div>
        </header>
        <div class="entry-content clearfix">
          <p>{% highlight result.object.body with query %}</p>
          <div class="read-more cl-effect-14">
            <a href="{{ result.object.get_absolute_url }}" class="more-link">继续阅读 <span
                    class="meta-nav">→</span></a>
          </div>
        </div>
      </article>
    {% empty %}
      <div class="no-post">没有搜索到你想要的结果！</div>
    {% endfor %}

    {% if page.has_previous or page.has_next %}
      <div class="text-center" style="margin-top: 30px">
        {% if page.has_previous %}
          <a href="?q={{ query }}&amp;page={{ page.previous_page_number }}">{% endif %}&laquo; Previous
        {% if page.has_previous %}</a>{% endif %}
        <span style="margin: 0 10px">|</span>
        {% if page.has_next %}<a href="?q={{ query }}&amp;page={{ page.next_page_number }}">{% endif %}Next
        &raquo;{% if page.has_next %}</a>{% endif %}
      </div>
    {% endif %}
  {% else %}
    请输入搜索关键词，例如 django
  {% endif %}
{% endblock main %}
```

这个模板基本和 blog/index.html 一样，只是由于 haystack 对搜索结果做了分页，传给模板的变量是一个 page 对象，所以我们从 page 中取出这一页对应的搜索结果，然后对其循环显示，即 {% for result in page.object_list %}。另外要取得 Post（文章）以显示文章的数据如标题、正文，需要从 result 的 object 属性中获取。`query` 变量的值即为用户搜索的关键词。

## 高亮关键词

注意到百度的搜索结果页面，含有用户搜索的关键词的地方都是被标红的，在 django haystack 中实现这个效果也非常简单，只需要使用 {% highlight %} 模板标签即可，其用法如下：

```
# 使用默认值  
{% highlight result.summary with query %}  
  
# 这里我们为 {{ result.summary }} 里所有的 {{ query }} 指定了一个<div></div>标签，并且将class设置为highlight_me_please，这样就可以自己通过CSS为{{ query }}添加高亮效果了，怎么样，是不是很科学呢  
{% highlight result.summary with query html_tag "div" css_class "highlight_me_please" %}  
  
# 可以 max_length 限制最终{{ result.summary }} 被高亮处理后的长度
{% highlight result.summary with query max_length 40 %}
```

在博客文章搜索页中我们对 title 和 body 做了高亮处理：{% highlight result.object.title with query %}，{% highlight result.object.body with query %}。高亮处理的原理其实就是给文本中的关键字包上一个 span 标签并且为其添加 highlighted 样式（当然你也可以修改这个默认行为，具体参见上边给出的用法）。因此我们还要给 highlighted 类指定样式，在 base.html 中添加即可：

```
base.html

<head>
    <title>Black &amp; White</title>
  	...
    <style>
        /* 搜索关键词高亮 */
        span.highlighted {
          color: red;
        }
    </style>
    ...
</head>
```

## 建立索引文件

最后一步就是建立索引文件了，运行命令 ：

```
$ docker exec -it hellodjango_blog_tutorial_local python manage.py rebuild_index
```

就可以建立索引文件了。一切就绪后，就可以尝试搜索了。但是体验下来会发现搜索的结果并不是很友好，很多关键词文章中命名存在但搜索结果中却没有显示，原因是 haystack 专门为英文搜索设计，如果使用其默认的搜索引擎分词器，中文搜索的结果就不是很理想，接下来我们来将它默认的分词器设置为中文分词器。

## 修改搜索引擎为中文分词

还记得文章开头编排 elasticsearch 的 Docker 镜像时，我们将一个 elasticsearch 的中文分词插件复制到了 elasticsearch 的插件目录，接下来要做的，就是让 haystack 在创建索引时，使用指定的插件来对进行分词并创建索引，具体做法是，首先在 blog 应用下创建一个 elasticsearch2_ik_backend.py，代码如下：

```python
from haystack.backends.elasticsearch2_backend import Elasticsearch2SearchBackend, Elasticsearch2SearchEngine

DEFAULT_FIELD_MAPPING = {'type': 'string', "analyzer": "ik_max_word", "search_analyzer": "ik_smart"}

class Elasticsearch2IkSearchBackend(Elasticsearch2SearchBackend):

    def __init__(self, *args, **kwargs):
        self.DEFAULT_SETTINGS['settings']['analysis']['analyzer']['ik_analyzer'] = {
            "type": "custom",
            "tokenizer": "ik_max_word",
        }
        super(Elasticsearch2IkSearchBackend, self).__init__(*args, **kwargs)


class Elasticsearch2IkSearchEngine(Elasticsearch2SearchEngine):
    backend = Elasticsearch2IkSearchBackend
```

这些代码的作用是，继承 haystack 默认的 Elasticsearch2SearchBackend 和 Elasticsearch2SearchEngine，覆盖掉它的一些默认行为，这里主要就是让 haystack 在创建索引时，使用指定的 ik 分词器。

由于自定义了搜索引擎，因此在配置文件中将原来指定的 Elasticsearch2SearchEngine 替换为自定义的 Engine：

```
# 搜索设置
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'blog.elasticsearch2_ik_backend.Elasticsearch2IkSearchEngine',
        'URL': '',
        'INDEX_NAME': 'hellodjango_blog_tutorial',
    },
}
HAYSTACK_SEARCH_RESULTS_PER_PAGE = 10
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
```

由于修改了索引创建方式，因此需要重建一下索引：`python manage.py rebuild_index`。然后就可以查看搜索结果了，中文搜索体验是不是好了很多？

## 防止标题被截断

haystack 在展示搜索结果时，默认行为是将第一个出现的关键词前的内容截断，被截掉的部分用省略号代替。对于正文来说，因为内容较多，截断是合理的，但是对于标题这种较短的内容来说，截断就没有必要了。同样的，我们通过继承的方式，替换掉 haystack 的默认行为。我们在 blog/utils.py 中继承 `HaystackHighlighter` 这个用于高亮搜索关键词的辅助类。

```
from django.utils.html import strip_tags
from haystack.utils import Highlighter as HaystackHighlighter


class Highlighter(HaystackHighlighter):
    """
    自定义关键词高亮器，不截断过短的文本（例如文章标题）
    """

    def highlight(self, text_block):
        self.text_block = strip_tags(text_block)
        highlight_locations = self.find_highlightable_words()
        start_offset, end_offset = self.find_window(highlight_locations)
        if len(text_block) < self.max_length:
            start_offset = 0
        return self.render_html(highlight_locations, start_offset, end_offset)
```

关键代码是：`if len(text_block) < self.max_length:`，`start_offset` 是 haystack 根据关键词算出来第一个关键词在文本中出现的位置。`max_length` 指定了展示结果的最大长度。我们在代码中做一个判断，如果文本内容 `text_block` 没有超过允许的最大长度，就将 `start_offset` 设为 0，这样就从文本的第一个字符开始展示，标题这种短文本就不会被截断了。

然后设置，让 haystack 在高亮文本时，使用我们自定义的辅助类：

```
HAYSTACK_CUSTOM_HIGHLIGHTER = 'blog.utils.Highlighter'
复制代码
```

在来看一下搜索效果吧！



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="630"></svg>)



## 线上发布

以上步骤都是在本地运行调试的，elasticsearch 服务也是在本地的 Docker 容器中运行，接下来在 production.yml 中加入 elasticsearch 服务，就可以发布线上了，配置内容和 local.yml 是一样的，只是简单修改一下服务名和容器名等命名：

```
  elasticsearch:
    build:
      context: .
      dockerfile: ./compose/production/elasticsearch/Dockerfile
    image: hellodjango_blog_tutorial_elasticsearch
    container_name: hellodjango_blog_tutorial_elasticsearch
    volumes:
      - esdata:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nproc: 65536
      nofile:
        soft: 65536
        hard: 65536
```

别忘了修改 settings/production.py，修改线上环境 elasticsearch 服务的连接地址：

```
HAYSTACK_CONNECTIONS['default']['URL'] = 'http://hellodjango_blog_tutorial_elasticsearch:9200/'
```

这样就可以直接发布线上了！
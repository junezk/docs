# Django+Markdown2+bootstrap+google-code-prettify

[TOC]
Markdown是一种可以使用普通文本编辑器编写的标记语言，通过类似HTML的标记语法，它可以使普通文本内容具有一定的格式。
在现在的网站中都使用 Markdown 格式存储数据,(或者是在录入时使用 Markdwon 格式,录入完成后使用 Markdown 解析器解析成 HTML 存入数据库).
使用 Django + Markdown 就可以实现一个网站构架,这个内容比较多有时间在另开篇幅,今天我们主要来讨论如何使用 boostrap + google-code-prettify 来显示漂亮的代码段.

## Django 中使用 Markdown2

在 Django 中使用 Markdown2 很简单.首先要确保你机器中安装了 Markdown2:

```
pip install markdonw2    
```

在我的网站中,数据中存储 markdown 格式数据在渲染时在通过 Markdown 解析器解析成 HTML 插入 Django 模板中(我不知道这样对性能的影响有多大,有经验的童鞋请赐教^_^).部分代码如下:

```python
from django.utils.safestring import mark_safe
from markdown2 import markdown

class Blog(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    content = models.TextField(verbose_name=u'文章正文markdown')

    ...

    def __unicode__(self):
        return self.title

    def get_tags(self):
        return [tag.strip() for tag in self.tags.split(',')]

    def get_content_html(self):
        html = markdown(self.content, extras=['code-friendly'])
        return mark_safe(html)
```

content 中存储这 Markdown 格式的数据, 使用 markdwon 函数转化成 HTML,
然后, 要注意使用 mark_safe 函数.

## bootstrap

Bootstrap，来自 Twitter，是目前最受欢迎的前端框架。Bootstrap 是基于 HTML、CSS、JAVASCRIPT 的，它简洁灵活，使得 Web 开发更加快捷。[1] 它由Twitter的设计师Mark Otto和Jacob Thornton合作开发，是一个CSS/HTML框架。Bootstrap提供了优雅的HTML和CSS规范，它即是由动态CSS语言Less写成。Bootstrap一经推出后颇受欢迎，一直是GitHub上的热门开源项目，包括NASA的MSNBC（微软全国广播公司）的Breaking News都使用了该项目。

bootstrap 的使用很简单,下载 bootstrap 然后在你的模板中添加:

```
<link href="/static/bootstrap/css/bootstrap.min.css" rel="stylesheet">
```

注意替换路径. bootstrap 需要使用 jquery, 下载 jqery 到 bootstrap 的 js 文件夹下.

## google-code-prettify 代码高亮

github主页:[https://github.com/tcollard/google-code-prettify](https://github.com/tcollard/google-code-prettify)
google-code-prettify 的使用也比较简单.只需要引用 google-code-prettify 的 js 和 css 文件:

```
    <link href="/static/google-code-prettify/prettify.css" type="text/css" rel="stylesheet"/>
    <script type="text/javascript" src="/static/google-code-prettify/prettify.js"></script>
```

然后在 body 标签的 onload 事件中调用函数 prettyPrint

```
<body onload="prettyPrint">
...
</body>
```

这样所有

```
<pre class="prettyprint">
...
</pre>
```

标签中的代码都会高亮, google-code-prettify 会自动猜测你的语言类型,当然你也可以指定语言,具体看 google-code prettify 的帮助文档了.
但是,由于我们的 HTML 内容是使用 Markdown 实时解析的, 故而,我使用了 jquery 为所有 pre 标签加上 class=”prettyprint”

```
(document).ready(function(){
    $('pre').addClass("prettyprint");
    prettyPrint();
});
```

prettyPrint 也在这里调用了,所以不需要 body 标签中调用. 如果对这个语法很疑惑的童鞋需要去看看 jquery 的语法.

好了,这就是我建站过程中用到一些东西,做一些记录,就酱紫…..
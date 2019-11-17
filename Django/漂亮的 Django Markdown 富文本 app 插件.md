# 漂亮的 Django Markdown 富文本 app 插件

# django-mdeditor

**Django-mdeditor** 是基于 [Editor.md](https://github.com/pandao/editor.md) 的一个 [django](https://juejin.im/post/djangoproject.com) Markdown 文本编辑插件应用。

**Django-mdeditor** 的灵感参考自伟大的项目 [django-ckeditor](https://github.com/django-ckeditor/django-ckeditor).

**注：** 关于Markdown页面渲染问题，建议后端渲染。因`Editor.md` 已长时间不更新有些bug和兼容性问题需要自己调试，当然前端同学可自行选择。

## 功能

- 支持 Editor.md 大部分功能
  - 支持标准的Markdown 文本、 CommonMark 和 GFM (GitHub Flavored Markdown) 文本;
  - 支持实时预览、图片上传、格式化代码、搜索替换、皮肤、多语言等。
  - 支持TOC 目录和表情；
  - 支持 TeX, 流程图、时序图等图表扩展。
- 可自定义 Editor.md 工具栏。
- 提供了 `MDTextField` 字段用来支持模型字段使用。
- 提供了 `MDTextFormField` 字段用来支持 `Form` 和 `ModelForm`.
- 提供了 `MDEditorWidget` 字段用来支持 `admin` 自定义样式使用。

## 快速入门

- 安装

```
    pip install django-mdeditor
```

- 在 `settings` 配置文件 `INSTALLED_APPS` 中添加 `mdeditor`:

```
    INSTALLED_APPS = [
        ...
        'mdeditor',
    ]
```

- 在 `settings` 中添加媒体文件的路径配置:

```
MEDIA_ROOT = os.path.join(BASE_DIR, 'uploads')
MEDIA_URL = '/media/'
```

在你项目根目录下创建 `uploads/editor` 目录，用于存放上传的图片。

- 在你项目的根 `urls.py` 中添加扩展url和媒体文件url:

```
from django.conf.urls import url, include
from django.conf.urls.static import static
from django.conf import settings
...

urlpatterns = [
    ...
    url(r'mdeditor/', include('mdeditor.urls'))
]

if settings.DEBUG:
    # static files (images, css, javascript, etc.)
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

- 编写一个测试 model :

```
from django.db import models
from mdeditor.fields import MDTextField

class ExampleModel(models.Model):
    name = models.CharField(max_length=10)
    content = MDTextField()
复制代码
```

- 向 `admin.py` 中注册model:

```
from django.contrib import admin
from . import models

admin.site.register(models.ExampleModel)
```

- 运行 `python manage.py makemigrations` 和 `python manage.py migrate` 来创建你的model 数据库表.
- 登录 django admin后台，点击 '添加'操作，你会看到如下界面。

![img](漂亮的 Django Markdown 富文本 app 插件.assets/16c3c13f10a8b1f5)

到此，你已经初步体验了 `djang-mdeditor` ，接下来详细看下他的其他使用吧。

## 用法说明

### 在model 中使用 Markdown 编辑字段

在model 中使用 Markdown 编辑字段，我们只需要将 model 的`TextField` 替换成`MDTextField` 即可。

```
from django.db import models
from mdeditor.fields import MDTextField

class ExampleModel(models.Model):
    name = models.CharField(max_length=10)
    content = MDTextField()
```

在后台admin中，会自动显示 markdown 的编辑富文本。

在前端 template 中使用时，可以这样用：

```
{% load staticfiles %}
<!DOCTYPE html>
<html lang="zh">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />

    </head>
    <body>
        <form method="post" action="./">
            {% csrf_token %}
            {{ form.media }}
            {{ form.as_p }}
            <p><input type="submit" value="post"></p>
        </form>
    </body>
</html>
```

### 在 Form 中使用 markdown 编辑字段

在 Form 中使用 markdown 编辑字段，使用 `MDTextFormField` 代替 `forms.CharField`, 如下：

```
from mdeditor.fields import MDTextFormField

class MDEditorForm(forms.Form):
    name = forms.CharField()
    content = MDTextFormField()
```

`ModelForm` 可自动将model 对应的字段转为 form字段， 可正常使用：

```
class MDEditorModleForm(forms.ModelForm):

    class Meta:
        model = ExampleModel
        fields = '__all__'
```

### 在 admin 中使用 markdown 小组件

在 admin 中使用 markdown 小组件，如下：

```
from django.contrib import admin
from django.db import models

# Register your models here.
from . import models as demo_models
from mdeditor.widgets import MDEditorWidget

class ExampleModelAdmin(admin.ModelAdmin):
    formfield_overrides = {
        models.TextField: {'widget': MDEditorWidget}
    }

admin.site.register(demo_models.ExampleModel, ExampleModelAdmin)
```

### 自定义工具栏

在 `settings` 中增加如下配置 ：

```
MDEDITOR_CONFIGS = {
    'width': '90%',  # 自定义编辑框宽度
    'heigth': 500,   # 自定义编辑框高度
    'toolbar': ["undo", "redo", "|",
                "bold", "del", "italic", "quote", "ucwords", "uppercase", "lowercase", "|",
                "h1", "h2", "h3", "h5", "h6", "|",
                "list-ul", "list-ol", "hr", "|",
                "link", "reference-link", "image", "code", "preformatted-text", "code-block", "table", "datetime",
                "emoji", "html-entities", "pagebreak", "goto-line", "|",
                "help", "info",
                "||", "preview", "watch", "fullscreen"],  # 自定义编辑框工具栏
    'upload_image_formats': ["jpg", "jpeg", "gif", "png", "bmp", "webp"],  # 图片上传格式类型
    'image_folder': 'editor',  # 图片保存文件夹名称
    'theme': 'default',  # 编辑框主题 ，dark / default
    'preview_theme': 'default',  # 预览区域主题， dark / default
    'editor_theme': 'default',  # edit区域主题，pastel-on-dark / default
    'toolbar_autofixed': True,  # 工具栏是否吸顶
    'search_replace': True,  # 是否开启查找替换
    'emoji': True,  # 是否开启表情功能
    'tex': True,  # 是否开启 tex 图表功能
    'flow_chart': True,  # 是否开启流程图功能
    'sequence': True,  # 是否开启序列图功能
    'watch': True,  # 实时预览
    'lineWrapping': False,  # 自动换行
    'lineNumbers': False  # 行号
}
```
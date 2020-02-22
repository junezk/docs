# Django如何上传处理文件及Ajax文件上传示范(附GitHub源码)

小编我今天要写篇值得大家收藏的文章。我将重点解释Django上传处理文件中需要考虑的重要事项，并提供一般文件上传及Ajax文件上传的示范(附GitHub源码)。如果你的项目需要用到文件上传，你可以从GitHub获取源代码，简化你的开发。

## Django文件上传需要考虑的重要事项

文件一般通过表单进行。用户在前端点击文件上传，然后以POST方式将数据和文件提交到服务器。服务器在接收到POST请求后需要将其存储在服务器上的某个地方。Django默认的存储地址是相对于根目录的/media/文件夹，存储的默认文件名就是文件本来的名字。上传的文件如果不大于2.5MB，会先存入服务器内存中，然后再写入磁盘。如果上传的文件很大，Django会把文件先存入临时文件，再写入磁盘。

Django默认处理方式会出现一个问题，所有文件都存储在一个文件夹里。不同用户上传的有相同名字的文件可能会相互覆盖。另外用户还可能上传一些不安全的文件如js和exe文件，我们必需对允许上传文件的类型进行限制。因此我们在利用Django处理文件上传时必需考虑如下3个因素:

- 设置存储上传文件的文件夹地址
- 对上传文件进行重命名
- 对可接受的文件类型进行限制(表单验证)

本文将会讲解在Django示范代码中如何实现上述3个功能。

## Django文件上传的3种常见方式

Django文件上传一般有3种方式(如下所示)。我们会针对3种方式分别提供代码示范。

- 使用一般的表单上传，在视图中手动编写代码处理上传的文件
- 使用由模型创建的表单(ModelForm)上传，使用form.save()方法自动存储
- 使用Ajax实现文件异步上传，上传页面无需刷新即可显示新上传的文件

### **项目创建与设置**

我们先使用django-admin startproject命令创建一个叫file_project的项目，然后cd进入file_project, 使用python manage.py startapp创建一个叫file_upload的app。

我们首先需要将file_upload这个app加入到我们项目里，然后设置/media/和/STATIC_URL/文件夹。我们上传的文件都会放在/media/文件夹里。我们还需要使用css和js这些静态文件，所以需要设置STATIC_URL。

\#file_project/settings.py

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'file_upload',
]
```

\#file_project/settings.py

```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [os.path.join(BASE_DIR, "static"), ]

# specify media root for user uploaded files,
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
```

\#file_project/urls.py

```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('file/', include("file_upload.urls")),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### **创建模型**

使用Django上传文件创建模型不是必需，然而如果我们需要对上传文件进行系统化管理，模型还是很重要的。我们的File模型包括file和upload_method两个字段。我们通过upload_to选项指定了文件上传后存储的地址，并对上传的文件进行了重命名。如果你想了解如何自定义用户上传文件夹地址和对上传文件进行重命名，请阅读[这里](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMjM5OTMyODA4Nw%3D%3D%26mid%3D2247483887%26idx%3D2%26sn%3D3d8531c15ffda4faa32bde775bfd61fb%26chksm%3Da73c61d7904be8c1621603284549aa7eca6e144d8603e6759172d6b5a6b92a0bf6e08ab2d672%26scene%3D21%23wechat_redirect)。

\#file_upload/models.py

```python
from django.db import models
import os
import uuid

def user_directory_path(instance, filename):
    ext = filename.split('.')[-1]
    filename = '{}.{}'.format(uuid.uuid4().hex[:10], ext)
    return os.path.join("files", filename)


class File(models.Model):
    file = models.FileField(upload_to=user_directory_path, null=True)
    upload_method = models.CharField(max_length=20, verbose_name="Upload Method")
```

注意：

- 如果你不使用ModelForm，你还需要手动编写代码存储上传文件。

### **URLConf配置**

本项目一共包括5个urls, 分别对应普通表单上传，ModelForm上传和Ajax上传。还有两个urls，一个用来显示文件清单，一个专门处理ajax请求。

\#file_upload/urls.py

```python
from django.urls import re_path, path
from . import views

app_name = "file_upload"
urlpatterns = [

    # Upload File Without Using Model Form
    re_path(r'^upload1/$', views.file_upload, name='file_upload'),

    # Upload Files Using Model Form
    re_path(r'^upload2/$', views.model_form_upload, name='model_form_upload'),

    # Upload Files Using Ajax Form
    re_path(r'^upload3/$', views.ajax_form_upload, name='ajax_form_upload'),

    # Handling Ajax requests
    re_path(r'^ajax_upload/$', views.ajax_upload, name='ajax_upload'),

    # View File List
    path('', views.file_list, name='file_list'),
]
```

### 使用一般表单上传文件

我们先定义一个一般表单FileUploadForm，并通过clean方法对用户上传的文件进行验证，如果上传的文件名不以jpg, pdf或xlsx结尾，将显示表单验证错误信息。关于表单的自定义和验证更多内容见[Django基础(5): 表单forms的设计与使用](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMjM5OTMyODA4Nw%3D%3D%26mid%3D2247483813%26idx%3D1%26sn%3Dbda48ff63fd4cc3b2cc0de408e7a1fa1%26chksm%3Da73c619d904be88b13d9c27bc4afb5ca45c76473c5d56cf3b5a0f7f9cf7a57775168ff6150f1%26scene%3D21%23wechat_redirect)。

\#file_upload/forms.py

```python
from django import forms
from .models import File

class FileUploadForm(forms.Form):
    file = forms.FileField(widget=forms.ClearableFileInput(attrs={'class': 'form-control'}))
    upload_method = forms.CharField(label="Upload Method", max_length=20,
                                    widget=forms.TextInput(attrs={'class': 'form-control'}))

    def clean_file(self):
        file = self.cleaned_data['file']
        ext = file.name.split('.')[-1].lower()
        if ext not in ["jpg", "pdf", "xlsx"]:
            raise forms.ValidationError("Only jpg, pdf and xlsx files are allowed.")
        # return cleaned data is very important.
        return file
```

注意：

- 使用clean方法对表单字段进行验证时，别忘了return验证过的数据，即cleaned_data。只有返回了cleaned_data, 视图中才可以使用form.cleaned_data.get('xxx')获取验证过的数据。

对应一般文件上传的视图file_upload方法如下所示。当用户的请求方法为POST时，我们通过form.cleaned_data.get('file')获取通过验证的文件，并调用自定义的handle_uploaded_file方法来对文件进行重命名，写入文件。如果用户的请求方法不为POST，则渲染一个空的FileUploadForm在upload_form.html里。我们还定义了一个file_list方法来显示文件清单。

\#file_upload/views.py

```python
from django.shortcuts import render, redirect
from .models import File
from .forms import FileUploadForm, FileUploadModelForm
import os
import uuid
from django.http import JsonResponse
from django.template.defaultfilters import filesizeformat

def file_list(request):
    files = File.objects.all().order_by("-id")
    return render(request, 'file_upload/file_list.html', {'files': files})

# Regular file upload without using ModelForm
def file_upload(request):
    if request.method == "POST":
        form = FileUploadForm(request.POST, request.FILES)
        if form.is_valid():
            # get cleaned data
            upload_method = form.cleaned_data.get("upload_method")
            raw_file = form.cleaned_data.get("file")
            new_file = File()
            new_file.file = handle_uploaded_file(raw_file)
            new_file.upload_method = upload_method
            new_file.save()
            return redirect("/file/")
    else:
        form = FileUploadForm()

    return render(request, 'file_upload/upload_form.html', {'form': form,
                                                            'heading': 'Upload files with Regular Form'})


def handle_uploaded_file(file):
    ext = file.name.split('.')[-1]
    file_name = '{}.{}'.format(uuid.uuid4().hex[:10], ext)
    # file path relative to 'media' folder
    file_path = os.path.join('files', file_name)
    absolute_file_path = os.path.join('media', 'files', file_name)

    directory = os.path.dirname(absolute_file_path)
    if not os.path.exists(directory):
        os.makedirs(directory)

    with open(absolute_file_path, 'wb+') as destination:
        for chunk in file.chunks():
            destination.write(chunk)

    return file_path
```

注意：

- handle_uploaded_file方法里文件写入地址必需是包含/media/的绝对路径，如果/media/files/xxxx.jpg，而该方法返回的地址是相对于/media/文件夹的地址，如/files/xxx.jpg。存在数据中字段的是相对地址，而不是绝对地址。
- 构建文件写入绝对路径时请用os.path.join方法，因为不同系统文件夹分隔符不一样。
- 写入文件前一个良好的习惯是使用os.path.exists检查目标文件夹是否存在，如果不存在先创建文件夹，再写入。

上传表单模板upload_form.html代码如下。

\#file_upload/templates/upload_form.html

```django
{% extends "file_upload/base.html" %}
{% block content %}
{% if heading %}
<h3>{{ heading }}</h3>
{% endif %}
<form action="" method="post" enctype="multipart/form-data" >
  {% csrf_token %}
  {{ form.as_p }}
 <button class="btn btn-info form-control " type="submit" value="submit">Upload</button>
</form>
{% endblock %}
```

注意：

- 发送<form>必需有属性enctype="multipart/form-data"，否则表单不能发送文件，request.FILES为空。
- 我们的模板继承了base.html, 别忘了添加哦， 目的是为了显示更漂亮。

\#file_upload/templates/base.html

```django
{% load static %}

<html lang="en">
<head>
<title>{% block title %}Django File Upload and Download{% endblock %} </title>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
</head>

<body>
 <!-- Page content of course! -->
<main class="container-fluid">
<div class="container">

 {% block content %} {% endblock %}

</div> 
</main>

<!-- Bootstrap core JavaScript
================================================== -->

<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>

{% block js %} {% endblock %}

</body>
</html>
```

普通表单上传文件页面显示如下所示:

![img](https://pic3.zhimg.com/80/v2-24b5eecc14f021bae394102c0f902b5a_hd.jpg)

显示文件清单模板file_list.html代码如下所示:

\#file_upload/templates/file_list.html

```django
{% extends "file_upload/base.html" %}

{% block content %}
<h3>File List</h3>
<p> <a href="/file/upload1/">RegularFormUpload</a> | <a href="/file/upload2/">ModelFormUpload</a>
    | <a href="/file/upload3/">AjaxUpload</a></p>
{% if files %}
<table class="table table-striped">

    <tbody>
    <tr>
        <td>Filename & URL</td>
        <td>Filesize</td>
        <td>Upload Method</td>
    </tr>
    {% for file in files %}
    <tr>
        <td><a href="{{ file.file.url }}">{{ file.file.url }}</a></td>
        <td>{{ file.file.size | filesizeformat }}</td>
        <td>{{ file.upload_method }}</td>
    </tr>
    {% endfor %}
    </tbody>
</table>

{% else %}
<p>No files uploaded yet. Please click <a href="{% url 'file_upload:file_upload' %}">here</a>
    to upload files.</p>
{% endif %}

{% endblock %}
```

注意：

- 对于上传的文件我们可以调用file.url, file.name和file.size来查看上传文件的链接，地址和大小。
- 上传文件的大小默认是以B显示的，数字非常大。使用Django模板过滤器filesizeformat可以将文件大小显示为人们可读的方式，如MB，KB。

文件清单显示效果如下所示:

![img](https://pic1.zhimg.com/80/v2-b899cb8e457a73d119a5980f79812738_hd.jpg)

### 使用ModelForm上传文件

使用ModelForm上传是小编我推荐的上传方式，前提是你已经在模型中通过upload_to选项自定义了用户上传文件存储地址，并对文件进行了重命名。

我们首先要自定义自己的FileUploadModelForm，由模型重建的。代码如下所示:

\#file_upload/forms.py

```python
from django import forms
from .models import File

class FileUploadModelForm(forms.ModelForm):
    class Meta:
        model = File
        fields = ('file', 'upload_method',)

        widgets = {
            'upload_method': forms.TextInput(attrs={'class': 'form-control'}),
            'file': forms.ClearableFileInput(attrs={'class': 'form-control'}),
        }

    def clean_file(self):
        file = self.cleaned_data['file']
        ext = file.name.split('.')[-1].lower()
        if ext not in ["jpg", "pdf", "xlsx"]:
            raise forms.ValidationError("Only jpg, pdf and xlsx files are allowed.")
        # return cleaned data is very important.
        return file
```

使用ModelForm处理文件上传的视图model_form_upload方法非常简单，只需使用form.save()即可，无需再手动编写代码写入文件。

\#file_upload/views.py

```python
from django.shortcuts import render, redirect
from .models import File
from .forms import FileUploadForm, FileUploadModelForm
import os
import uuid
from django.http import JsonResponse
from django.template.defaultfilters import filesizeformat

# Upload File with ModelForm
def model_form_upload(request):
    if request.method == "POST":
        form = FileUploadModelForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect("/file/")
    else:
        form = FileUploadModelForm()
    return render(request, 'file_upload/upload_form.html', {'form': form,
                                                            'heading': 'Upload files with ModelForm'})
```

上传表单模板也是upload_form.html，和前例供用的。

\#file_upload/templates/upload_form.html

```django
{% extends "file_upload/base.html" %}
{% block content %}
{% if heading %}
<h3>{{ heading }}</h3>
{% endif %}
<form action="" method="post" enctype="multipart/form-data" >
  {% csrf_token %}
  {{ form.as_p }}
 <button class="btn btn-info form-control " type="submit" value="submit">Upload</button>
</form>
{% endblock %}
```

显示效果如下所示:

![img](Django如何上传处理文件及Ajax文件上传示范(附GitHub源码).assets/v2-de166d7a59b67bde7ea21e785fd8150a_hd.jpg)

### 使用Ajax上传文件

使用Ajax上传文件的好处是，你上传文件后无需刷新页面或跳转即可立刻显示新上传的文件信息(如下所示)。Ajax应用场景还是非常普遍的，比如用户上传头像后无需刷新实时显示新上传的头像。或则用户添加评论后无需刷新页面直接显示新增的评论。

![img](Django如何上传处理文件及Ajax文件上传示范(附GitHub源码).assets/v2-c17680ef9f4d45158567f7dc92537d61_hd.jpg)

AJAX文件上传代码最重要的部分在前端（代码如下所示)。我们构建了FormData对象，添加了file和upload_method, 并通过设置processData=False告诉jQuery不要处理上传的文件，交由后台处理。由于发送POST请求还需要提供csrftoken，我们还通过jQuery的cookie库获取crsftoken，添加到请求头里，一起发到服务器上。如果后台返回的data没有error_msg, 就显示后台返回的更新过的文件清单。处理ajax的请求地址是/file/ajax_upload/, 对应的视图方法是ajax_upload.

\#file_upload/templates/ajax_upload_form.html

```text
{% extends "file_upload/base.html" %}
{% block content %}

{% if heading %}
<h3>{{ heading }}</h3>
{% endif %}

<form action="" method="post" enctype="multipart/form-data" id="form">
    <ul class="errorlist"></ul>
    {% csrf_token %}
{{ form.as_p }}
 <input type="button" class="btn btn-info form-control" value="submit" id="btn" />
</form>
<table class="table table-striped" id="result">
</table>
{% endblock %}


{% block js %}
<script src=" https://cdn.jsdelivr.net/jquery.cookie/1.4.1/jquery.cookie.min.js ">
</script>
<script>
var csrftoken = $.cookie('csrftoken');
function csrfSafeMethod(method) {
    return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
}

$(document).ready(function(){
   $('#btn').click(function(e){
        e.preventDefault();
        // 构建FormData对象
      var form_data = new FormData();
        form_data.append('file', $('#id_file')[0].files[0]);
        form_data.append('upload_method', $('#id_upload_method').val());
        $.ajax({
        url: '/file/ajax_upload/',
        data: form_data,
        type: 'POST',
        dataType: 'json',
        // 告诉jQuery不要去处理发送的数据, 发送对象。
      processData : false,
        // 告诉jQuery不要去设置Content-Type请求头
      contentType : false,
        // 获取POST所需的csrftoken
        beforeSend: function(xhr, settings) {
            if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
            xhr.setRequestHeader("X-CSRFToken", csrftoken);
            }},
        success: function (data) {
            if(data['error_msg']) {
                var content = '<li>Only jpg, pdf and xlsx files are allowed.</li>';
                $('ul.errorlist').html(content);
            }
            else
            {
            var content= '<thead><tr>' +
            '<th>Name and URL</th>' +
            '<th>Size</th>' +
            '<th>Upload Method</th>' +
            '</tr></thead><tbody>';

             $.each(data, function(i, item) {
                  content = content +
                  '<tr><td>' +
                  "<a href= ' " +
                  item['url'] +
                  " '> " +
                  item['url'] +
                  '</a></td><td>' +
                  item['size'] +
                  '</td><td>' +
                  item['upload_method'] +
                   '</td><tr>'
                });
             content = content + "</tbody>";
             $('#result').html(content);
             }
           },
        });
   });
 });
  </script>
{% endblock %}
```

注意：

- Ajax代码部分代码请注意不要随意变动，尤其评论//部分要特别注意。

负责处理Ajax请求的视图ajax_upload方法如下所示。该方法将ajax发过来的数据于FileUploadModelForm先结合，然后直接调用form.save方法存储，最后以json格式返回更新过的文件清单。如何用户上传文件不符合要求，返回错误信息。

\#file_upload/views.py

```python
# Upload File with ModelForm
def ajax_form_upload(request):
    form = FileUploadModelForm()
    return render(request, 'file_upload/ajax_upload_form.html', {'form': form,
                                                            'heading': 'File Upload with AJAX'})

# handling AJAX requests
def ajax_upload(request):
    if request.method == "POST":
        form = FileUploadModelForm(data=request.POST, files=request.FILES)
        if form.is_valid():
            form.save()
            # Obtain the latest file list
            files = File.objects.all().order_by('-id')
            data = []
            for file in files:
                data.append({
                    "url": file.file.url,
                    "size": filesizeformat(file.file.size),
                    "upload_method": file.upload_method,
                    })
            return JsonResponse(data, safe=False)
        else:
            data = {'error_msg': "Only jpg, pdf and xlsx files are allowed."}
            return JsonResponse(data)
    return JsonResponse({'error_msg': 'only POST method accpeted.'})
```

**GitHub源码**

[https://github.com/shiyunbo/django-file-upload-download](https://link.zhihu.com/?target=https%3A//github.com/shiyunbo/django-file-upload-download)

**小结**

本文提供并解读了利用Django上传文件的3种主要方式(一般表单上传，ModelForm上传和Ajax上传)及示范代码。我们后续会专题讲解多文件上传和文件下载(如大文件下载), 欢迎关注我们的微信公众号。

## 文件下载的3种方法及文件私有化

下面我们就来讲解下如何利用Django处理文件下载，并谈下文件私有化的大概思路。项目相关配置方面参阅前面内容。

### 为什么需要编写下载视图方法?

你或许知道，我们上传的文件默认放在media文件夹中的，且Django会为每个上传的静态文件分配一个静态url。在模板中，你可以使用{{ mymodel.file.url }}获取每个文件的链接(url)，浏览器也是可以直接打开这个url的，如下所示。

<td><a href="/media/files/b1957d79f3.JPG/">/media/files/b1957d79f3.JPG</a></td>

然而当你碰到如下2种情况时，你需要编写自己的视图下载方法。

- 你希望用户以附件形式获得文件，而不是浏览器直接打开。
- 你希望允许用户下载一些保密文件，而不希望在html模板中暴露它们。

### 具体思路

我们先新建一个file_download的app，添加如下urls。该URL了包含了一个文件的相对路径file_path作为参数, 其对应视图是file_download方法。我们现在就开始尝试用不同方法来处理文件下载。

```python
from django.urls import path, re_path
from . import views

# namespace
app_name = 'file_download'

urlpatterns = [

    re_path(r'^download/(?P<file_path>.*)/$', views.file_download, name='file_download'),

]
```

模板templates/file_upload/file_list.html改为如下所示, 而不再采用option 1。

```django
{% for file in files %}
<tr>
    <!-- Option 1 <td><a href="{{ file.file.url }}/">{{ file.file.url }}</a></td> -->
    <td><a href="/file/download{{ file.file.url }}/">{{ file.file.url }}</a></td>
    <td>{{ file.file.size | filesizeformat }}</td>
    <td>{{ file.upload_method }}</td>
</tr>
{% endfor %}
```

### 方法一: 使用HttpResonse

下面方法从url获取file_path, 打开文件，读取文件，然后通过HttpResponse方法输出。

```python
import os
from django.http import HttpResponse

def file_download(request, file_path):
    # do something...
    with open(file_path) as f:
        c = f.read()
    return HttpResponse(c)
```

然而该方法有个问题，如果文件是个二进制文件，HttpResponse输出的将会是乱码。对于一些二进制文件(图片，pdf)，我们更希望其直接作为附件下载。当文件下载到本机后，用户就可以用自己喜欢的程序(如Adobe)打开阅读文件了。这时我们可以对上述方法做出如下改进， 给response设置content_type和Content_Disposition。

```python
import os
from django.http import HttpResponse, Http404

def media_file_download(request, file_path):
    with open(file_path, 'rb') as f:
        try:
            response = HttpResponse(f)
            response['content_type'] = "application/octet-stream"
            response['Content-Disposition'] = 'attachment; filename=' + os.path.basename(file_path)
            return response
        except Exception:
            raise Http404
```

HttpResponse有个很大的弊端，其工作原理是先读取文件，载入内存，然后再输出。如果下载文件很大，该方法会占用很多内存。对于下载大文件，Django更推荐StreamingHttpResponse和FileResponse方法，这两个方法将下载文件分批(Chunks)写入用户本地磁盘，先不将它们载入服务器内存。

### 方法二: 使用SteamingHttpResonse

```python
import os
from django.http import HttpResponse, Http404, StreamingHttpResponse

def stream_http_download(request, file_path):
    try:
        response = StreamingHttpResponse(open(file_path, 'rb'))
        response['content_type'] = "application/octet-stream"
        response['Content-Disposition'] = 'attachment; filename=' + os.path.basename(file_path)
        return response
    except Exception:
        raise Http404
```

### 方法三: 使用FileResonse

FileResponse方法是SteamingHttpResponse的子类，是小编我推荐的文件下载方法。如果我们给file_response_download加上@login_required装饰器，那么我们就可以实现用户需要先登录才能下载某些文件的功能了。

```python
import os
from django.http import HttpResponse, Http404, FileResponse

def file_response_download1(request, file_path):
    try:
        response = FileResponse(open(file_path, 'rb'))
        response['content_type'] = "application/octet-stream"
        response['Content-Disposition'] = 'attachment; filename=' + os.path.basename(file_path)
        return response
    except Exception:
        raise Http404
```

然而即使加上了@login_required的装饰器，用户只要获取了文件的链接地址, 他们依然可以通过浏览器直接访问那些文件。我们等会再谈保护文件的链接地址和文件私有化，因为此时我们还有个更大的问题需要担忧。我们定义的下载方法可以下载所有文件，不仅包括.py文件，还包括不在media文件夹里的文件(比如非用户上传的文件)。比如当我们直接访问127.0.0.1:8000/file/download/file_project/settings.py/时，你会发现我们连file_project目录下的settings.py都下载了。如果哪个程序员这么蠢，你可以将他直接fire了。所以我们在编写下载方法时，我们一定要限定那些文件可以下，哪些不能下或者限定用户只能下载media文件夹里的东西。

```python
def file_response_download(request, file_path):
    ext = os.path.basename(file_path).split('.')[-1].lower()
    # cannot be used to download py, db and sqlite3 files.
    if ext not in ['py', 'db',  'sqlite3']:
        response = FileResponse(open(file_path, 'rb'))
        response['content_type'] = "application/octet-stream"
        response['Content-Disposition'] = 'attachment; filename=' + os.path.basename(file_path)
        return response
    else:
        raise Http404
```

### 文件私有化的两种方法

如果你想实现只有登录过的用户才能查看和下载某些文件，大概有两种方法，这里仅提供思路。

- 上传文件放在media文件夹，文件名使用很长的随机字符串命名(uuid), 让用户无法根据文件名猜出这是什么文件。视图和模板里验证用户是否已登录，登录或通过权限验证后才显示具体的url。- 简单易实现，安全性不高，但对于一般项目已足够。
- 上传文件放在非media文件夹，用户即使知道了具体文件地址也无法访问，因为Django只会给media文件夹里每个文件创建独立url资源。视图和模板里验证用户是否已登录，登录或通过权限验证后通过自己编写的下载方法下载文件。- 安全性高，但实现相对复杂。

### GitHub源码

[https://github.com/shiyunbo/django-file-upload-download](https://link.zhihu.com/?target=https%3A//github.com/shiyunbo/django-file-upload-download)

### 小结

我们总结了何时需要自定义文件下载方法，并讲解了HttpResponse, StreamingHttpResponse和FileResponse的基本用法。我们还总结了使用自定义下载方法需要注意的事项(比如限定文件和文件夹)，最后谈了下如何实现文件私有化。



## 限制用户上传文件格式与大小

Django模型中自带的ImageField和FileField字段并不会也不能限制用户上传的图片或文件的格式和大小，这给Web APP开发带来了很大的安全隐患。

当然你可以通过[自定义form类中的clean的方法来添加对image或file字段进行验证](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMjM5OTMyODA4Nw%3D%3D%26mid%3D2247483813%26idx%3D1%26sn%3Dbda48ff63fd4cc3b2cc0de408e7a1fa1%26chksm%3Da73c619d904be88b13d9c27bc4afb5ca45c76473c5d56cf3b5a0f7f9cf7a57775168ff6150f1%26scene%3D21%23wechat_redirect)，从而限制上传文件格式和大小，然而这并不是最佳处理方法，因为这意味者每次你的模型里包含了图片或文件字段，你都要自定义forms类，并添加clean方法，从而造成大量代码重复。

**一个处理该问题的最佳方式是扩展Django的FileFiled字段，在创建模型时直接设置可以接受的文件类型，并限定可以上传的文件的最大尺寸**。

下面我们展示如何扩展Django的FileField字段，并展示如何在模型中使用它，从而以最佳方式限制用户上传文件的格式与大小。

### 扩展FileField字段

首先在你APP文件夹内新建fields.py, 并添加如下代码(来自stackoverflow)。新扩展的FileField叫RestrictedFileField，继承了FileField类，并包含了额外的两个可选参数: 可接受的内容类型content_types和max_upload_size最大上传尺寸(比如5242880=5MB)。

```python
from django.db.models import FileField
from django.forms import forms
from django.template.defaultfilters import filesizeformat

class RestrictedFileField(FileField):
    """ max_upload_size:
            2.5MB - 2621440
            5MB - 5242880
            10MB - 10485760
            20MB - 20971520
            50MB - 5242880
            100MB 104857600
            250MB - 214958080
            500MB - 429916160
    """
def __init__(self, *args, **kwargs):
        self.content_types = kwargs.pop("content_types", [])
        self.max_upload_size = kwargs.pop("max_upload_size", [])

        super().__init__(*args, **kwargs)

    def clean(self, *args, **kwargs):
        data = super().clean(*args, **kwargs)
        file = data.file

        try:
            content_type = file.content_type
            if content_type in self.content_types:
                if file.size > self.max_upload_size:
                    raise forms.ValidationError('Please keep filesize under {}. Current filesize {}'
.format(filesizeformat(self.max_upload_size), filesizeformat(file.size)))
            else:
                raise forms.ValidationError('This file type is not allowed.')
        except AttributeError:
            pass
        return data
```

### 如何使用扩展后的RestrictedFileField

首先你要从创建的fields.py里导入RestrictedFileField，然后在模型中按如下代码使用它。该字段可以用于文件，也可以用于图片。与ImageField和FileField相比，它多了content_types和max_upload_size选项，从此你再也不用担心用户上传文件时为所欲为啦。 这个字段是如此有用，说不定Django某一天会把它变成默认字段哦。小编我期待有那一天的到来。

```python
from django.db import models
from .fields import RestrictedFileField

class File(models.Model):
    file = RestrictedFileField(upload_to=user_directory_path, max_length=100,
content_types=['application/pdf', 'application/excel', 'application/msword',
'text/plain', 'text/csv', 'application/zip',
max_upload_size=5242880,)



class Image(models.Model):
    file = RestrictedFileField(upload_to=user_directory_path, max_length=100,
content_types=['image/jpeg', 'image/gif', 'image/gif', 'image/bmp', 'image/tiff'],
max_upload_size=5242880,)
```

注意：

用户上传文件还要考虑其它安全因素，比如不同用户上传了文件名相同的文件怎么办？用户上传文件的应该放在哪里比较好？更多内容见[Django自定义图片和文件上传路径(upload_to)的2种方式](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMjM5OTMyODA4Nw%3D%3D%26mid%3D2247483887%26idx%3D2%26sn%3D3d8531c15ffda4faa32bde775bfd61fb%26chksm%3Da73c61d7904be8c1621603284549aa7eca6e144d8603e6759172d6b5a6b92a0bf6e08ab2d672%26scene%3D21%23wechat_redirect)。

### 小结

重要的话可以说三遍。Django限制用户上传文件格式与大小的最佳处理方式是什么？当然是扩展Django原生的FileField类啦。该技术你get到了吗？
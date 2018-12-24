# Python Django 实现restful API



##### 最近在写测试平台，需要实现一个节点服务器的api,正好在用django，准备使用djangorestframework插件实现。

* * *

* 需求

    > 实现一个接口，在调用时，通过传递的参数，直接运行对应项目的自动化测试

* 环境

    > Python3.6 ,PyCharm,W7

* 项目结构
    ![这里写图片描述](https://img-blog.csdn.net/20171012212000849?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGllXzA3MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 功能实现

    * 流程
        ![处理流程](https://img-blog.csdn.net/20171012210553588?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGllXzA3MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
        我们要做的就是实现以上流程

    * 安装
        pip install djangorestframework
        pip install markdown
        pip install django-filter # Filtering support

    * 配置

        ```
        INSTALLED_APPS = (
            ...
            'rest_framework',
        )
        ```

    * 编写代码(本次代码不涉及数据库操作，只简单的写一个api)

        ①:打开AutoApi/Api/views.py 编写如下代码

    ```
    from django.http import JsonResponse, HttpResponseNotAllowed, HttpResponse
    from django.views.decorators.csrf import csrf_exempt
    from rest_framework.parsers import JSONParser
    from rest_framework import status

    @csrf_exempt
    def run_job(request):
        # 判断请求头是否为json
        if request.content_type != 'application/json':  
            # 如果不是的话，返回405
            return HttpResponse('only support json data', status=status.HTTP_415_UNSUPPORTED_MEDIA_TYPE)
        # 判断是否为post 请求
        if request.method == 'POST':
            try:
                # 解析请求的json格式入参
                data = JSONParser().parse(request)
            except Exception as why:
                print(why.args)
            else:
                content = {'msg': 'SUCCESS'}
                print(data)
                # 返回自定义请求内容content,200状态码
                return JsonResponse(data=content, status=status.HTTP_200_OK)
        # 如果不是post 请求返回不支持的请求方法
        return HttpResponseNotAllowed(permitted_methods=['POST'])
    ```

    ②:打开AutoApi/Api/urls.py 编写如下代码

    ```
    from django.conf.urls import url
    from Api import views

    urlpatterns = [
        url(r'^runJob/$',views.run_job),
    ]
    ```

    ③:打开AutoApi/AutoApi/urls.py 修改如下代码

    ```
    ALLOWED_HOSTS = '*' # 修改为* 代码允许任意host

    from django.conf.urls import url,include

    urlpatterns = [
        url(r'^admin/', admin.site.urls),
        url(r'^',include('Api.urls')),# 新增
    ]
    ```

    ④:启动服务

    ```
     python manage.py runserver 0.0.0.0:8080

    ```

    ![这里写图片描述](https://img-blog.csdn.net/20171012214429268?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGllXzA3MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

    ⑤:我们请求试试看

    ![这里写图片描述](https://img-blog.csdn.net/20171012215036214?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGllXzA3MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* * *

#### 以上就是简单的实现一个api ，其实开发说的接口就这么简单，没有那么神秘

```
接下来把post 的数据env ，project，cases 解析出来传给对应的自动化测试入口函数，就可以实现通过接口请求，启动自动化测试的目的。

```

后续

*   实现接口调用自动化测试项目
*   实现异步接口
*   实现定时任务
    .
    .
    .

**以上就是最近一些工作的总结，加油！**
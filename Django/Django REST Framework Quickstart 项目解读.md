# Django REST Framework Quickstart 项目解读

### 初衷

在2018年5月份接触并搭建过基于Django的后端平台，由于当时时间紧任务单一，很多细节也没去深究。但是我清除地记着我是有去找过Django REST相关的解读的，但是无果。最近有些时间，开始搭建一个较完整的基于Django的平台，结果发现“毒”还在，看来只能自制“解药”了。**因为离上一次写类似博客的东西已经很久了，所以语言组织已经退化，不求写清楚了，但求和我一样的入门者有一个可聊的话题。**

### 创建工程，添加一个应用

- 基于virtualenv创建工作环境，远离python版本管理的烦恼

```bash
virtualenv -p /usr/local/bin/python3 py3env
source py3env/bin/activate
pip install django
pip install djangorestframework
```

- 创建工程和应用

```bash
django-admin.py startproject tutorial .
cd tutorial
django-admin.py startapp quickstart
```

- 默认使用sql数据库初始化数据库设置

```
python manage.py migrate
```

- 创建管理员账号

```bash
python manage.py createsuperuser
```

### 关于Quickstart

这是一个官方的demo：<http://www.django-rest-framework.org/tutorial/quickstart/>， 目的是为了向开发者展示该组件的易用性，但是立足点似乎有点高，作为刚刚开始接触Django并想要了解REST组件的同学来说，这个例子的文档比较晦涩，而且绝大部分资料也只是对官方文档的翻译。**该demo的功能，是实现一套简单的api（含视图），用于管理员查看和编辑Django的用户和组信息。

### 了解流程

由于Django本身是MVC（MVT）结构，便于开发，但不便于入门理解，简单地说，就是不便于在入门时理解Demo代码中的关联性。

- Django的MTV模块

  - M：负责业务和数据库的关系映射
  - T：负责如何把html页面展示给用户
  - V：负责业务逻辑，调用M和T（MVC中C的角色）

- Django对请求的处理流程并不是仅仅MTV三个模块
![Django处理流程](https://upload-images.jianshu.io/upload_images/1354011-aa3ce9be160d20fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

  Django处理流程
  
  其实从WSGI以后，都算作Django的工作范围，WSGI通过调用Django中的Django.core.wsgi的入口方法（wgsi.py文件）以后，就将请求的数据结构传递给Django的各种中间件，具体会流经哪些中间件？可见setting.py中的MIDDLEWARE配置（跨域过滤之类的一般会设置在中间件中）
  
  
  
  Django app内部的流程会相对清晰一些：
  
  - URLconf是指urls.py选择一个视图来处理请求
  - 被选择的那个视图通常要做下面所列出的一件或者更多件事情： 
    - 通过model与数据库对话
    - 使用模板渲染HTML或者任何格式化过的响应
    - 返回一个纯文本响应（不被显示的）（API）
  - 返回response
  
  ### 额外的重点：序列化
  
  序列化和反序列化的目的就是对象和传输数据（或是存储数据）之间的转换。Django本身和Django REST 组件都提供了序列化的辅助类，目的是为了简化代码（可以想象一下JAVA里的序列化操作，虽然有工具可以生成代码），特别是ModelForm、ModelSerializer这些类，可以通过直接连接model模块，完成序列化和反序列化的工作。**序列化功能的实现，往往是实现功能的第一步。我们可以理解成数据流驱动了编码顺序。**
  
  - 序列化：对象 -> 字节序列（数据库或者response）
  - 反序列化：字节序列 -> 对象
     **从上面的描述上看，序列化功能类一般会用在连接model、return response，以及数据库操作上。** 
  
  ### 再看Demo代码
  
  1、真正的第一步应该是设计并建立数据库，而demo是要实现项目用户的操作，使用了自带的用户信息所在数据库。
   2、实现序列化功能
   在知道后端有什么（数据库），前端要什么之后，我们就可以设计序列化功能。因为数据库有什么我们就可以知道应该反序列出来什么，前端要什么，我们就知道需要序列化到response什么内容了。
   **Django提供了通用的serializers、ModelSerializer、HyperlinkedModelSerializer等封装等级不同和类型不同的序列化器辅助类，唯一的区别就是简化代码的程度不同**，如果对序列化的具体操作不熟悉的话，可以参照<https://darkcooking.gitbooks.io/django-rest-framework-cn/content/chapter1.html> 使用普通的serializers来实现一遍该类会帮助很大。
  
  ```python
  from django.contrib.auth.models import User, Group
  from rest_framework import serializers
  
  
  class UserSerializer(serializers.HyperlinkedModelSerializer):
  
      """
      用户信息的序列化器（根据前端需要的数据信息来组织结构）
      """
      class Meta:
          model = User
          fields = ('url', 'username', 'email', 'groups')
  
  
  class GroupSerializer(serializers.HyperlinkedModelSerializer):
  
      """
      组信息的序列化器
      """
      class Meta:
          model = Group
          fields = ('url', 'name')
  ```
  
  3、实现View
   这个Demo变得晦涩难懂的一大原因就是直接用了最简洁的代码方式，这里的viewsets就是其中之一，包括上面HyperlinkedModelSerializer。差别之处可以参考：<https://darkcooking.gitbooks.io/django-rest-framework-cn/content/chapter6.html>
  
  ```python
  """
  View中，必然会看到model（操作数据库）和序列化器（组织response内容）
  """
  from django.contrib.auth.models import User, Group
  from rest_framework import viewsets
  from quickstart.serializers import UserSerializer, GroupSerializer
  
  
  class UserViewSet(viewsets.ModelViewSet):
  
      """
      处理查看、编辑用户的界面
      """
      queryset = User.objects.all().order_by('-date_joined')
      serializer_class = UserSerializer
  
  
  class GroupViewSet(viewsets.ModelViewSet):
  
      """
      处理查看、编辑用户组的界面
      """
      queryset = Group.objects.all()
      serializer_class = GroupSerializer
  ```
  
  #### 为了理解这些晦涩的用法，还是总结下，Django Rest对视图写法的精简可以一步步归纳为：
  
  - 1、使用常规方式实现，最显式的实现方式，也是最好理解的方式：（读取数据库字节序列->反序列化->序列化后response）
  
  ```python
  @csrf_exempt
  def snippet_list(request):
      """
      展示所以snippets,或创建新的snippet.
      """
      if request.method == 'GET':
          snippets = Snippet.objects.all()
          serializer = SnippetSerializer(snippets, many=True)
          return JSONResponse(serializer.data)
      elif request.method == 'POST':
          data = JSONParser().parse(request)
          serializer = SnippetSerializer(data=data)
          if serializer.is_valid():
              serializer.save()
              return JSONResponse(serializer.data, status=201)
          return JSONResponse(serializer.errors, status=400)
  ```
  
  - 2、使用基于函数视图的@api_view装饰器，加快实现
  
  ```python
  @api_view(['GET', 'POST'])
  def snippet_list(request):
      """
      展示或创建snippets.
      """
      if request.method == 'GET':
          snippets = Snippet.objects.all()
          serializer = SnippetSerializer(snippets, many=True)
          return Response(serializer.data)
      elif request.method == 'POST':
          serializer = SnippetSerializer(data=request.data)
          if serializer.is_valid():
              serializer.save()
              return Response(serializer.data, status=status.HTTP_201_CREATED)
          return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
  ```
  
  - 3、使用基于类视图的APIView
  
  ```python
  class SnippetList(APIView):
      """
      List all snippets, or create a new snippet.
      """
      def get(self, request, format=None):
          snippets = Snippet.objects.all()
          serializer = SnippetSerializer(snippets, many=True)
          return Response(serializer.data)
      def post(self, request, format=None):
          serializer = SnippetSerializer(data=request.data)
          if serializer.is_valid():
              serializer.save()
              return Response(serializer.data, status=status.HTTP_201_CREATED)
          return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
  ```
  
  - 4、基于ViewSets的抽象行为（配合Routers实现），该方法也可以简化urlconf，所以这可能就是Django生态的好处，让我们更加关注api的交互，但是这之前还是需要清楚这些方法的演变过程。从这个角度上看，读了N遍官方文档之后，能发现官方真正想要传达的意思。
  
  ```python
  class SnippetViewSet(viewsets.ModelViewSet):
      """
      This viewset automatically provides `list`, `create`, `retrieve`,
      `update` and `destroy` actions.
      Additionally we also provide an extra `highlight` action.
      """
      queryset = Snippet.objects.all()
      serializer_class = SnippetSerializer
      permission_classes = (permissions.IsAuthenticatedOrReadOnly,
                            IsOwnerOrReadOnly,)
      @detail_route(renderer_classes=[renderers.StaticHTMLRenderer])
      def highlight(self, request, *args, **kwargs):
          snippet = self.get_object()
          return Response(snippet.highlighted)
      def perform_create(self, serializer):
          serializer.save(owner=self.request.user)
  ```
  
  4、实现urlpatterns
  
  使用viewsets和router后，由于封装处理了“get”、“post” 等默认动作，所以在url.py指出特定的uri便可，这就是所谓的RESTFul，因为所有的“动作”都没有暴露到api的命名中。
  
  ```python
  from django.contrib import admin
  from django.urls import path
  from django.conf.urls import url, include
  from rest_framework import routers
  from quickstart import views
  
  router = routers.DefaultRouter()
  router.register(r'users', views.UserViewSet)
  router.register(r'groups', views.GroupViewSet)
  
  # 使用URL路由来管理我们的API
  # 另外添加登录相关的URL
  urlpatterns = [
      url(r'^', include(router.urls)),
      url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework')),
      path('admin/', admin.site.urls),
  ]
  ```
  
  5、运行测试。
  
  ```
  python manage.py runserver
  ```
  
  便可看到一套自带界面的api，一开始可能会有疑问，**说好是写一套apis的，怎么成了一个界面了，其实就是Django MTV的福利（自带doc有没有）**，大家试着去访问：<http://127.0.0.1:8000/users/?format=json> ，就能看到一个正常的接口返回了。
  
  ![运行结果](https://upload-images.jianshu.io/upload_images/1354011-a53167823ec42352.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
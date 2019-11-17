# 让你的Django应用变DRY的几个最佳实践

Django+Django REST Framework下的DRY实践

> 目前在Python的Web框架中被应用最广泛的就是Django和Django REST Framework. 这两种框架都提供了非常健壮的功能，能满足Web开发的各个方面。DRY是Don't-Repeat-Yourself的缩写，是一种代码编写的原则，即不要重复自己的工作。我个人有些代码洁癖，凡是发现我需要复制粘贴代码的地方，就想着能怎样去除重复的工作。在日常的开发中也总结出了一些个人的实践，分享给大家。

总的来说，要使得你的应用很DRY，要遵循以下两个原则：

- 全局都应用的变更，收拢到一个地方配置
- 有少数与其他不一样的行为，将多数行为定义为全局行为，将少数行为分别配置，并尽可能简化配置方法。

Django和Django REST framework（后简称DRF）提供了海量的全局配置、局部配置，来实现上述思想，但配置项太多了，有时人们往往不知道该如何利用。

## 一、用户鉴权

### 1. Django的配置`AUTHENTICATION_BACKENDS`

`AUTHENTICATION_BACKENDS`控制了应用根据传入的参数校验用户是否属于合法用户（用户名是否存在？密码是否正确？）。使用时通过`django.contrib.auth.authenticate`函数，传入想要的参数，该函数会自动选择对应的后端进行用户校验，常用的校验方式有数据库校验、配置文件校验、LDAP校验等等。如果你想接入第三方登录，OAuth登录，都应该自定义一个Backend，无需继承任何基类，只需实现一个authenticate方法，该方法参数与`django.contrib.auth.authenticate`的传入参数相同，返回一个用户对象，然后将这个Backend添加到`AUTHENTICATION_BACKENDS`就可以了。

**注意：**在使用到用户模型的时候，要使用`django.contrib.auth.get_user_model()`而不是导入具体的model类，这样可以方便用`AUTH_USER_MODEL`配置去改变用户模型。

```python
class PowerOAuthBackend:
    """请求Power单点登录后跳转的验证"""

    def authenticate(self, request, user=None, password=None):
        if check_user_password(user, password):
            # 返回用户对象
            return get_user_model().get(username=user)
        else:
            # 用户名密码错误 403
            raise PermissionDenied()
   
	def get_user(self, user_id):
        # 若通过浏览器访问则需要定义次方法，获取已登录的用户对象
        # 若只有RESTful调用则跳过
        return get_user_model().objects.get(staff_id=user_id)

# 登录
def login_view(request):
    username = request.POST.get('user')
    password = request.POST.get('password')
    user = authenticate(user=username, password=password)
    # 将用户存入会话
    login(request, user)
    return redirect('/')
```

### 2. DRF的配置 `DEFAULT_AUTHENTICATION_CLASSES`

`DEFAULT_AUTHENTICATION_CLASSES`，以及针对每个`APIView`配置的`authentication_classes`，是对RESTful请求的身份验证，通过分析请求带的身份信息判断来源方的身份，一般有以下几种方式：

- 会话鉴权（登录态）
- BasicAuth鉴权
- Token鉴权

这些类都包含在`rest_framework.authentication`模块中。如果你要通过智能网关转发后端请求，则需要写一个Authentication类，继承自`rest_framework.authentication.BaseAuthentication`类，其中有两个比较重要的方法，函数签名及说明如下：

```python
class MyAuthentication(BaseAuthentication):
    def authenticate(self, request):
        # 若鉴权成功，则返回一个(user, auth)的元组
        return user, auth
        # 否则，若想交给后面的authentication处理，则返回None
        return None
        # 否则抛出401错误
        raise rest_framework.exceptions.AuthenticationFailed()
	
    def authenticate_header(self, request):
        # DRF会选择第一顺位的Authentication的此方法返回的结果作为WWW-Authentication头
        # 如果返回为空则会将401错误转换成403错误
        return 'OMS'
```

### 3. DRF的`DEFAULT_PERMISSION_CLASSES`

如果说Authentication是判断「你是谁」，那么Authorization就是判断「你能做什么」，就好比你进入公司大楼需要用工卡（Authentication），但你有了工卡也不能随便去总裁办公室（Authorization）。在DRF中完成Authorization工作的就是`DEFAULT_PERMISSION_CLASSES`配置项，以及针对每个`APIView`配置的`permission_classes`，他是用来精确控制请求放对某一资源有无权限。在RESTful规范中，无鉴权信息是401错误而无权限是403错误。在[DRF的官方文档](https://www.django-rest-framework.org/tutorial/4-authentication-and-permissions/)中有详细例子这里就不再赘述。

## 二、自定义响应体

很多时候（如前端框架、开发SDK）对响应体的格式是有要求的，我看到大多数的实现只是用一个格式化的类去填充响应信息，但这种方法有两个缺点：

1. 每次需要人为构造响应
2. 无法适用于DRF的`ModelViewSet`，因为它自带的方法的响应是默认的，如果要挨个重载就无法利用到`ModelViewSet`的懒人特性

所以我们需要将这种格式自定义收拢到一处，做到使用时无感知，响应自动形成期望的格式。要达成这种效果，大致有两种途径：

1. 写自定义中间件，修改响应格式
2. 写自定义renderer

这里第一种途径有几处劣势：

1. 在中间件处理时`rest_framework.response.Response`已完成渲染，修改内部数据不起作用
2. 若重新构造一个`rest_framework.response.Response`则会报未渲染错误，而渲染过程比较复杂
3. 若选择用`django.http.response.JSONResponse`重新构造响应则放弃了DRF的自动渲染特性

我对这些缺陷不能忍，于是想到了第二种途径，也就是自定义renderer，它有以下好处：

1. 即可全局生效（`DEFAULT_RENDERER_CLASSES`），又可针对单个`APIView`生效，非常灵活
2. 保留了DRF的智能渲染特性，即浏览器请求渲染HTML页面，后端请求渲染JSON响应

DRF的默认renderer有两个：`rest_framework.renderers.JSONRenderer`和`rest_framework.renderers.BrowsableAPIRenderer`。这里可以按需重载，如果浏览器和后端响应都需要，则都重载，如果只需要JSON响应，则重载第一个就可以了，这里两个类的重载点不一样：

```python
class JSONRenderer(renderers.JSONRenderer):

    def render(self, data, accepted_media_type=None, renderer_context=None):
        request = renderer_context['request']
        # 在此处修改data
        return super().render(data, accepted_media_type=accepted_media_type, renderer_context=renderer_context)


class BrowsableAPIRenderer(renderers.BrowsableAPIRenderer):

    def get_content(self, renderer, data,
                    accepted_media_type, renderer_context):
        request = renderer_context['request']
        # 在此处修改data
        return super().get_content(renderer, data,
                                   accepted_media_type, renderer_context)
```

## 三、异常处理

我们经常会需要抛出异常，有些是主动抛出、有些是未捕获的异常，在这些情况下，我们都希望日志记录异常的堆栈信息，然后返回一个规范的响应（格式与上一节中一致），这样我们就需要更改异常处理。在Django+DRF中异常处理有两个重载点：

1. 中间件中的`process_exception`函数
2. DRF的`EXCEPTION_HANDLER`配置

而其中`EXCEPTION_HANDLER`的作用时间早于中间件，这就导致了有些DRF内置的异常，在到达中间件之前已经渲染为正常的响应了，这明显不是我们期望的效果，所以我们选择第二个重载点。

```python
def exception_handler(exc, context):
    # copy自DRF默认exception_handler
    if isinstance(exc, Http404):
        exc = exceptions.NotFound()
    elif isinstance(exc, PermissionDenied):
        exc = exceptions.PermissionDenied()

    if isinstance(exc, exceptions.APIException):
        # DRF内置异常
        headers = {}
        if getattr(exc, 'auth_header', None):
            headers['WWW-Authenticate'] = exc.auth_header
        if getattr(exc, 'wait', None):
            headers['Retry-After'] = '%d' % exc.wait

        if isinstance(exc.detail, (list, dict)):
            body = exc.detail
            message = str(exc)
        else:
            body = {}
            message = str(exc.detail)
		# copy结束
        # 组装响应体
        return Response({...})

    else:
        # 其他未捕获异常
        logger.error(traceback.format_exc())
        if not isinstance(exc, ApiError):
            exc = ApiError(str(exc))
        # 组装响应体
        return exc.as_response()
```

美中不足的是有一大段的代码是从DRF默认的异常处理函数copy过来的，这是DRF为数不多的不合理设计，留了一个配置项供你改变默认行为，但却没有留出一个好的重载点。

## 总结

DRY原则能使你的代码结构好、易维护、易扩展。在日常的开发中，要时刻反思自己的代码是否过于重复，可以精简。在Python中，可以说只要你想，一定能把多处一样的代码给抽取出来。只是有时候为了抽出这些代码，又产生了很多额外的代码，这是需要取舍的。相信本文中提到的三个大方向，能对你有所启发。
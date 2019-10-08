# Django - 单元测试

单元测试是软件开发中一个重要的质量保障手段。

通过单元测试，你可以「测试先行」，将 TDD 落地；你也可以在重构代码时保证原有的逻辑不受影响。

在 Django 官方文档的「[测试](https://link.juejin.im/?target=https%3A%2F%2Fdocs.djangoproject.com%2Fzh-hans%2F2.2%2Ftopics%2Ftesting%2F)」一章中，已经比较详尽地介绍了如何完成单元测试，本文的目的在于“**以尽量小的篇幅介绍如何编写一个单元测试基础类**”，加上一些高级用法（如 `Mock` ）的实践，让写单测变得简单而非煎熬。

## 官方文档

### 基础用法

单元测试通过调用一个方法（执行），判断这个方法执行后产生的作用是否与预期相符（断言），都是执行并判断结果的过程。

那么下面的代码就比较好理解了：

```
from django.test import TestCase
from myapp.models import Animal

class AnimalTestCase(TestCase):
    def test_animals_can_speak(self):
        """Animals that can speak are correctly identified"""
        Animal.objects.create(name="lion", sound="roar")
        Animal.objects.create(name="cat", sound="meow")
        lion = Animal.objects.get(name="lion")
        cat = Animal.objects.get(name="cat")
        self.assertEqual(lion.speak(), 'The lion says "roar"')
        self.assertEqual(cat.speak(), 'The cat says "meow"')
```

> 示例来自 Django 官网

在以 `test_` 开头的测试方法中，通过调用函数，然后使用 `assert` 方法判定结果。

如果还有环境准备以及测试数据回收的过程，那么就可以使用 `setUp` 和 `tearDown` 方法进行处理：

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
from django.test import TestCase
from myapp.models import Animal

class AnimalTestCase(TestCase):
    def setUp(self):
        Animal.objects.create(name="lion", sound="roar")
        Animal.objects.create(name="cat", sound="meow")
        
    def test_animals_can_speak(self):
        """Animals that can speak are correctly identified"""
        lion = Animal.objects.get(name="lion")
        cat = Animal.objects.get(name="cat")
        self.assertEqual(lion.speak(), 'The lion says "roar"')
        self.assertEqual(cat.speak(), 'The cat says "meow"')
    
    def tearDown(self):
        Animal.objects.filter(name__in=["lion", "cat"]).delete()
```

#### 运行测试

```
python manage.py test
```

在运行单测时，添加 `--keepdb` 参数，来避免每次执行单测时需要重建数据库的问题，提高执行速度：

```
python manage.py test --keepdb
```

### 接口测试

对于接口的测试，通常 Web 框架自身都会集成「测试套件」，通过模拟请求的方式来执行单元测试用例。Django 已经实现了一个 `RequestFactory` 类，可以直接用它来发送请求：

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
from django.contrib.auth.models import User
from django.test import RequestFactory, TestCase

from .views import MyView

class SimpleTest(TestCase):
    def setUp(self):
        # Every test needs access to the request factory.
        self.factory = RequestFactory()
        self.user = User.objects.create_user(
            username='jacob', email='jacob@…', password='top_secret')

    def test_details(self):
        # Create an instance of a GET request.
        request = self.factory.get('/customer/details')
        # Recall that middleware are not supported. You can simulate a
        # logged-in user by setting request.user manually.
        request.user = self.user
        # Use this syntax for class-based views.
        response = MyView.as_view()(request)
        self.assertEqual(response.status_code, 200)
```

#### DRF 中的 APITestCase

可以看到，构造请求的方法还是比较麻烦，每个用例中我们都需要先初始化一个 `RequestFactory` 对象，调用的时候（`response = MyView.as_view()(request)`）也不够直观。

在 `Djang Rest Framework` 中就解决了这个问题，可以直接通过 `APITestCase` 中的 `self.client` 发送请求：

```
from django.urls import reverse
from rest_framework import status
from rest_framework.test import APITestCase
from myproject.apps.core.models import Account

class AccountTests(APITestCase):
    def test_create_account(self):
        """
        Ensure we can create a new account object.
        """
        url = reverse('account-list')
        data = {'name': 'DabApps'}
        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Account.objects.count(), 1)
        self.assertEqual(Account.objects.get().name, 'DabApps')
```

> 示例来自 DRF 官网

## 解决与优化问题

那么我们需要考虑一下，Django 与 DRF 这些方法是否都满足我们的需求？是否有更简单的实现方法？

### 问题1. 如何模拟一个用户登录状态

我们的项目有对用户进行权限控制，那么首先遇到的问题就是“如何模拟一个用户登录状态”，使得权限相关的逻辑不会出现错误。

在 Django 的示例中，由于先构造 request，再显式地将 `request.user` 配置为创建好的 `User` 对象来实现，但在 DRF 中，由于封装了构造 request 的过程，无法再使用这种方法，这是需要解决的问题。

#### 解决方法

在解决这个问题时，我在 GitHub 上搜索了 Star 数较多的 Django 项目，学习了不同的项目如何优化单元测试。

其中 [Sentry](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fgetsentry%2Fsentry) 封装了一个 [login_as](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fgetsentry%2Fsentry%2Fblob%2Fmaster%2Ftests%2Fsentry%2Fapi%2Fendpoints%2Ftest_auth_index.py%23L14) 方法，通过在当前的 session 中添加用户信息来绕过用户登录。

根据这个思路，我简化了 Sentry 的实现代码，得到下面的方法：

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
from django.conf import settings
from django.contrib.auth import login
from django.contrib.auth.models import AnonymousUser
from django.utils.functional import cached_property
from django.http import HttpRequest
from rest_framework.test import APITestCase as BaseAPITestCase

class APITestCase(BaseAPITestCase):
    @staticmethod
    def create_session():
        engine = import_module(settings.SESSION_ENGINE)
        session = engine.SessionStore()
        session.save()
        return session
        
    @cached_property
    def session(self):
        return self.create_session()

    def save_session(self):
        self.session.save()
        self.save_cookie(
            name=settings.SESSION_COOKIE_NAME,
            value=self.session.session_key,
            expires=None
        )

    def save_cookie(self, name, value, **params):
        self.client.cookies[name] = value
        self.client.cookies[name].update({
            k.replace('_', '-'): v
            for k, v in params.items()
        })

    def login(self, user):
        """登录用户，用于通过权限校验"""
        user.backend = settings.AUTHENTICATION_BACKENDS[0]
        request = self.make_request()
        login(request, user)
        request.user = user
        self.save_session()

    def make_request(self, user=None, auth=None, method=None):
        request = HttpRequest()
        if method:
            request.method = method
        request.META['REMOTE_ADDR'] = '127.0.0.1'
        request.META['SERVER_NAME'] = 'testserver'
        request.META['SERVER_PORT'] = 80
        request.REQUEST = {}

        # order matters here, session -> user -> other things
        request.session = self.session
        request.auth = auth
        request.user = user or AnonymousUser()
        request.is_superuser = lambda: request.user.is_superuser
        request.successful_authenticator = None
        return request
```

在发送请求前，通过调用 `login` 方法将模拟登录后的 Session 保存，使得调用 `self.client` 发送请求时带上 SessionId，来达到登录效果：

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
class MyViewTest(APITestCase):
    def setUp(self) -> None:
        self.user = self.create_user(is_staff=True)
        self.login(self.user)
        
    def test_get_myview_details(self) -> None:
        # 假设 /api/myview 只有在 is_staff 用户登录情况下才可请求
        response = self.client.get(path=/api/myview)
        # status_code 不为 401，说明用户已经登录
        self.assertEqual(response.status_code, 200)
```

### 问题2. 如何便捷地初始化准备数据

接口逻辑代码通常依赖已有的数据进行读写操作，“初始化准备数据”需要变得更灵活，减少在本测试用例下不关心的属性的初始化，同时还要保持用例执行的操作清晰。

#### 写一些带默认值的 create_ 方法

在单元测试中初始化一些数据的时候，我们通常只是希望对某一些字段进行配置，因此可以将所有数据模型创建数据的方法封装至测试套件中，并对所有字段自动添加默认值：

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
from django.contrib.auth.models import User
from uuid import uuid4
from rest_framework.test import APITestCase as BaseAPITestCase

class APITestCase(BaseAPITestCase):
    @staticmethod
    def create_user(username=None, **kwargs):
        if username is None:
            username = uuid4().hex

        return User.objects.create_user(username=username, **kwargs)
```

更好的方式是抽取一个类，专门用于处理初始化数据，这样的代码看起来将会更加美观，也更容易维护：

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
from rest_framework.test import APITestCase as BaseAPITestCase

class Factories(object):
    @staticmethod
    def create_user(username=None, **kwargs):
        ...
       
    @staticmethod 
    def create_task(task_name=None, **kwargs):
        ...
        
class APITestCase(Factories, BaseAPITestCase):
    pass
```

#### 对 URL 的处理

URL 可以是写路径格式，但是万一路径更改了，维护起来比较麻烦。

Django 中的路由支持通过 Endpoint 反向查找路径：

```
>>>from django.urls import reverse
>>>reverse("app_label.endpoint")
/api/my-endpoint
```

我们在 `APITestCase` 类中添加 `app_label` 与 `endpoint` 两个属性，提供 `get_url` 方便调用：

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
class APITestCase(Factories, BaseAPITestCase):
    # django App 名
    app_label = 'my_app'
    # 端点，用于标识 URL
    endpoint = None
    
    def get_url(self, *args, **kwargs):
        return reverse(f"{self.app_label}:{self.endpoint}", args=args, kwargs=kwargs)
```

在测试用例中：

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
class TaskDetailTest(APITestCase):
    endpoint = 'task-detail'

    def setUp(self) -> None:
        self.url = self.get_url(task_id=self.task.pk)
        ...
    
    def test_get_task_details(self):
        result = self.client.get(self.url)
        ...
```

### 问题3. 如何在不启动服务的情况下执行异步任务

再就是我们使用celery执行异步任务，“如何在不启动服务的情况下执行异步任务”？

#### 解决方法1：异步代码同步执行

通过修改 Celery 的配置即可：

```
class MyTest(TestCase):

    def setUP(self):
        celery.conf.update(CELERY_ALWAYS_EAGER=True)
```

修改后，异步任务将被同步执行，也就不需要启动 Celery Worker与 RabbitMQ 等服务了。以下两者是等效的：

```
add.delay(2, 2)
add(2, 2)
```

但是这种处理方法有个坏处，即测试用例的数量将会变为 Celery 任务的分支数 * 主任务的条件分支数，发生倍数增长。

因此需要借助 Mock 来减少交叉覆盖场景。

#### 解决方法2：Mock

假设代码如下：

*tasks.py*

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
@celery_task
def add(x, y):
    print(x + y)
    
def main_func():
    x, y = do_something()
    add.delay(x, y)
```

那么可以用 `Mock` 方法将异步任务的 `delay` 跳过，达到只测试 `main_func` 方法中的其他代码； 因为跳过了 `add` 方法，那么还需额为对 `add` 方法进行测试即可：

*test.py*

```
"""
@Author: Shin Yang
@WeChat: shin-devops
"""
from unittest import mock
from tasks import add, main_func

class MyTest(TestCase):

    @mock.patch("tasks.add.delay")
    def test_main_func(self, mocked_delay):
        mocked_delay.return_value = None
        result = main_func()
        mocked_delay.assert_called_with(1, 2)
        self.assertEqual(result, my_expect1)
        
    def test_add(self):
        result = add()
        self.assertEqual(result, my_expect2)
```

## 总结

再回到开头，“单元测试是软件开发中一个重要的质量保障手段”，但是很多公司/团队/开发人员为了肉眼可见的「产出」而忽视单测。

单元测试属于长线投资，在开发需求时编写单元测试可能觉得写了两份代码，但是实际上，对公司而言，有单测的代码能够降低维护代码过程中产生风险的概率，以及为了解决这些风险而造成的损失；对开发人员而言，质量意识与测试思维能够让你在开发时考虑更多代码的可维护性、可测试性，写出更优质的代码。

## 参考

- [docs.djangoproject.com/zh-hans/2.2…](https://link.juejin.im/?target=https%3A%2F%2Fdocs.djangoproject.com%2Fzh-hans%2F2.2%2Ftopics%2Ftesting%2F)
- [www.django-rest-framework.org/api-guide/t…](https://link.juejin.im/?target=https%3A%2F%2Fwww.django-rest-framework.org%2Fapi-guide%2Ftesting%2F%23testing)
- [github.com/getsentry/s…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fgetsentry%2Fsentry)

> 版权声明：作者「严北(微信`shin-devops`)」，发布于「[掘金](https://juejin.im/user/5a9ea689f265da23906b89e1)」，未经授权禁止转载！
# Django - 集成CAS单点登录

## CAS 简介

CAS 全称集中式认证服务(Central Authentication Service)，是实现单点登录(SSO)的一中手段。

CAS 的通讯流程图如下(图片来自Google图库)：

![CAS通讯流程图](Django - 集成CAS单点登录.assets/16b378ad5932473a)

对于本文用户可感知的层面，认证过程如下：

1. 前端访问后端登录接口
2. 后端返回重定向到 CAS 服务器的登录页面，并携带当前用户访问的网页链接
3. 用户登录，浏览器发送请求到 CAS 服务器进行认证
4. CAS 认证通过，将本次登录保存到会话，返回回调地址给后端
5. 后端返回重定向请求给前端
6. 前端重定向到跳转登录前的页面

> 中间涉及到的 TGT 处理逻辑已经由开源 CAS Client(python-cas) 实现。

要注意，CAS 服务器本身有一些过滤条件，例如域名白名单等，因此接入的时候需要将新系统的域名或 IP 加入 CAS 服务端配置中。

出于安全考虑，CAS 一般不支持跨域，因此前后端分离开发时可能比较麻烦。（似乎有解决方案，但是未尝试过）

## 接入 CAS

因为是第一次接触 CAS ，为了方便调试，我在本地直接启动一个 CAS 服务端用于调试。

CAS 客户端也就是集成于我们实际开发的Django代码中。

### CAS 服务端

GitHub 中有很多 CAS 项目，我选了一个基于 Django 的 `django-mama-cas` 应用。

#### 配置

创建 `django-cas-server` 项目：

```
django-admin startproject django-cas-server
```

安装 `django-mama-cas` 依赖：

```
pip install django-mama-cas
```

在 `INSTALLED_APPS` 中添加 `'mama_cas'` 应用：

*settings.py*

```
INSTALLED_APPS = [
    ...
    'mama_cas',
]
```

添加 `mama_cas` 应用中的路由：

*urls.py*

```
urlpatterns += [url(r'', include('mama_cas.urls'))]
```

配置 CAS 信息：

```
MAMA_CAS_SERVICES = [
    {
        # 必填项，此项为**Client** IP:Port，相当于白名单
        'SERVICE': 'http://127.0.0.1:8000',
        # 回调模式，具体参考官方文档
        'CALLBACKS': [
            'mama_cas.callbacks.user_model_attributes',
        ],
    },
]
```

#### 使用

```
# 使用任意端口都可，此处我使用 30000
python manage.py runserver 0.0.0.0:30000
```

服务启动后，可以访问 `http://0.0.0.0:30000/login` 到达 CAS 登录页面。

问题来了，用户名密码是什么呢？

我着实花了点时间才解决这个问题————`django-mama-cas` 默认使用的是 `django.auth` 模块 `User`，使用 `django-admin` 创建超级用户，该用户也就可以用于登录 CAS ：

```bash
python manage.py createsuperuser
```

输入用户密码即完成超级用户创建，接着使用这个用户登录即可。

### CAS 客户端

Python 有开源的 CAS 客户端 `python-cas`，由于我使用的 Django 开发后端，因此直接选用封装好 `python-cas` 的 Django 应用 `django-cas-ng` 。

#### 配置

同样需要先安装依赖：

```
pip install django-cas-ng
```

在 `settings.py` 中的 `INSTALLED_APPS` 和 `AUTHENTICATION_BACKENDS` 两处添加 `django-cas-ng` 的配置：

*settings.py*

```
INSTALLED_APPS = (
    # ... other installed apps
    'django_cas_ng',
)

AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'django_cas_ng.backends.CASBackend',
)
```

同时参考准备接入的 CAS 地址和版本，添加几个对应的配置：

```
# CAS 的地址
CAS_SERVER_URL = 'http://127.0.0.1:30000'
# CAS 版本
CAS_VERSION = '3'
# 存入所有 CAS 服务端返回的 User 数据。
CAS_APPLY_ATTRIBUTES_TO_USER = True
```

添加登入登出的路由（这两部分逻辑已由 `django-cas-ng` 完成，可以直接使用。如果需要扩展，可以参照源码自己实现即可）：

*urls.py*

```
import django_cas_ng.views as cas_views
urlpatterns = [
    ...
    path('login/', django_cas_ng.views.LoginView.as_view(), name='cas_ng_login'),
    path('logout/', django_cas_ng.views.LogoutView.as_view(), name='cas_ng_logout'),
]
```

#### 调试

启动当前服务：

```
python manage.py runserver
```

访问 `http://127.0.0.1:8000/login`，网页将会跳转到 `http://127.0.0.1:30000/login?service=http://127.0.0.1:8000` CAS 登录页面（注意端口不同），登录成功后将会跳转回来。

## 总结

CAS 本身逻辑需要理解，但是毕竟是成熟的单点登录架构，一般都存在开源的客户端实现，代码量不多，多调试多参考文档配置即可。

## 参考

- [github.com/jbittel/dja…](https://github.com/jbittel/django-mama-cas)
- [django-mama-cas.readthedocs.io](https://django-mama-cas.readthedocs.io/)
- [github.com/mingchen/dj…](https://github.com/mingchen/django-cas-ng)
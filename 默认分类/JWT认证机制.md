# JWT认证机制

### 1. 定义

> `Json web token (JWT)`, 是为了在网络应用环境间传递声明而执行的一种基于`JSON`的开放标准（[(RFC 7519](https://link.jianshu.com/?t=https://tools.ietf.org/html/rfc7519)).该`token`被设计为紧凑且安全的，特别适用于分布式站点的单点登录（`SSO`）场景。`JWT`的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该`token`也可直接被用于认证，也可被加密。

### 2. 传统的Session认证

#### 2.1 流程

session认证的流程一般如下：

1. 用户向服务器发送用户名和密码。
2. 服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。
3. 服务器向用户返回一个 session_id，写入用户的 Cookie。
4. 用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。
5. 服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

#### 2.2 缺点

- session保存在服务器，当注册用户很多，会增加服务器的开销。
- 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，限制了负载均衡的能力。这也意味着限制了应用的扩展能力。
- session是基于cookie来进行用户识别的, cookie如果被截获，用户就会很容易受到跨站请求伪造（**`CSRF`**）的攻击。

### 3.JWT认证

#### 3.1 token

`JWT`是基于`token`的鉴权机制类似于`http`协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于`token`认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。

#### 3.2 流程

1. 用户使用用户名密码来请求服务器
2. 服务器进行验证用户的信息
3. 服务器通过验证发送给用户一个token
4. 客户端存储token，并在每次请求时附送上这个token值
5. 服务端验证token值，并返回数据

这个token必须要在每次请求时传递给服务端，它应该保存在`请求头`里， 另外，服务端要支持`CORS(跨来源资源共享)`策略，一般我们在服务端这么做就可以了`Access-Control-Allow-Origin: *`。

#### 3.3 JWT构成

JWS是JWT的一种实现，除了JWS外，JWE(JSON Web Encryption)也是JWT的一种实现。JWE的生成过程较为复杂，虽保证了安全性，但是降低了访问效率，下面主要介绍JWT的一种实现方式--JWS。

第一部分我们称它为头部（`header`),第二部分我们称其为载荷（`payload`)，第三部分是签证（`signature`).

官方链接[jwt.io/](https://jwt.io/)

##### header

jwt的头部承载两部分信息：

- 声明类型，这里是`jwt`
- 声明加密的算法 通常直接使用 `SHA256`

```
# 头部类似如下信息：{  "alg": "HS256",  "typ": "JWT"}
```

将头部进行`base64`加密（该加密是可以对称解密的),构成了第一部分.得到的加密信息是如下的一串字符串：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

##### payload

载荷就是存放有效信息的地方。这些有效信息包含三个部分:

- 标准中注册的声明
- 公共的声明
- 私有的声明

```
{  "name": "John",  "admin": true，  “email”:"xxx@gmail.com"}
```

该部分也是`base64`加密的，加密后得到下面字符串：

```
ewogICJuYW1lIjogIkpvaG4iLAogICJhZG1pbiI6IHRydWXvvIwKICDigJxlbWFpbOKAnToieHh4QGdtYWlsLmNvbSIKfQ
```

##### signature

`JWT`的第三部分是一个签证信息，这个签证信息由三部分组成：

- `header` (`base64`加密)
- `payload` (`base64`加密)
- `secret` 密钥

这个部分需要`base64`加密后的`header`和`base64`加密后的`payload`使用`.`连接组成的字符串，然后通过`header`中声明的加密方式进行`SHA256`组合加密（不可逆加密），然后就构成了`jwt`的第三部分。将这三部分用`.`连接成一个完整的字符串,构成了最终的`jwt`。

**注意：`secret`是保存在服务器端的，`jwt`的签发生成也是在服务器端的，`secret`就是用来进行`jwt`的签发和`jwt`的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个`secret`, 那就意味着客户端是可以自我签发`jwt`了。如果觉得密钥泄露了，请及时修改。**

### 4. 后台设置

下面介绍的是在`djangorestframework`基础上的`jwt`集成方案。首先确保`djangorestframework`在设置文件中应用中注册。

安装

```
pip install djangorestframework-jwt
```

配置

```
INSTALLED_APPS = [
    ...
    'djangorestframework',
    ...
]
# 认证
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ),
}

JWT_AUTH = {
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1), # JWT_EXPIRATION_DELTA 指明token的有效期
}
```

路由

```
from rest_framework_jwt.views import obtain_jwt_token

urlpatterns = [
    # JWT 完成登录
    url(r'^authorizations/$', obtain_jwt_token),
]
```

默认的`jwt`只会返回给前端一个token值，我们还需要重写`jwt_response_payload_handler`方法，返回前端user及id，然后在配置文件中指明。

```
def jwt_response_payload_handler(token, user=None, request=None):
    """
    自定义jwt认证成功返回数据
    """
    return {
        'token': token,
        'id': user.id,
        'username': user.username
    }
```

配置更新为：

```
# JWT配置
JWT_AUTH = {
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),
    'JWT_RESPONSE_PAYLOAD_HANDLER': 'utils.jwt_response.jwt_response_payload_handler',
}
```

利用djangorestframework-jwt生成的`token`值,前端可以使用js保存在浏览器的`Storage`中。

![img](JWT认证机制.assets/16b7da7783650d44)

### 优点

- 因为json的通用性，所以JWT是可以进行跨语言支持的，像JAVA,JavaScript,NodeJS,PHP等很多语言都可以使用；
- 因为有了payload部分，所以JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息；
- 便于传输，jwt的构成非常简单，字节占用很小，所以它是非常便于传输的；
- 它不需要在服务端保存会话信息, 所以它易于应用的扩展。

### 安全相关

- 不应该在jwt的payload部分存放敏感信息，因为该部分是客户端可解密的部分;
- 保护好secret私钥，该私钥非常重要;
- 如果可以，请使用https协议.
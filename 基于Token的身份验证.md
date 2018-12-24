# 基于Token的身份验证

在实现登录功能的时候,正常的B/S应用都会使用cookie+session的方式来做身份验证,后台直接向cookie中写数据,但是由于移动端的存在,移动端是没有cookie机制的,所以使用token可以实现移动端和客户端的token通信.

# 验证流程

整个基于Token的验证流程如下:
1\. 客户端使用用户名跟密码请求登录
2\. 服务器收到请求,去验证用户名和密码
3\. 验证成功后,服务端会签发一个Token,再把这个Token发送到客户端
4\. 客户端收到的Token以后可以把它存储起来,比如放在Cookie或LocalStorage里
5\. 客户端每次向服务器发送其他请求的时候都要带着服务器签发的Token
6\. 服务器收到请求,去验证客户端请求里面带着的Token,如果验证成功,就像客户端返回请求的数据


```sequence

participant 客户端

participant 服务器

participant 数据库

Note over 客户端: 用户输入通行证的账号、密码

客户端->服务器: 登录请求{username:'',password:''}

服务器->数据库: 验证用户名密码

数据库-->服务器: 验证成功

服务器->服务器: 生成token

服务器-->>客户端: 把这个Token传递给客户端

客户端->客户端: 存储Token

客户端->服务器: 其他请求(携带Token)

服务器->服务器: 验证token

服务器-->>客户端: 返回数据

```


# JWT

构造Token的方法挺多的,可以说只要是客户端和服务器端约定好了格式,是想怎么写就怎么写的,然而还有一些标准写法,例如JWT读作/jot/,表示:JSON Web Tokens.
JWT标准的Token有三个部分:
- header
- payload
- signature

三个部分会用点分割开,并且都会使用Base64编码,所以真正的Token看起来像这样

> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ.SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc

## Header

header部分主要是两部分内容,一个是Token的类型,另一个是使用的算法,比如下面的类型就是JWT,使用的算法是HS256:

```
{
  "typ": "JWT",
  "alg": "HS256"
}
```

上面的内容要用 Base64 的形式编码一下，所以就变成这样：

> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9

## Payload

Payload 里面是 Token 的具体内容，这些内容里面有一些是标准字段，你也可以添加其它需要的内容。下面是标准字段：
- iss：Issuer，发行者
- sub：Subject，主题
- aud：Audience，观众
- exp：Expiration time，过期时间
- nbf：Not before
- iat：Issued at，发行时间
- jti：JWT ID

## Signature

JWT的最后一部分是Signature,这部分相当于前两段的摘要,用来防止其他人来篡改Token中的信息,在处理时可以首先将前两段生成的内容使用Base64生成一下再加盐然后利用MD5等摘要算法在生成一遍

# 服务端生成Token

在服务端生成Token的时候,需要解决两个问题
1\. 使用什么加密算法
2\. Token如何存储

## 加密算法

这里的加密算法并不是MD5,SHA1这样的哈希算法,因为这种算法是无法解密的,只能用来生成摘要,在Django中内置了一个加密前面模块django.core.signing模块,可以用来加密和解密任何数据,使用签名模块的dumps和load函数来实现

示例:

```
from django.core import signing
value = signing.dumps({"foo":"bar"})
src = signing.loads(value)
print(value)
print(src)
```

结果:

> eyJmb28iOiJiYXIifQ:1NMg1b:zGcDE4-TCkaeGzLeW9UQwZesciI
> {‘foo’: ‘bar’}

## Token如何存储

### 用什么存储

在服务器中Token可以存储在内存中,因为本质是字符串,所以并不会占用很大的内存空间,如果是分布式的存储可以将所有的token信息分段存储在不同的服务器中,也可以存储在数据库中,在Django中提供了缓存类,可以用来存储Token,Django的缓存可以结合Redis来使用,可以借助django-redis来实现

#### 安装

> pip install django-redis

#### 配置

为了使用django-redis,需要将django cache setting修改,修改settings.py,默认是没有Cache的配置信息的,在其中添加:

```
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
```

当然,需要你首先安装了redis

#### cache使用

cache使用的时候基本可以使用set和get方法来进行存/取数据

```
>>> cache.set('my_key', 'hello, world!', 30)
>>> cache.get('my_key')
'hello, world!'
```

### 如何存

由于redis是使用k-v模式来进行存储数据的,我们可以使用用户名作为key,而token信息作为value,相较于直接使用token作为key的方式,好处是我们可以使用更少的空间实现一些功能,例如当用户修改了密码或点击注销之后,它的token可以直接失效,直接将该用户名所对应的数据删除就好了,或者用户在一次登录成功后,又一次请求了登录接口,我们可以很简单的更新该用户的token信息,而这样存储所依赖于我们的token可以进行解密,如果你是直接生成了一串无法解密的数据作为token,不能使用用户名作为token了

### code

```
import time
from django.core import signing
import hashlib
from django.core.cache import cache

HEADER = {'typ': 'JWP', 'alg': 'default'}
KEY = 'CHEN_FENG_YAO'
SALT = 'www.lanou3g.com'
TIME_OUT = 30 * 60  # 30min

def encrypt(obj):
    """加密"""
    value = signing.dumps(obj, key=KEY, salt=SALT)
    value = signing.b64_encode(value.encode()).decode()
    return value

def decrypt(src):
    """解密"""
    src = signing.b64_decode(src.encode()).decode()
    raw = signing.loads(src, key=KEY, salt=SALT)
    print(type(raw))
    return raw

def create_token(username):
    """生成token信息"""
    # 1\. 加密头信息
    header = encrypt(HEADER)
    # 2\. 构造Payload
    payload = {"username": username, "iat": time.time()}
    payload = encrypt(payload)
    # 3\. 生成签名
    md5 = hashlib.md5()
    md5.update(("%s.%s" % (header, payload)).encode())
    signature = md5.hexdigest()
    token = "%s.%s.%s" % (header, payload, signature)
    # 存储到缓存中
    cache.set(username, token, TIME_OUT)
    return token

def get_payload(token):
    payload = str(token).split('.')[1]
    payload = decrypt(payload)
    return payload

# 通过token获取用户名
def get_username(token):
    payload = get_payload(token)
    return payload['username']
    pass

def check_token(token):
    username = get_username(token)
    last_token = cache.get(username)
    if last_token:
        return last_token == token
    return False

```

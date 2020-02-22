# Django集成OpenLDAP认证

> 本文详细介绍了django-auth-ldap的使用方法，参数含义，并提供了示例代码

## 版本说明

- Django==2.2
- django-auth-ldap==1.7.0

## 集成过程

Django集成LDAP认证有现成的`django-auth-ldap`模块可以使用，本文也主要以这个模块的使用为主，先安装模块

```
pip install django-auth-ldap
```

然后在setting.py全局配置文件中添加如下内容就可以正常使用了：

```
import ldap
from django_auth_ldap.config import LDAPSearch, GroupOfNamesType

# Baseline configuration.
AUTH_LDAP_SERVER_URI = 'ldap://ldap.ops-coffee.cn'

AUTH_LDAP_BIND_DN = 'uid=authz,ou=Public,dc=ops-coffee,dc=cn'
AUTH_LDAP_BIND_PASSWORD = 'CzfdX629K7'

AUTH_LDAP_USER_SEARCH = LDAPSearch(
    'ou=People,dc=ops-coffee,dc=cn',
    ldap.SCOPE_SUBTREE,
    '(uid=%(user)s)',
)
# Or:
# AUTH_LDAP_USER_DN_TEMPLATE = 'uid=%(user)s,ou=People,dc=ops-coffee,dc=cn'

AUTH_LDAP_USER_ATTR_MAP = {
    'first_name': 'cn',
    'last_name': 'sn',
    'email': 'mail',
}

AUTHENTICATION_BACKENDS = (
    'django_auth_ldap.backend.LDAPBackend',
    'django.contrib.auth.backends.ModelBackend',
)
```

这里详细解释下上边配置的含义：

**AUTH_LDAP_SERVER_URI：** LDAP服务器的地址

**AUTH_LDAP_BIND_DN：** 一个完整的用户DN，用来登录LDAP服务器验证用户输入的账号密码信息是否正确

**AUTH_LDAP_BIND_PASSWORD：** BIND_DN用户的密码，这里我们简单说明下LDAP的认证逻辑以便更好的理解为啥需要这两个配置

Django使用`AUTH_LDAP_BIND_DN`和`AUTH_LDAP_BIND_PASSWORD`作为用户名和密码登陆LDAP服务器，根据`AUTH_LDAP_USER_SEARCH`指定的查询规则来查找用户输入的属性（即username）的值有没有，如果查找的条数为0或者大于1，则返回错误，如果查找的条数等于1，则使用查找到的这个条目的DN和用户输入的密码进行匹配验证，成功则返回成功允许登录，失败则不允许登录

**AUTH_LDAP_USER_SEARCH：** 可通过LDAP登录的用户的范围，如上配置会去`ou=People,dc=ops-coffee,dc=cn`下搜索用户是否存在

其中`(uid=%(user)s)'`指明了作为Django的username所对应的LDAP的属性，这里为LDAP用户的`uid`属性作为Django的username

以上配置是在一个OU下查找用户，当需要在多个OU下搜索用户时用如下配置：

```
from django_auth_ldap.config import LDAPSearch, LDAPSearchUnion

AUTH_LDAP_USER_SEARCH = LDAPSearchUnion(
    LDAPSearch(
        'ou=Public,dc=ops-coffee,dc=cn',
        ldap.SCOPE_SUBTREE,
        '(uid=%(user)s)'
    ),
    LDAPSearch(
        'ou=PeoPle,dc=ops-coffee,dc=cn',
        ldap.SCOPE_SUBTREE,
        '(uid=%(user)s)'
    ),
)
```

**AUTH_LDAP_USER_ATTR_MAP：** LDAP中的用户属性跟Django后台用户属性的对应关系，当用户第一次登录且验证成功后会将LDAP中对应的用户属性写入到Django的User表中

**AUTHENTICATION_BACKENDS：** 配置Django的后端认证列表

当Django调用auth.authenticate方法进行验证时，Django将尝试`AUTHENTICATION_BACKENDS`元组中指定的所有认证后端。如果第一个认证方法失败了，Django将会继续尝试下一个，直到所有认证方式都尝试完成

Django默认的认证后端是`django.contrib.auth.backends.ModelBackend`，如上配置我们添加了ldap的认证到`AUTHENTICATION_BACKENDS`中，那么Django在登录的时候就会先去LDAP服务器验证用户，验证失败后再去查询本地数据库的User表进行验证，如果只希望Django验证LDAP不验证本地数据库的话去掉`AUTHENTICATION_BACKENDS`中的ModelBackend配置即可

其他几个django-auth-ldap的全局配置参数解释如下：

**AUTH_LDAP_ALWAYS_UPDATE_USER：** 是否同步LDAP的修改，默认为True，即当LDAP中用户的属性修改后用户通过LDAP系统认证时自动同步更新到Django的User表中，如果设置为False则不自动更新

**AUTH_LDAP_CACHE_TIMEOUT：** 设置LDAP认证缓存的时间

## 登录验证

上边的配置没有问题后就可以通过LDAP系统账号进行登录操作了，默认登陆逻辑及前端登录代码均无需修改，可以参考github的相关代码，地址：

[github.com/ops-coffee/…](https://github.com/ops-coffee/demo/tree/master/openldap)

## 高级配置

所谓高级配置这里主要是说明下`django-auth-ldap`中组相关的配置，这需要对LDAP的组有一定的概念，为了方便理解，接下来我们以实际的例子来说明

假如我们有三个组overmind、kerrigan、admin，配置如下：

```
# ldapsearch -LLL -x -D "uid=authz,ou=Public,dc=ops-coffee,dc=cn" -w "CzfdX629K7" -b cn=overmind,ou=Group,dc=ops-coffee,dc=cn 
dn: cn=overmind,ou=Group,dc=ops-coffee,dc=cn
cn: overmind
member: uid=sre,ou=People,dc=ops-coffee,dc=cn
objectClass: groupOfNames
objectClass: top
```

```
# ldapsearch -LLL -x -D "uid=authz,ou=Public,dc=ops-coffee,dc=cn" -w "CzfdX629K7" -b cn=kerrigan,ou=Group,dc=ops-coffee,dc=cn 
dn: cn=kerrigan,ou=Group,dc=ops-coffee,dc=cn
cn: kerrigan
objectClass: groupOfNames
objectClass: top
member: uid=u1,ou=Public,dc=ops-coffee,dc=cn
member: uid=u2,ou=People,dc=ops-coffee,dc=cn
```



```
# ldapsearch -LLL -x -D "uid=authz,ou=Public,dc=ops-coffee,dc=cn" -w "CzfdX629K7" -b cn=admin,ou=Group,dc=ops-coffee,dc=cn 
dn: cn=admin,ou=Group,dc=ops-coffee,dc=cn
cn: admin
member: uid=u3,ou=Admin,dc=ops-coffee,dc=cn
objectClass: groupOfNames
objectClass: top
```

我们需要实现Django集成LDAP认证，且不允许隶属于kerrigan分组的用户登录系统，如果用户隶属于admin分组，则需要在登录Django时给设置为管理员，接下来的配置将会解释如何实现该需求

django-auth-ldap中与group有关的配置：

```
AUTH_LDAP_GROUP_SEARCH = LDAPSearch(
    'ou=Group,dc=ops-coffee,dc=cn',
    ldap.SCOPE_SUBTREE,
    '(objectClass=groupOfNames)',
)
AUTH_LDAP_GROUP_TYPE = GroupOfNamesType(name_attr='cn')

# Simple group restrictions
# AUTH_LDAP_REQUIRE_GROUP = 'cn=overmind,ou=Group,dc=ops-coffee,dc=cn'
AUTH_LDAP_DENY_GROUP = 'cn=kerrigan,ou=Group,dc=ops-coffee,dc=cn'

AUTH_LDAP_USER_FLAGS_BY_GROUP = {
    'is_superuser': 'cn=admin,ou=Group,dc=ops-coffee,dc=cn',
}
```

以上配置的详细解释如下：

**AUTH_LDAP_GROUP_SEARCH：** 搜索某个ou下的信息，与`AUTH_LDAP_USER_SEARCH`参数类似，这里的ou一般指group，例如`ou=Group,dc=ops-coffee,dc=cn`的组目录

**AUTH_LDAP_GROUP_TYPE：** 返回的组的类型，组DN的第一个属性值，例如组DN`cn=overmind,ou=Group,dc=ops-coffee,dc=cn`,那么这里为`cn`

**AUTH_LDAP_REQUIRE_GROUP：** 设置允许哪些组成员登录，如果我们只允许overmind组的成员可以登录系统的话这里可以设置

```
AUTH_LDAP_REQUIRE_GROUP = 'cn=overmind,ou=Group,dc=ops-coffee,dc=cn'
```

**AUTH_LDAP_DENY_GROUP：** 设置拒绝哪些组成员登录，如果我们不允许kerrigan组的成员可以登录系统的话这里可以设置

```
AUTH_LDAP_DENY_GROUP = 'cn=kerrigan,ou=Group,dc=ops-coffee,dc=cn'
```

当我们同时设置了用户既属于overmind组又属于kerrigan组，也就是这个用户即设置了允许登录，又设置了拒绝登录，那么以拒绝登录为准，用户无法登录

**AUTH_LDAP_USER_FLAGS_BY_GROUP：** 根据LDAP的group设置Django用户的额外属性，例如我们想要设置LDAP中admin组具有Django中超级管理员的权限，除了在Django中手动设置外，还可以直接在setting中配置`AUTH_LDAP_USER_FLAGS_BY_GROUP`

```
AUTH_LDAP_USER_FLAGS_BY_GROUP = {
    'is_superuser': 'cn=admin,ou=Group,dc=ops-coffee,dc=cn',
}
```

当admin组用户登录的时候就会自动给用户的`is_superuser`属性设置为True

至此我们对django-auth-ldap有了一个全面的了解，在实际项目集成中可以做到游刃有余，如有问题可以参考我github的代码

## 踩坑记录

windowns 10下安装`python-ldap`即`django-auth-ldap`报错：

```
c:\users\ops-coffee\appdata\local\temp\pip-install-sec1o036\python-ldap\modules\constants.h(7): fatal error C1083: Cannot open include file: 'lber.h': No such file or directory
    error: command 'C:\\Program Files (x86)\\Microsoft Visual Studio 14.0\\VC\\BIN\\x86_amd64\\cl.exe' failed with exit status 2
```

这个报错需要手动安装下whl文件，具体方法为：

先在这个网站[www.lfd.uci.edu/~gohlke/pyt…](https://www.lfd.uci.edu/~gohlke/pythonlibs/#python-ldap)下载对应版本的python-ldap的whl文件

然后使用pip命令安装whl，注意文件路径要正确

```
D:\demo\openldap>python -m pip install python_ldap-3.2.0-cp36-cp36m-win_amd64.whl
Processing d:\demo\openldap\python_ldap-3.2.0-cp36-cp36m-win_amd64.whl
Requirement already satisfied: pyasn1>=0.3.7 in c:\python36\lib\site-packages (from python-ldap==3.2.0) (0.4.2)
Requirement already satisfied: pyasn1-modules>=0.1.5 in c:\python36\lib\site-packages (from python-ldap==3.2.0) (0.2.4)
Installing collected packages: python-ldap
Successfully installed python-ldap-3.2.0
```

相关文章推荐阅读：

- [OpenLDAP部署及管理维护](https://mp.weixin.qq.com/s/JyH5mqwWFt0N1nGYZqBCBQ)
- [Django+JWT实现Token认证](https://mp.weixin.qq.com/s/Kgj_5ydnT2W4Gt0vrw7gtw)
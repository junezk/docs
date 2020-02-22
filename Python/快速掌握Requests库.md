# 快速掌握Requests库

## 安装

利用 pip 安装：$ pip install requests

利用 easy_install安装：$ easy_install requests

下载代码后安装：
$ git clone git://github.com/kennethreitz/requests.git
$ cd requests
$ python setup.py install

再懒一点，通过IDE安装吧，如pycharm！

## 简单入门

```python
#HTTP请求类型
#get类型
r = requests.get('https://github.com/timeline.json')
#post类型
r = requests.post("http://m.ctrip.com/post")
#put类型
r = requests.put("http://m.ctrip.com/put")
#delete类型
r = requests.delete("http://m.ctrip.com/delete")
#head类型
r = requests.head("http://m.ctrip.com/head")
#options类型
r = requests.options("http://m.ctrip.com/get")

#获取响应内容
print r.content #以字节的方式去显示，中文显示为字符
print r.text #以文本的方式去显示

#URL传递参数
payload = {'keyword': '日本', 'salecityid': '2'}
r = requests.get("http://m.ctrip.com/webapp/tourvisa/visa_list", params=payload) 
print r.url #示例为http://m.ctrip.com/webapp/tourvisa/visa_list?salecityid=2&keyword=日本

#获取/修改网页编码
r = requests.get('https://github.com/timeline.json')
print r.encoding
r.encoding = 'utf-8'

#json处理
r = requests.get('https://github.com/timeline.json')
print r.json() #需要先import json    

#定制请求头
url = 'http://m.ctrip.com'
headers = {'User-Agent' : 'Mozilla/5.0 (Linux; Android 4.2.1; en-us; Nexus 4 Build/JOP40D) AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.166 Mobile Safari/535.19'}
r = requests.post(url, headers=headers)
print r.request.headers

#复杂post请求
url = 'http://m.ctrip.com'
payload = {'some': 'data'}
r = requests.post(url, data=json.dumps(payload)) #如果传递的payload是string而不是dict，需要先调用dumps方法格式化一下

#post多部分编码文件
url = 'http://m.ctrip.com'
files = {'file': open('report.xls', 'rb')}
r = requests.post(url, files=files)

#响应状态码
r = requests.get('http://m.ctrip.com')
print r.status_code
    
#响应头
r = requests.get('http://m.ctrip.com')
print r.headers
print r.headers['Content-Type']
print r.headers.get('content-type') #访问响应头部分内容的两种方式
    
#Cookies
url = 'http://example.com/some/cookie/setting/url'
r = requests.get(url)
r.cookies['example_cookie_name']    #读取cookies
    
url = 'http://m.ctrip.com/cookies'
cookies = dict(cookies_are='working')
r = requests.get(url, cookies=cookies) #发送cookies

#设置超时时间
r = requests.get('http://m.ctrip.com', timeout=0.001)

#设置访问代理
proxies = {
           "http": "http://10.10.10.10:8888",
           "https": "http://10.10.10.100:4444",
          }
r = requests.get('http://m.ctrip.com', proxies=proxies)
```

## 发送请求与传递参数

一个小例子：



```
r = requests.get('http://www.baidu.com')
print type(r)
print r.status_code  #获取响应状态码
print r.encoding  #获取网页编码
print r.text  # r.text来获取响应的内容。以字符的形式获取
print r.content # 以字节的形式获取响应的内容。requests会自动将内容转码。大多数unicode字体都会无缝转码。
print r.cookies # 获取cookies复制代码
```

不但GET方法简单，其他方法都是统一的接口样式！

```
requests.get(‘https://github.com/timeline.json’) #GET请求
requests.post(“http://httpbin.org/post”) #POST请求
requests.put(“http://httpbin.org/put”) #PUT请求
requests.delete(“http://httpbin.org/delete”) #DELETE请求
requests.head(“http://httpbin.org/get”) #HEAD请求
requests.options(“http://httpbin.org/get”) #OPTIONS请求
```

### GET请求

最基本的GET请求可以直接用get方法

```
r = requests.get("http://httpbin.org/get")
```

如果想要加参数，可以利用 params 参数

```
import requests

payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.get("http://httpbin.org/get", params=payload)
print r.url
```

运行结果

```
http://httpbin.org/get?key2=value2&key1=value1复制代码
```

```
url = "https://www.baidu.com/s"
r = requests.get(url, params={'wd': 'python'})
print r.url
r = requests.get(url, params={'wd': 'php'})
print r.url
print r.text
```

**带参数的请求实例：**

```
import requests
requests.get('http://www.baidu.com/s', params={'wd': 'python'})    #GET参数实例
requests.post('http://www.itwhy.org/wp-comments-post.php', data={'comment': '测试POST'})    #POST参数实例
```

```
url = "http://httpbin.org/post"
# get 方法 使用 params 参数传递数据
# post 方法 使用 data 参数传递数据
r = requests.get("http://httpbin.org/get", params={'wd': 'python'})    # get 参数示例
print r.url
print r.text
# post 方法 如果使用 params 参数传递数据时，传递的数据可以在url中以明文看到
r = requests.post(url, params={'wd': 'python'})
print r.url
print r.text
# post 如果使用 data 参数传递数据时，传递的数据在url中无法看到
r = requests.post(url, data={'comment': 'TEST_POST'})    # post 参数示例
print r.url
print r.text
```

如果想请求JSON文件，可以利用 json() 方法解析。例如自己写一个JSON文件命名为a.json，内容如下

```
["foo", "bar", {
  "foo": "bar"
}]
```

![img](快速掌握Requests库.assets/16ee92c0a0b569a8)

利用如下程序请求并解析

```python
import requests

r = requests.get("a.json") # a.json 代表的一个服务器json文件，这里为了演示，实际是：http://xxx.com/a.json  形式的URL地址
print r.text
print r.json()
运行结果如下，其中一个是直接输出内容，另外一个方法是利用 json() 方法解析。
["foo", "bar", {
 "foo": "bar"
 }]
 [u'foo', u'bar', {u'foo': u'bar'}]
```

如果想获取来自服务器的原始套接字响应，可以取得 r.raw 。 不过需要在初始请求中设置 stream=True 。

```python
r = requests.get('https://github.com/timeline.json', stream=True)
r.raw
<requests.packages.urllib3.response.HTTPResponse object at 0x101194810>
r.raw.read(10)
'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'
```

这样就获取了网页原始套接字内容。如果想添加 headers，可以传 headers 参数

```python
import requests

payload = {'key1': 'value1', 'key2': 'value2'}
headers = {'content-type': 'application/json'}
r = requests.get("http://httpbin.org/get", params=payload, headers=headers)
print r.url
```

通过headers参数可以增加请求头中的headers信息

**定制header：**

```python
import requests
import json
 
data = {'some': 'data'}
headers = {'content-type': 'application/json',
           'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:22.0) Gecko/20100101 Firefox/22.0'}
 
r = requests.post('https://api.github.com/some/endpoint', data=data, headers=headers)
print(r.text)
```

请求头内容可以用r.request.headers来获取。

```
>>> r.request.headers
```

**自定义请求头部：定制headers，使用headers参数来传递**

伪装请求头部是采集时经常用的，我们可以用这个方法来隐藏：

```python
r = requests.get('http://www.zhidaow.com')
print r.request.headers['User-Agent']
#python-requests/1.2.3 CPython/2.7.3 Windows/XP

headers = {'User-Agent': 'alexkh'}
r = requests.get('http://www.zhidaow.com', headers = headers)
print r.request.headers['User-Agent']
#alexkh
```

### POST请求

对于 POST 请求来说，我们一般需要为它增加一些参数。那么最基本的传参方法可以利用 data 这个参数。

```python
import requests

payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.post("http://httpbin.org/post", data=payload)
print r.text
```

运行结果

```
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "key1": "value1", 
    "key2": "value2"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "23", 
    "Content-Type": "application/x-www-form-urlencoded", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.9.1"
  }, 
  "json": null, 
  "url": "http://httpbin.org/post"
}
```

可以看到参数传成功了，然后服务器返回了我们传的数据。有时候我们需要传送的信息不是表单形式的，需要我们传JSON格式的数据过去，所以我们可以用 json.dumps() 方法把表单数据序列化。

POST发送JSON数据：

```python
import json
import requests

url = 'http://httpbin.org/post'
payload = {'some': 'data'}
r = requests.post(url, data=json.dumps(payload))
print r.text
```

运行结果

```python
{
  "args": {}, 
  "data": "{\"some\": \"data\"}", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "16", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.9.1"
  }, 
  "json": {
    "some": "data"
  },  
  "url": "http://httpbin.org/post"
}
```


```python
import requests
import json
 
r = requests.post('https://api.github.com/some/endpoint', data=json.dumps({'some': 'data'}))
print(r.json())
```

通过上述方法，我们可以POST JSON格式的数据。如果想要上传文件，那么直接用 file 参数即可。新建一个 a.txt 的文件，内容写上 Hello World!

```python
import requests

url = 'http://httpbin.org/post'
files = {'file': open('test.txt', 'rb')}
r = requests.post(url, files=files)
print r.text
```

运行结果如下

```python
{
  "args": {}, 
  "data": "", 
  "files": {
    "file": "Hello World!"
  }, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "156", 
    "Content-Type": "multipart/form-data; boundary=7d8eb5ff99a04c11bb3e862ce78d7000", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.9.1"
  }, 
  "json": null, 
  "url": "http://httpbin.org/post"
}
```

这样便成功完成了一个文件的上传。
requests 是支持流式上传的，这允许你发送大的数据流或文件而无需先把它们读入内存。要使用流式上传，仅需为你的请求体提供一个类文件对象即可。这是一个非常实用方便的功能。

```python
with open('massive-body') as f:
    requests.post('http://some.url/streamed', data=f)
import requests
 
url = 'http://127.0.0.1:5000/upload'
files = {'file': open('/home/lyb/sjzl.mpg', 'rb')}
#files = {'file': ('report.jpg', open('/home/lyb/sjzl.mpg', 'rb'))}     #显式的设置文件名
 
r = requests.post(url, files=files)
print(r.text)
```

你可以把字符串当着文件进行上传：

```python
import requests
 
url = 'http://127.0.0.1:5000/upload'
files = {'file': ('test.txt', b'Hello Requests.')}     #必需显式的设置文件名
 
r = requests.post(url, files=files)
print(r.text)
```

发送文件的post类型，这个相当于向网站上传一张图片，文档等操作，这时要使用files参数

```bash
>>> url = 'http://httpbin.org/post'
>>> files = {'file': open('touxiang.png', 'rb')}
>>> r = requests.post(url, files=files)
```

### POST请求模拟登陆及一些返回对象的方法：

```python
#-*- coding:utf-8 -*-
import requests
url1 = 'http://www.exanple.com/login'#登陆地址
url2 = "http://www.example.com/main"#需要登陆才能访问的地址
data={"user":"user","password":"pass"}
headers = { "Accept":"text/html,application/xhtml+xml,application/xml;",
            "Accept-Encoding":"gzip",
            "Accept-Language":"zh-CN,zh;q=0.8",
            "Referer":"http://www.example.com/",
            "User-Agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36"
            }
res1 = requests.post(url1, data=data, headers=headers)
res2 = requests.get(url2, cookies=res1.cookies, headers=headers)

print res2.content#获得二进制响应内容
print res2.raw#获得原始响应内容,需要stream=True
print res2.raw.read(50)
print type(res2.text)#返回解码成unicode的内容
print res2.url
print res2.history#追踪重定向
print res2.cookies
print res2.cookies['example_cookie_name']
print res2.headers
print res2.headers['Content-Type']
print res2.headers.get('content-type')
print res2.json#讲返回内容编码为json
print res2.encoding#返回内容编码
print res2.status_code#返回http状态码
print res2.raise_for_status()#返回错误状态码
```

## Response对象

使用requests方法后，会返回一个response对象，其存储了服务器响应的内容，如上实例中已经提到的 r.text、r.status_code……
获取文本方式的响应体实例：当你访问 r.text 之时，会使用其响应的文本编码进行解码，并且你可以修改其编码让 r.text 使用自定义的编码进行解码。

```python
r.status_code #响应状态码
r.raw #返回原始响应体，也就是 urllib 的 response 对象，使用 r.raw.read() 读取
r.content #字节方式的响应体，会自动为你解码 gzip 和 deflate 压缩
r.text #字符串方式的响应体，会自动根据响应头部的字符编码进行解码
r.headers #以字典对象存储服务器响应头，但是这个字典比较特殊，字典键不区分大小写，若键不存在则返回None
#*特殊方法*#
r.json() #Requests中内置的JSON解码器
r.raise_for_status() #失败请求(非200响应)抛出异常
```

### Cookies

如果一个响应中包含了cookie，那么我们可以利用 cookies 变量来拿到。会话对象让你能够**跨请求**保持某些参数，最方便的是在**同一个Session实例发出的所有请求之间保持cookies**，且这些都是自动处理的

```python
import requests

url = 'http://example.com'
r = requests.get(url)
print r.cookies
print r.cookies['example_cookie_name']
```

以上程序仅是样例，可以用 cookies 变量来得到站点的 cookies。另外可以利用 cookies 变量来向服务器发送 cookies 信息

```python
import requests

url = 'http://httpbin.org/cookies'
cookies = dict(cookies_are='working')
r = requests.get(url, cookies=cookies)
print r.text

运行结果
'{"cookies": {"cookies_are": "working"}}'
```

如果某个响应中包含一些Cookie，你可以快速访问它们：
```python
import requests
 
r = requests.get('http://www.google.com.hk/')
print(r.cookies['NID'])
print(tuple(r.cookies))
```

要想发送你的cookies到服务器，可以使用 cookies 参数：

```python
import requests
 
url = 'http://httpbin.org/cookies'
cookies = {'testCookies_1': 'Hello_Python3', 'testCookies_2': 'Hello_Requests'}
# 在Cookie Version 0中规定空格、方括号、圆括号、等于号、逗号、双引号、斜杠、问号、@，冒号，分号等特殊符号都不能作为Cookie的内容。
r = requests.get(url, cookies=cookies)
print(r.json())
```

如下是快盘签到脚本

```python
import requests
 
headers = {'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
           'Accept-Encoding': 'gzip, deflate, compress',
           'Accept-Language': 'en-us;q=0.5,en;q=0.3',
           'Cache-Control': 'max-age=0',
           'Connection': 'keep-alive',
           'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:22.0) Gecko/20100101 Firefox/22.0'}
 
s = requests.Session()
s.headers.update(headers)
# s.auth = ('superuser', '123')
s.get('https://www.kuaipan.cn/account_login.htm')
 
_URL = 'http://www.kuaipan.cn/index.php'
s.post(_URL, params={'ac':'account', 'op':'login'},
       data={'username':'****@foxmail.com', 'userpwd':'********', 'isajax':'yes'})
r = s.get(_URL, params={'ac':'zone', 'op':'taskdetail'})
print(r.json())
s.get(_URL, params={'ac':'common', 'op':'usersign'})
```

获取响应中的cookies

```bash
>>> r = requests.get('http://www.baidu.com')
>>> r.cookies['BAIDUID']
'D5810267346AEFB0F25CB0D6D0E043E6:FG=1'
```

也可以自已定义请求的COOKIES
```bash
>>> url = 'http://httpbin.org/cookies'
>>> cookies = {'cookies_are':'working'}
>>> r = requests.get(url,cookies = cookies)
>>> 
>>> print r.text
{
  "cookies": {
    "cookies_are": "working"
  }
}
>>>
```

## 超时配置

可以利用 timeout 变量来配置最大请求时间。**timeout 仅对连接过程有效，与响应体的下载无关。**

```python
requests.get('http://github.com', timeout=0.001)
```

**注：timeout 仅对连接过程有效，与响应体的下载无关。**
也就是说，这个时间只限制请求的时间。即使返回的 response 包含很大内容，下载需要一定时间，然而这并没有什么卵用。

## Session

**[自动登录教务处网站](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fwww.cnblogs.com%2Flucky-pin%2Fp%2F5806394.html)**

**会话对象**

访问中使用session
\1. 先初始化一个session对象，s = requests.Session()
\2. 然后使用这个session对象来进行访问，r = s.post(url,data = user)

在以上的请求中，每次请求其实都相当于发起了一个新的请求。也就是相当于我们每个请求都用了不同的浏览器单独打开的效果。也就是它并不是指的一个会话，即使请求的是同一个网址。

比如

```python
import requests
requests.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
r = requests.get("http://httpbin.org/cookies")
print(r.text)

结果是：
{
  "cookies": {}
}
```

很明显，这不在一个会话中，无法获取 cookies，那么在一些站点中，我们需要保持一个持久的会话怎么办呢？就像用一个浏览器逛淘宝一样，在不同的选项卡之间跳转，这样其实就是建立了一个长久会话。

解决方案如下

```python
import requests
s = requests.Session()
s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
r = s.get("http://httpbin.org/cookies")
print(r.text)
```

在这里我们请求了两次，一次是设置 cookies，一次是获得 cookies。运行结果

```python
{
  "cookies": {
    "sessioncookie": "123456789"
  }
}
```

发现可以成功获取到 cookies 了，这就是建立一个会话到作用。那么既然会话是一个全局的变量，那么我们肯定可以用来全局的配置了。

```python
import requests

s = requests.Session()
s.headers.update({'x-test': 'true'})
r = s.get('http://httpbin.org/headers', headers={'x-test2': 'true'})
print r.text
```

通过 s.headers.update 方法设置了 headers 的变量。然后我们又在请求中设置了一个 headers，那么会出现什么结果？

很简单，两个变量都传送过去了。运行结果：

```python
{
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.9.1", 
    "X-Test": "true", 
    "X-Test2": "true"
  }
}
```

如果get方法传的headers 同样也是 x-test 呢？

```
r = s.get('http://httpbin.org/headers', headers={'x-test': 'true'})
```

嗯，它会覆盖掉全局的配置

```
{
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.9.1", 
    "X-Test": "true"
  }
}
```

那如果不想要全局配置中的一个变量了呢？很简单，设置为 None 即可

```
r = s.get('http://httpbin.org/headers', headers={'x-test': None})
```

运行结果

```
{
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.9.1"
  }
}
```

以上就是 session 会话的基本用法

### 使用Session()对象的写法（Prepared Requests）:



```python
#-*- coding:utf-8 -*-
import requests
s = requests.Session()
url1 = 'http://www.exanple.com/login'#登陆地址
url2 = "http://www.example.com/main"#需要登陆才能访问的地址
data={"user":"user","password":"pass"}
headers = { "Accept":"text/html,application/xhtml+xml,application/xml;",
            "Accept-Encoding":"gzip",
            "Accept-Language":"zh-CN,zh;q=0.8",
            "Referer":"http://www.example.com/",
            "User-Agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36"
            }

prepped1 = requests.Request('POST', url1,
    data=data,
    headers=headers
).prepare()
s.send(prepped1)
```


也可以这样写
```python
res = requests.Request('POST', url1,
data=data,
headers=headers
)
prepared = s.prepare_request(res)
# do something with prepped.body
# do something with prepped.headers
s.send(prepared)

prepare2 = requests.Request('POST', url2,
    headers=headers
).prepare()
res2 = s.send(prepare2)

print res2.content
```

另一种写法 :

```python
#-*- coding:utf-8 -*-
import requests
s = requests.Session()
url1 = 'http://www.exanple.com/login'#登陆地址
url2 = "http://www.example.com/main"#需要登陆才能访问的页面地址
data={"user":"user","password":"pass"}
headers = { "Accept":"text/html,application/xhtml+xml,application/xml;",
            "Accept-Encoding":"gzip",
            "Accept-Language":"zh-CN,zh;q=0.8",
            "Referer":"http://www.example.com/",
            "User-Agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36"
            }
res1 = s.post(url1, data=data)
res2 = s.post(url2)
print(resp2.content)
```

## SSL证书验证

现在随处可见 https 开头的网站，Requests可以为HTTPS请求验证SSL证书，就像web浏览器一样。要想检查某个主机的SSL证书，你可以使用 verify 参数。
现在 12306 证书不是无效的嘛，来测试一下

```python
import requests

r = requests.get('https://kyfw.12306.cn/otn/', verify=True)
print r.text
```
结果
```
requests.exceptions.SSLError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:590)
```

果真如此，来试下 github 的

```python
import requests

r = requests.get('https://github.com', verify=True)
print r.text
```

正常请求，内容我就不输出了。如果我们想跳过刚才 12306 的证书验证，把 verify 设置为 False 即可

```python
import requests

r = requests.get('https://kyfw.12306.cn/otn/', verify=False)
print r.text
```

发现就可以正常请求了。在默认情况下 verify 是 True，所以如果需要的话，需要手动设置下这个变量。

## 身份验证

基本身份认证(HTTP Basic Auth):

```python
import requests
from requests.auth import HTTPBasicAuth
 
r = requests.get('https://httpbin.org/hidden-basic-auth/user/passwd', auth=HTTPBasicAuth('user', 'passwd'))
# r = requests.get('https://httpbin.org/hidden-basic-auth/user/passwd', auth=('user', 'passwd'))
# 简写
print(r.json())
```

另一种非常流行的HTTP身份认证形式是摘要式身份认证，Requests对它的支持也是开箱即可用的:

```python
requests.get(URL, auth=HTTPDigestAuth('user', 'pass'))
```

## 代理

如果需要使用代理，你可以通过为任意请求方法提供 proxies 参数来配置单个请求

```python
import requests

proxies = {
  "https": "http://41.118.132.69:4433"
}
r = requests.post("http://httpbin.org/post", proxies=proxies)
print r.text
```

也可以通过环境变量 HTTP_PROXY 和 HTTPS_PROXY 来配置代理

```bash
export HTTP_PROXY="http://10.10.1.10:3128"
export HTTPS_PROXY="http://10.10.1.10:1080"
```

采集时为避免被封IP，经常会使用代理。requests也有相应的proxies属性。

```python
import requests

proxies = {
"http": "http://10.10.1.10:3128",
"https": "http://10.10.1.10:1080",
}

requests.get("http://www.zhidaow.com", proxies=proxies)
```

如果代理需要账户和密码，则需这样：

```
proxies = {
"http": "http://user:pass@10.10.1.10:3128/",
}
```

## 利用Requests来抓取火车票数据

根据观察，数据接口如下:

```
https://kyfw.12306.cn/otn/lcxxcx/query?purpose_codes=ADULT&queryDate=2015-05-23&from_station=NCG&to_station=CZQ
```

返回的是2015-5-23南昌到郴州的火车票信息,格式为json。
返回的数据的如下(只截取了一部分):

```json
{"validateMessagesShowId":"_validatorMessage","status":true,"httpstatus":200,"data":{"datas":[{"train_no":"5u000G140101","station_train_code":"G1401","start_station_telecode":"NXG","start_station_name":"南昌西","end_station_telecode":"IZQ","end_station_name":"广州南","from_station_telecode":"NXG","from_station_name":"南昌西","to_station_telecode":"ICQ","to_station_name":"郴州西","start_time":"07:29","arrive_time":"10:42","day_difference":"0","train_class_name":"","lishi":"03:13","canWebBuy":"Y","lishiValue":"193","yp_info":"O030850182M0507000009097450000","control_train_day":"20991231","start_train_date":"20150523","seat_feature":"O3M393","yp_ex":"O0M090","train_seat_feature":"3","seat_types":"OM9","location_code":"G2","from_station_no":"01","to_station_no":"11","control_day":59,"sale_time":"0930","is_support_card":"1","note":"","gg_num":"--","gr_num":"--","qt_num":"--","rw_num":"--","rz_num":"--","tz_num":"--","wz_num":"--","yb_num":"--","yw_num":"--","yz_num":"--","ze_num":"182","zy_num":"无","swz_num":"无"}}
```

看着很乱，我们稍加整理:

```json
{
    "validateMessagesShowId":"_validatorMessage",
    "status":true,"httpstatus":200,
    "data":{
            "datas":[
                        {
                             "train_no":"5u000G140101",
                             "station_train_code":"G1401",
                             "start_station_telecode":"NXG",
                             "start_station_name":"南昌西",
                             "end_station_telecode":"IZQ",
                             "end_station_name":"广州南",
                             "from_station_telecode":"NXG",
                             "from_station_name":"南昌西",
                             "to_station_telecode":"ICQ",
                             "to_station_name":"郴州西",
                             "start_time":"07:29",
                             "arrive_time":"10:42",
                             "day_difference":"0",
                             ...
                             "swz_num":"无"
                        },
                        {
                              ...
                        }
                    ]
}
```

这样就比较清晰了,代码如下，提取自己需要的信息。

```python
#-*- coding:utf-8 -*-
import requests
import json

class trainTicketsSprider:

    def getTicketsInfo(self,purpose_codes,queryDate,from_station,to_station):
        self.url = 'https://kyfw.12306.cn/otn/lcxxcx/query?purpose_codes=%s&queryDate=%s&from_station=%s&to_station=%s' %(purpose_codes,queryDate,from_station,to_station)
        self.headers = { 
                    "Accept":"text/html,application/xhtml+xml,application/xml;",
                    "Accept-Encoding":"gzip",
                    "Accept-Language":"zh-CN,zh;q=0.8",
                    "User-Agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36"
                  }
        self.TicketSession = requests.Session()
        self.TicketSession.verify = False #关闭https验证   
        self.TicketSession.headers = self.headers
        try:
            self.resp_json = self.TicketSession.get(self.url)
            self.ticketsDatas = json.loads(self.resp_json.text)["data"]["datas"]
            return self.ticketsDatas
        except Exception,e:
            print e

def isZero(num):
    if num == '--' or '无':
        return '0'
    else:
        return num

def main():
    purpose_codes = 'ADULT'
    queryDate = '2015-05-23'
    from_station = 'NCG'
    to_station = 'CZQ'
    TicketSprider = trainTicketsSprider()
    res= TicketSprider.getTicketsInfo(purpose_codes,queryDate,from_station,to_station)
    for i,ticketInfo in enumerate(res):        
                print u"车次:%s" %ticketInfo["station_train_code"]
                print u"起始站:%s" %ticketInfo["start_station_name"]
                print u"目的地:%s" %ticketInfo["to_station_name"]
                print u"开车时间:%s" %ticketInfo["start_time"]
                print u"到达时间:%s" %ticketInfo["arrive_time"]
                print u"二等座还剩:%s张票" %isZero(ticketInfo["ze_num"])
                print u"硬座还剩:%s张票" %isZero(ticketInfo["yz_num"])
                print u"硬卧还剩:%s张票" %isZero(ticketInfo["yw_num"])
                print u"无座还剩:%s张票" %isZero(ticketInfo["wz_num"])
                print u"是否有票:%s" %ticketInfo["canWebBuy"]
                print "**********************************"


if __name__ == '__main__':
    main()
```
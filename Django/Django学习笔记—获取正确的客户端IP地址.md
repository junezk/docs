# Django学习笔记—获取正确的客户端IP地址

在使用了comments库为网站添加评论系统后，我发现一个奇怪的现象：在admin后台看到的评论者IP地址居然全部都是127.0.0.1，这明显不是真正的客户端IP地址。我去查看了一下comments的源代码，获取客户IP地址的一句代码在Python27\Lib\site-packages\django\contrib\comments\views文件里，即：

```python
comment.ip_address=request.META.get("REMOTE_ADDR",None)
```

感觉没有任何问题啊，左思右想得不到要领。这个问题曾困扰过我一段时间，直到有一天我终于把之前看《TheDjangoBook》时跳过的“中间件”一章看了一遍，才搞懂了原委。这事给我的教训就是看书真不能偷懒。下面把这个问题的解决方法记录一下吧。

thedjangobook里对此问题的解释就是：高流量的站点通常需要将Django部署在负载平衡proxy(参见第20章)之后。这种方式将带来一些复杂性，其一就是每个request中的远程IP地址(request.META["REMOTE_IP"])将指向该负载平衡proxy，而不是发起这个request的实际IP。负载平衡proxy处理这个问题的方法在特殊的 X-Forwarded-For 中设置实际发起请求的IP。 因此，需要一个中间件来确保运行在proxy之后的站点也能在request.META['REMOTE_ADDR']中得到正确的IP。

解决办法就是：在settings.py中需要添加如下这个中间件（即middleware）：

```python
'django.middleware.http.SetRemoteAddrFromForwardedFor',
```

需要注意的是：Djangobook上说SetRemoteAddrFromForwardedFor这个类已经被内置在Django当中了，而事实情况是很多Django版本里它又被移除了，故你需要自己将这个类补进去。

所以现在我们需要将这个类写入Django源代码当中，文件路径为Lib\site-packages\django\middleware\http.py，具体代码为：

```python
classSetRemoteAddrFromForwardedFor(object):
defprocess_request(self, request):
   try:
       real_ip=request.META['HTTP_X_FORWARDED_FOR']
   exceptKeyError:
       pass
   else:
       # HTTP_X_FORWARDED_FOR can be a comma-separated list of IPs.
       # Take just the first one.
       real_ip=real_ip.split(",")[0]
       request.META['REMOTE_ADDR']=real_ip
```

当然，我们也可以不用在Django源代码上修改，直接在自己的项目中加上这个类也可以。比如在your_project目录下新建文件http.py，然后将上述代码添加进去，再在settings.py文件中这样引用一下即可：

```
'your_project.http.SetRemoteAddrFromForwardedFor',
```

经过这个操作，你的Django项目就可以正确的获取客户端的IP地址了。 
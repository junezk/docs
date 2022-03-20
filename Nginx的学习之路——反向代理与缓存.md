# Nginx的学习之路——反向代理与缓存

## 反向代理的意义

使用反向代理，可以对用户隐藏服务器的信息，同时代理将用户的请求通过负载均衡算法分发到不同的上游服务器处理，不仅增强了服务的处理性能，也保证了服务的高可用性，在上游的某一台服务器挂掉时，服务仍然可以通过其他的上游服务器对外服务。

## Nginx反向代理配置

这里简单演示一下反向代理，所以只部署了一台上游服务器，这里修改一下现有的`nginx`监听的套接字

**这里nginx监听docker容器内的8080端口，在我本机中的8080端口映射容器的80端口**

![img](https://user-gold-cdn.xitu.io/2019/7/7/16bc817ac08fe7c7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

重新启动`nginx``nginx`

在容器中重新开启一个`nginx`服务器，并修改这个新的服务器配置

![img](https://user-gold-cdn.xitu.io/2019/7/7/16bc83ea183c841d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这里需要注意的是，因为走了一层代理，所以上游服务器是没法拿到客户端的ip等信息的，因为上游服务器的客户端其实是我们的代理服务器，所以我们需要在代理服务器转发的时候将客户端的信息放在头信息中转发出去`proxy_set_header`

此时再在本机访问8080端口（本机8080=>容器中代理服务器80=>容器中上游服务器8080）可以看到可以访问之前的页面了。

而且我们可以看到在响应中加上了代理的信息

![img](https://user-gold-cdn.xitu.io/2019/7/7/16bc8444e3825678?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 缓存

在我们请求的页面中，有一部分内容是一段时间内不变的，比如说`js` `css` 静态`html`文档等，这类文件我们就可以将其缓存在代理服务器上，从而减轻上游服务器的压力。 那么如何设置呢？

首先我们需要指定代理服务器缓存存放的位置：

![img](https://user-gold-cdn.xitu.io/2019/7/7/16bc869e033defd6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`proxy_cache_path`指定了缓存存放的文件的位置以及相关配置

> Sets the path and other parameters of a cache. Cache data are stored in files. The file name in a cache is a result of applying the MD5 function to the cache key. The levels parameter defines hierarchy levels of a cache: from 1 to 3, each level accepts values 1 or 2. For example, in the following configuration

> proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=one:10m; file names in a cache will look like this:

> /data/nginx/cache/c/29/b7f54b2df7773722d382f4809d65029c

更多配置的详细信息可以查看`nginx`的[官方文档](https://link.juejin.im/?target=https%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_proxy_module.html%23proxy_cache)

重启代理服务器，并访问一次首页，使代理服务器缓存下页面，然后我们断掉上游服务器

![img](https://user-gold-cdn.xitu.io/2019/7/7/16bc87ab602f5657?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这里我们看到只有代理服务器进程在服务器，然后我们访问首页发现依然是可以访问的

![img](https://user-gold-cdn.xitu.io/2019/7/7/16bc87baca5b6309?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

同时我们去看我们指定的缓存路径下的文件，发现`nginx`将`html`缓存到我们指定的路径下了

![img](https://user-gold-cdn.xitu.io/2019/7/7/16bc87fd909d4c0a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
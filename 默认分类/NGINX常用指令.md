# [NGINX常用指令](https://segmentfault.com/a/1190000021711621)



最近经常用到nginx，所以想着系统的看一下常用的指令

# HttpCore指令集

## server_name

含义：设置虚拟服务器的名字，第一个名字将成为主服务器名称；服务器名称可以使用*代替名称的第一部分或者最后一部分；也可以使用正则表达式进行捕获，目前不常用

示例

```nginx
server {
    server_name example.com *.example.com www.example.*;
}
# 使用正则
server {
    server_name ~^(www\.)?(.+)$;

    location / {
        root /sites/$2;
    }
}
```

## location

含义：根据URI设置配置，可以使用合法的字符串或者正则表达式

语法：location [=|~|~*|^~] /uri/ { ... }

上下文：server

| 匹配符 | 含义               |
| ------ | ------------------ |
| =      | 精确匹配           |
| ~      | 正则，大小写敏感   |
| ~      | 正则，大小写不敏感 |
| ^~     | 前缀匹配           |

示例

```
location = /ceshi1/ {
    proxy\_pass http://127.0.0.1:7001/xxxx;
    # /ceshi1 -> /xxxx
    # /ceshi1/a -> 404 Not Found
    # /ceshi1a -> 404 Not Found
}
location ^~ /ceshi2/ {
    proxy\_pass http://127.0.0.1:7001/xxxx;
    # /ceshi2 -> /xxxx
    # /ceshi2/a -> /xxxxa
    # /ceshi2a -> 404 Not Found
}
```

**详细对比测试见附录**

## 常用变量

| 变量名          | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| $arg_PARAMETER  | GET请求的参数，PARAMETER为参数名                             |
| query_string    | GET请求的query_string，比如/test?xxxx=ceshi1&yyyy=ceshi2，则Misplaced &args_xxxx为ceshi1 |
| $content_type   | 请求头Content-Type                                           |
| $cookie_COOKIE  | 名称为COOKIE的cookie                                         |
| $host           | 按照以下优先顺序：来自请求行的主机名，来自 Host 请求头字段的主机名，或与请求匹配的服务器名 |
| $https          | 如果连接以 SSL 模式运行，则为 on，否则为空字符串             |
| $is_args        | 如果请求行有参数则为 ?，否则为空字符串                       |
| $http_HEADER    | 获取请求头字段，注意要将中划线给为下划线，示例为http_referer |
| $remote_addr    | 客户端地址                                                   |
| $remote_port    | 客户端端口                                                   |
| $request_method | 请求方法                                                     |
| $request_uri    | 完整的原始请求URI(带参数)                                    |
| $scheme         | 请求模式，http或https                                        |
| $status         | 响应状态                                                     |
| document_uri    | 请求的url path，可能和初始不同，比如使用rewrite              |

# HttpUpstream指令集

该模块用于定义可被proxy_pass等指令应用的服务器组

## upstream

含义：定义一组服务器，服务器可以监听不同端口。

示例

```
upstream backend {
    server backend1.example.com weight=5;
    server 127.0.0.1:8080       max_fails=3 fail_timeout=30s;
    server unix:/tmp/backend3;
 
    server backup1.example.com  backup;
}
server {
    location / {
        proxy_pass http://backend;
    }
}
```

默认情况下，使用加权轮询均衡算法在服务器间分配请求。在上面的示例中，每 7 个请求将按如下方式分发：5 个请求转到 `backend1.example.com`，另外 2 个请求分别转发给第二个和第三个服务器。如果在与服务器通信期间发生错误，请求将被传递到下一个服务器，依此类推，直到尝试完所有正常运行的服务器。如果无法从这些服务器中获得成功响应，则客户端将接收到与最后一个服务器的通信结果。

## server

含义：定义服务器地址和其他参数

# HttpRewrite指令集

## rewrite

含义：根据正则表达式修改URL或者修改字符串，注意，重写表达式只对相对路径有效，如果你想匹配主机名，应该使用if语句

使用位置：server或者location或者if

语法：rewrite regex replacement flag

关于flag

| 值        | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| last      | 停止当前请求，并根据该次修改重新发起请求，然后走一遍这个配置文件 |
| break     | 替换后继续往下执行指令(完成rewrite指令集)                    |
| redirect  | 临时重定向，返回302                                          |
| permanent | 永久重定向，返回301                                          |

关于参数

如果不想让匹配的内容的参数附在重定向的后面，则在最后加一个问号，如下

```
rewrite  ^/users/(.*)$  /show?user=$1?  last;
```

工程示例

```
rewrite . /test/index.html break; # 转发所有的请求给index.html页面，并完成所有的rewrite指令集
```

## break

含义：完成所有的rewrite指令集

使用位置：server,location,if

## if

含义：逻辑判断

使用位置：server,location

语法：if(condition){}

比较符

| 符号       | 含义                                           |
| ---------- | ---------------------------------------------- |
| =          | 相等                                           |
| !=         | 不等                                           |
| ~*         | 正则匹配，大小写不敏感                         |
| ~          | 正则匹配，大小写敏感                           |
| !~*        | 不符合，大小写不敏感                           |
| !~         | 不符合，大小写敏感                             |
| -f and !-f | 判断文件存在或者不存在                         |
| -d and !-d | 检测一个目录是否存在                           |
| -e and !-e | 检测是否存在一个文件，一个目录或者一个符号链接 |
| -x and !-x | 检测一个文件是否可执行                         |

示例

```
set $xsrfToken "";
    if ($http_cookie ~* "XSRF-TOKEN=(.+?)(?=;|$)"){
    set $xsrfToken $1; # 设置变量xsrfToken为cookie中匹配到的值
}
```

## set

含义：设置变量的值

使用位置(上下文)：server,location,if

示例：如上

## return

含义：返回一个状态值给客户端

使用位置(上下文)：server,location,if

示例：

```
if ($invalid_referer) { # 检测到Referers不合法，则禁止访问，返回403
    return 403;
}
```

# HttpProxy指令集

## proxy_pass

含义：设置代理服务器的协议、地址以及映射位置的可选URL，协议可以指定http或https，可以将地址指定为域名或IP地址，以及一个可选端口号；如果域名解析为多个地址，则所有这些地址将以轮询方式使用；可以将地址指定为upstream

使用位置 http -> server -> location

语法：proxy_pass URL

转发时，URI的传递方式如下

- 如果proxy_pass指定了URI，则转发时会将location匹配的规则部分替换为URI部分

  ```
  location /name/ {
      proxy_pass http://127.0.0.1/remote/; # http://a.com/name/test -> http://127.0.0.1/remote/test
  }
  ```

- 如果proxy_pass没有指定URI，则请求URL将会以location匹配为准，示例如下

  ```
  location /xxxx {
      proxy_pass http://127.0.0.1; # 将http://a.com/xxxx/test -> http://127.0.0.1/xxxx/test
  }
  ```

例外情况，无法确定怎么替换URI

- location使用正则匹配，proxy_pass不使用URI
- 使用rewrite，将忽略proxy_pass中的URI

## proxy_set_header

含义：设置请求头内容

语法：proxy_set_header header value

使用位置：http,server,location

```
proxy_set_header X-XSRF-TOKEN $xsrfToken;
```

## proxy_connect_timeout

含义：定义与代理服务器建立连接的超时时间，注意，次超时时间通常不会超过75s

```
proxy_connect_timeout 60;
```

## proxy_send_timeout

含义：设置将请求传输到代理服务器的超时时间。超时时间仅作用于两个连续的写操作之间，而不是整个请求的传输过程。如果代理服务器在该时间内未收到任何内容，则关闭连接。

```
proxy_send_timeout 60;
```

## proxy_read_timeout

含义：定义从代理服务器读取响应的超时时间。该超时时间仅针对两个连续的读操作之间设置，而不是整个响应的传输过程。如果代理服务器在该时间内未传输任何内容，则关闭连接。

```
proxy_read_timeout 60;
```

# HttpHeaders指令集

## add_header

含义：增加响应头内容

使用位置 http -> server -> location

语法： add_header name value;

```
add_header Cache-Control 'no-store'; # 设置所有内容不会被缓存
```

## expires

含义：增加响应头Expires字段，便于确定缓存时间

使用位置：http -> server -> location

语法：expires [time|epoch|max|off]

| 值    | 含义                                                         |
| ----- | ------------------------------------------------------------ |
| time  | 数量                                                         |
| epoch | 1 January, 1970, 00:00:01 GMT                                |
| max   | Cache-Control值为10年，Expires为31 December 2037 23:59:59 GMT |
| off   | 【默认值】不使用时间                                         |

```
expires off; #不使用过期时间
```

注意1：优先级 强缓存优先级 > 对比缓存优先级 ；对于强缓存优先级，pragma > Cache-Control > Expires；对于对比缓存优先级，ETag > Last-Modified

注意2：Cache-Control是http1.1的头字段，Expires是http1.0的头字段，建议两者都写

注意3：Cache-Control默认值为private，其他细节不赘述

细节参考：[https://www.imooc.com/article...](https://www.imooc.com/article/22841)

# HttpReferer指令集

## valid_referers

含义：判断请求头Referers的正确性，结果会赋值给$invalid_referer

使用位置：http -> server -> location

语法：valid_referers [none|blocked|server_names]

| 值           | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| none         | 无Referer，一般直接刷新会是如此                              |
| blocked      | 有，但是被删除，这些值不以[http://或者https](http://xn--https-wm6jl44o/)://开头 |
| server_names | 写一个匹配的路径规则，要带域名，会从scheme后面开始匹配，端口也会忽略，写正则以~开头 |

```
location /views {
    valid_referers *.com/chart/; #判断Referer是否匹配给出的路径，匹配则$invalid_referer为false，否则为true
    if ($invalid_referer) {
        return 403;
    }
}
# 如果Referer为a.com/chart/1则符合规则，如果为a.com/chart则不符合，如果没有referer也不符合
```

# 附录：location/proxy_pass/rewrite对比使用总结

将location和proxy_pass的所有匹配情况测试结果放在下面，对应关系为【访问path -> 转发成的path】

```nginx
location /test1 {
    proxy_pass http://127.0.0.1:7001/;
    # /test1 -> /
    # /test1/a -> //a
    # /test1a -> /a
}
location /test2 {
    proxy_pass http://127.0.0.1:7001/xxxx;
    # /test2 -> /xxxx
    # /test2/a -> /xxxx/a
    # /test2a -> /xxxxa
}
location /test3 {
    proxy_pass http://127.0.0.1:7001/xxxx/;
    # /test3 -> /xxxx//
    # /test3/a -> /xxxx//a
    # /test3a -> /xxxx/a
}
location /test4 {
    proxy_pass http://127.0.0.1:7001;
    # /test4 -> /test4
    # /test4/a -> /test4/a
    # /test4a -> /xxxx4a
}
location /test5/ {
    proxy_pass http://127.0.0.1:7001/;
    # /test5 -> /
    # /test5/a -> /a
    # /test5a -> 404 Not Found
}
location /test6/ {
    proxy_pass http://127.0.0.1:7001/xxxx;
    # /test6 -> /xxxx
    # /test6/a -> /xxxxa
    # /test6a -> 404 Not Found
}
location /test7/ {
    proxy_pass http://127.0.0.1:7001/xxxx/;
    # /test7 -> /xxxx/
    # /test7/a -> /xxxx/a
    # /test7a -> 404 Not Found
}
location /test8/ {
    proxy_pass http://127.0.0.1:7001;
    # /test8 -> /test8/
    # /test8/a -> /test8/a
    # /test8a -> 404 Not Found
}
location /test9/ {
    rewrite . /b break;
    proxy_pass http://127.0.0.1:7001/xxxx;
    # /test9 -> /b
    # /test9/a -> /b
    # /test9a -> 404 Not Found
}
location ~* /test10(.*)/ {
    #proxy_pass http://127.0.0.1:7001/xxxx;
    # nginx无法执行通过，会报错，不能带URI
}

location = /ceshi1/ {
    proxy_pass http://127.0.0.1:7001/xxxx;
    # /ceshi1 -> /xxxx
    # /ceshi1/a -> 404 Not Found
    # /ceshi1a -> 404 Not Found
}
location ^~ /ceshi2/ {
    proxy_pass http://127.0.0.1:7001/xxxx;
    # /ceshi2 -> /xxxx
    # /ceshi2/a -> /xxxxa
    # /ceshi2a -> 404 Not Found
}
```
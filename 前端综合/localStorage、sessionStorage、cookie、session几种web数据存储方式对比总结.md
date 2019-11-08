# localStorage、sessionStorage、cookie、session几种web数据存储方式对比总结

# cookie 和 session

cookie 和 session 都是普遍用来跟踪浏览用户身份的会话方式。

## cookie 和 session 区别

- cookie 数据存放在客户端，session 数据放在服务器端。
- cookie 本身并不安全，考虑到安全应当使用 session。
- session 会在一定时间内保存在服务器上。如果访问量比较大，会比较消耗服务器的性能。考虑到减轻服务器性能方面的开销，应当使用 cookie 。
- 单个 cookie 保存的数据不能超过 4K，很多浏览器都限制一个域名最多保存 50 个 cookie。 将登陆信息等重要信息存放为 session、其他信息如果需要保留，可以放在 cookie 中。

session 主要是服务端使用处理数据，本文主要针对前端技术故不多赘述。

## cookie 的使用

cookie 可通过 `document.cookie` 获取全部 cookie。它是一段字符串，是键值对的形式。操作起来有些麻烦，可引入封装好的库进行使用，比如 `js-cookie`[点我](https://github.com/js-cookie/js-cookie)。API 也很简洁：

```
Cookies.set("name", "value", { expires: 7 }); // 设置一个cookie，7天后失效

Cookies.get("name"); // => 'value'

Cookies.remove("name");
```

# localStorage 和 sessionStorage

在 web 本地存储场景上，cookie 的使用受到种种限制，最关键的就是存储容量太小和数据无法持久化存储。

在 HTML 5 的标准下，出现了 localStorage 和 sessionStorage 供我们使用。

- cookie、localStorage 以及 sessionStorage 的异同点：

| 分类           | 生命周期                                                     | 存储容量                                   | 存储位置                                   |
| -------------- | ------------------------------------------------------------ | ------------------------------------------ | ------------------------------------------ |
| cookie         | 默认保存在内存中，随浏览器关闭失效（如果设置过期时间，在到过期时间后失效） | 4KB                                        | 保存在客户端，每次请求时都会带上           |
| localStorage   | 理论上永久有效的，除非主动清除。                             | 4.98MB（不同浏览器情况不同，safari 2.49M） | 保存在客户端，不与服务端交互。节省网络流量 |
| sessionStorage | 仅在当前网页会话下有效，关闭页面或浏览器后会被清除。         | 4.98MB（部分浏览器没有限制）               | 同上                                       |

- 应用场景：localStorage 适合持久化缓存数据，比如页面的默认偏好配置等；sessionStorage 适合一次性临时数据保存。

WebStorage( localStorage 和 sessionStorage ) 本身就提供了比较好用的方法：

```
localStorage.setItem("name", "value");
localStorage.getItem("name"); // => 'value'
localStorage.removeItem("name");
localStorage.clear(); // 删除所有数据

sessionStorage.setItem("name", "value");
sessionStorage.setItem("name");
sessionStorage.setItem("name");
sessionStorage.clear();
```

注意事项：

- localStorage 写入的时候，如果超出容量会报错，但之前保存的数据不会丢失。
- localStorage 存储容量快要满的时候，getItem 方法性能会急剧下降。
- web storage 在保存复杂数据类型时，较为依赖 `JSON.stringify`，在移动端性能问题比较明显。

# 浏览器兼容性

cookie 兼容性完全没问题；localStorage 和 sessionStorage 也支持 IE8+ 的浏览器，基本都可以正常使用。

# 课外

另外留个小问题，除了手动设置 `localStorage.clear()`，还有哪些方式会丢失 localStorage？
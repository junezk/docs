# Scrapy框架--Settings配置详解及获取自定义变量

> 想要在项目中获取Setting自定义配置的属性时:
>
> 例如在setting文件中添加
>
> USER_AGENT = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36"

```python
#引入get_project_settingsfrom scrapy.utils.project import get_project_settings settings = get_project_settings()print "Your USER_AGENT is:\n%s" % (settings.get('USER_AGENT'))
```

 

[官方参考](https://doc.scrapy.org/en/0.24/topics/settings.html)

## 设置

Scrapy设置允许您自定义所有Scrapy组件的行为，包括核心，扩展，管道和爬虫本身。设置的基础结构提供了键值映射的全局命名空间，代码可以使用它从中提取配置值。可以通过不同的机制来填充设置，这将在下面描述。

## 指定设置

使用`Scrapy`必须要告诉它使用哪些设置。可以使用环境变量`SCRAPY_SETTINGS_MODULE`来进行制定，而且`SCRAPY_SETTINGS_MODULE`需要在Python的路径语法中，例如`myproject.settings`。

## 填充设置

可以使用不同的机制来填充设置，每个机制具有不同的优先级。这里是按优先级降序排列的列表：

1. 命令行选项（最高优先级）
2. 每个`Spider`的`Settings`
3. 项目的`Settings`
4. 每个命令的默认`Settings`
5. 默认的全局`Settings`（最低优先级）

### 1.命令行选项

在使用命令行时提供的参数拥有最高的优先级，将会覆盖所有其他方式设置的相同`Settings`选项。你可以使用`-s`(或者`--set`)来明确指定覆盖一个或多个`Settings`。如下：

```
scrapy crawl myspider -s LOG_FILE=scrapy.log
```

### 2.每个Spider的Settings

在每个`Spider`中，是可以定义这个`Spider`所特有的`Settings`的。只需要在`Spider`类中，定义好`custom_settings`这个类属性即可。如下：

```python
class MySpider(scrapy.Spider):
    name = 'myspider'
 
    custom_settings = {
        'SOME_SETTING': 'some value',
    }
```

### 3.项目的Settings模块

项目的`Settings`是`Scrapy`项目的标准配置文件，我们大部分的设置都会在这个地方配置好。对于一个标准的`Scrapy`项目来说，这意味着我们在`settings.py`这个文件中添加或者修改配置的字段。

这个文件基本的设置都以注释的形式保存在文件中。

### 4.每个命令的默认Settings

每一个`Scrapy`命令都有各自的默认`Settings`，会覆盖掉默认的全局设置。这些`Settings`设定在命令类的`default_settings`属性中。

### 5.默认全局设置

默认的全局变量设定在`scrapy.settings.default_settings`模块中。

## 访问设置

在`Scrapy`中，我们常常需要去获取`Settings`中的某一个值来进行使用，或者是将我们的自定义`Settings`写到`settings.py`文件中，然后在代码中进行获取。`Scrapy`提供了一些非常方便的钩子和方法来获得`Settings`。

### 1.Spider中获取Settings

在`Spider`中获取`Settings`非常的方便，直接访问`self.settings`即可：

```python
class MySpider(scrapy.Spider):
    name = 'myspider'
    start_urls = ['http://example.com']
 
    def parse(self, response):
        print("Existing settings: %s" % self.settings.attributes.keys())
```

注意：settings属性是在基础的Spider类中的__init__()中设定的，这意味着如果你想要在__init__()中使用settings的话，那么你需要重写from_crawler()方法。

### 2.extensions、middlewares、item pipelines中获取Settings

在`extensions`，`middlewares`和`item pipelines`中想要获取`Settings`的话，那么就只能使用`from_crawler()`方法。`from_crawler()`是一个类方法，这个方法一般用来生成当前这个组件的实例，在调用的时候会传入`crawler`参数，通过这个`crawler`参数为桥梁，我们就可以获取`Settings`了：

```python
class MyExtension(object):
    def __init__(self, log_is_enabled=False):
        if log_is_enabled:
            print("log is enabled!")
 
    @classmethod
    def from_crawler(cls, crawler):
        settings = crawler.settings
        return cls(settings.getbool('LOG_ENABLED'))
```

同样要注意的是，由于`from_crawler()`是用来生成组件的实例的，所以最终需要return一个类的实例，然后把我们要获得的`Settings`当作参数传递到这个实例中去，最后再在`__init__()`方法中进行接收。

## 内置基础设置（Built-in settings）

`Scrapy`中定义了许多基础的默认全局`Settings`，以下是一些常见设置的默认值和作用范围。这些设置的默认范围指的是只有启用特定的组件，那么这条设置才会生效。

### 常用设置

- **BOT_NAME**

默认值：`'scrapybot'，`这个`Scrapy`项目的Bot名称。将用于在默认状态下构造`User-Agent`，也用于日志记录。当使用`startproject`命令创建项目时，将会自动使用项目名称来填充这个设置。

- **DEFAULT_REQUEST_HEADERS**

`Scrapy`的`Request`中默认的请求头，将会在`DefaultHeadersMiddleware`中被填充。

```python
{
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
}
```

- **DOWNLOAD_TIMEOUT**

默认值：`180`

`Downloader`下载的最大超时时间。这个设置可以在每个`Spider`中使用`download_timeout`属性进行设置，也可以在每个`Request`中可以使用`Request.meta`中的`download_timeout`字段进行设置。

- **ROBOTSTXT_OBEY**

默认值：`True`

作用域：`scrapy.downloadermiddlewares.robotstxt`

如果设置为`True`，那么`Scrapy`将会自动遵守robots.txt协议。

- **COOKIES_ENABLED**

默认值：`True`

是否开启cookies middleware。如果关闭这个设置的话，请求时将不会发送任何cookies到网站。

### 并发

- **CONCURRENT_ITEMS**

默认值：`100`

在`Item`的处理器中并发处理的最大`Item`数量。

- **CONCURRENT_REQUESTS**

默认值：`16`

`Downloader`同时处理`Request`的最大数量

- **CONCURRENT_REQUESTS_PER_DOMAIN**

****默认值：`8`

任何单个域名同时发起的最大`Request`数量。用来控制单个域名的并发量。

- **CONCURRENT_REQUESTS_PER_IP**

****默认值：`0`

任何单个`IP`同时发起的最大`Request`数量。如果这个值不为0的话，那么`CONCURRENT_REQUESTS_PER_DOMAIN`将会被忽略。也就是说，并发量的限制条件，要么只能使用IP，要么只能使用Domain。

这一个设置同样会影响`DOWNLOAD_DELAY`，使其判断标准变为IP。

- **DOWNLOAD_DELAY**

默认值：`0`

`Downloader`从同一网站上访问页面应该等待的时间。这个设置常常被用来控制爬取的速度，以免对网站造成太大的压力。这个值的单位为秒，并且可以使用小数。

这个值同样被`RANDOMIZE_DOWNLOAD_DELAY`所影响。在默认的情况下，`Scrapy`不会每次都等待一个固定的时间，而是使用一个范围在0.5-1.5之间的值乘以`DOWNLOAD_DELAY`。

当`CONCURRENT_REQUESTS_PER_IP`的值不为0时，延迟将按照每个IP判断，而不再是Domain。

这个设置可以在`Spider`中通过`download_delay`属性进行设置。

- **RANDOMIZE_DOWNLOAD_DELAY**

****默认值：`True`

如果启用，那么`DOWNLOAD_DELAY`等待的时间将会随机乘以范围在0.5-1.5之间的一个小数。这个设置的作用是使爬虫的等待时间更有随机性，更难以被检测。

### 启用组件

- **DOWNLOADER_MIDDLEWARES**

默认值：`{}`

- **DOWNLOADER_MIDDLEWARES_BASE**

默认值：

```
{
'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': 100,    'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware': 300,    'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware': 350,    'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware': 400,    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': 500,    'scrapy.downloadermiddlewares.retry.RetryMiddleware': 550,    'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware': 560,    'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware': 580,    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 590,    'scrapy.downloadermiddlewares.redirect.RedirectMiddleware': 600,    'scrapy.downloadermiddlewares.cookies.CookiesMiddleware': 700,    'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 750,    'scrapy.downloadermiddlewares.stats.DownloaderStats': 850,    'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': 900,
}
```

1. **EXTENSIONS**

默认值：`{}`

包含项目中启用的扩展名及其顺序的字典。

- **EXTENSIONS_BASE**

默认值：

```python
{
    'scrapy.extensions.corestats.CoreStats': 0,
    'scrapy.extensions.telnet.TelnetConsole': 0,
    'scrapy.extensions.memusage.MemoryUsage': 0,
    'scrapy.extensions.memdebug.MemoryDebugger': 0,
    'scrapy.extensions.closespider.CloseSpider': 0,
    'scrapy.extensions.feedexport.FeedExporter': 0,
    'scrapy.extensions.logstats.LogStats': 0,
    'scrapy.extensions.spiderstate.SpiderState': 0,
    'scrapy.extensions.throttle.AutoThrottle': 0,
}

```

- **ITEM_PIPELINES**

默认值：`{}`

- **ITEM_PIPELINES_BASE**

默认值：`{}`

- **SPIDER_MIDDLEWARES**

默认值：`{}`

- **SPIDER_MIDDLEWARES_BASE**

默认值：

```
{
'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware': 50,    'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': 500,    'scrapy.spidermiddlewares.referer.RefererMiddleware': 700,    'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware': 800,    'scrapy.spidermiddlewares.depth.DepthMiddleware': 900,
}
```

### 记录日志

- **LOG_ENABLED**

默认值：`True`

是否开启日志记录。

- **LOG_ENCODING**

默认值：`'utf-8'`

记录日志使用的编码。

- **LOG_FILE**

****默认值：`None`

日志输出的文件路径。

- **LOG_FORMAT**

默认值：`'%(asctime)s [%(name)s] %(levelname)s: %(message)s'`

日志记录的格式。

- **LOG_DATEFORMAT**

默认值：`'%Y-%m-%d %H:%M:%S'`

记录日志的时间格式。

- **LOG_LEVEL**

默认值：`'DEBUG'`

输出日志的最小级别，可用的级别有：CRITICAL, ERROR, WARNING, INFO, DEBUG。

- **LOG_STDOUT**

默认值：`False`

如果设置为`True`，将会把所有的标准输出重定向到日志中。例如print打印的信息和异常信息。

### 内存控制

- **MEMDEBUG_ENABLED**

默认值：`False`

是否开启内存调试。

- **MEMDEBUG_NOTIFY**

****默认值：`[]`

当开启内存调试的时候，可以在这个列表中设置邮箱地址，那么内存调试的报告将会发送到邮箱中。如：

```
MEMDEBUG_NOTIFY = ['user@example.com']
```

- **MEMUSAGE_ENABLED**

****默认值：`True`

作用域：`scrapy.extensions.memusage`

是否开启内存使用的扩展。这个扩展会一直跟踪进程的峰值内存使用（写入到stats中），而且它可以在`Scrapy`进程使用的内存超过了限制的时候，中止掉进程，并且在这一切发生的时候使用email发出通知。

- **MEMUSAGE_LIMIT_MB**

默认值：`0`

作用域：`scrapy.extensions.memusage`

`Scrapy`最大允许使用的内存，单位为MB，超过这个值，`Scrapy`进程将会被终止。如果值为0，那么将被不会进行检查。

- **MEMUSAGE_CHECK_INTERVAL_SECONDS**

默认值：`60.0`

作用域：`scrapy.extensions.memusage`

每隔多长时间检查一次内存的使用量，单位为秒。

- **MEMUSAGE_NOTIFY_MAIL**

默认值：`False`

作用域：`scrapy.extensions.memusage`

接收内存使用报告的邮箱地址。

- **MEMUSAGE_WARNING_MB**

默认值：`0`

作用域：`scrapy.extensions.memusage`

使用内存超过这个值时，将会发出email警告，单位为MB。如果值为0，那么将被不会进行检查。

### 广度优先和深度优先

- **DEPTH_LIMIT**

默认值：`0`

作用域：`scrapy.spidermiddlewares.depth.DepthMiddleware`

对于任一网站，允许爬取的最大深度。如果值为0，则不会有任何限制。

- **DEPTH_PRIORITY**

默认值：`0`

作用域：`scrapy.spidermiddlewares.depth.DepthMiddleware`

用来根据深度调整请求优先级的值。

1. 如果为0，则不会有任何优先级的调整。
2. 如果值为正数，那么会降低优先级，也就是说深度更深的请求将会排在爬取队列的后方。这也意味着深度较浅的请求将会优先爬取。这通常被用于广度优先爬取（BFO）。
3. 如果值为负数，那么会提高优先级，也就是说深度更深的请求会优先爬取。这通常被用于深度优先爬取（DFO）。

- **DEPTH_STATS**

默认值：`True`

作用域：`scrapy.spidermiddlewares.depth.DepthMiddleware`

是否收集最大深度的统计数据。

- **DEPTH_STATS_VERBOSE**

默认值：`False`

作用域：`scrapy.spidermiddlewares.depth.DepthMiddleware`

是否收集最大深度的详细统计数据。如果启用，那么每个深度请求的数量将会被收集到数据中。

### 重试和重定向

- **REDIRECT_ENABLED**

默认值：`True`

是否启用重定向middleware。

- **REDIRECT_MAX_TIMES**

默认值：`20`

请求的最大重定向次数。重定向的次数超过这个值后，相应将被原样返回。

- **REDIRECT_PRIORITY_ADJUST**

默认值：`+2`

作用域：`scrapy.downloadermiddlewares.redirect.RedirectMiddleware`

用来调整重定向的请求跟原始请求的优先级：

1. 值为正数表示重定向的请求优先级更高。
2. 值为负数表示原始请求优先级更高。

- **RETRY_ENABLED**

默认值：`True`

是否启用重试middleware。

- **RETRY_TIMES**

默认值：`2`

除了第一次请求外，最大的重试次数。

这个值也可以在`Request.meta`中的`max_retry_times`字段进行设置。`max_retry_times`的优先级比`RETRY_TIMES`的优先级更高。

- **RETRY_HTTP_CODES**

默认值：`[500, 502, 503, 504, 408]`

需要重试的响应状态码。

- **RETRY_PRIORITY_ADJUST**

默认值：`-1`

作用域：`scrapy.downloadermiddlewares.retry.RetryMiddleware`

用来调整重试的请求跟原始请求的优先级：

1. 值为正数表示重试的请求优先级更高。
2. 值为负数表示原始请求优先级更高。
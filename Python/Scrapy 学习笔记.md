# scrapy 学习笔记

## 安装

```
pip install scrapy
```

## 创建项目

```bash
scrapy startproject tutorial
```

这将创建一个 `tutorial` 目录包含以下内容：

```
tutorial/
    scrapy.cfg            # deploy configuration file
    tutorial/             # project's Python module, you'll import your code from here
        __init__.py
        items.py          # project items definition file
        middlewares.py    # project middlewares file
        pipelines.py      # project pipelines file
        settings.py       # project settings file
        spiders/          # a directory where you'll later put your spiders
            __init__.py
```

## 创建第一个蜘蛛

爬行器是您定义的类，`Scrapy` 使用它从一个网站(或一组网站)中抓取信息。它们必须是 `Spider` 的子类，并定义要做出的初始请求。

我们编写第一个蜘蛛，将其保存在项目的`tutorial/spiders` 目录下，名为 `quotes_spider.py` 的文件中：

```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    def start_requests(self):
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)
    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = f'quotes-{page}.html'
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log(f'Saved file {filename}')
```


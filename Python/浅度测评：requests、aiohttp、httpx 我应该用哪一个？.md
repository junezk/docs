# 浅度测评：requests、aiohttp、httpx 我应该用哪一个？

在 Python 众多的 HTTP 客户端中，最有名的莫过于`requests`、`aiohttp`和`httpx`。在不借助其他第三方库的情况下，`requests`只能发送同步请求；`aiohttp`只能发送异步请求；`httpx`既能发送同步请求，又能发送异步请求。

所谓的同步请求，是指在单进程单线程的代码中，发起一次请求后，在收到返回结果之前，不能发起下一次请求。所谓异步请求，是指在单进程单线程的代码中，发起一次请求后，在等待网站返回结果的时间里，可以继续发送更多请求。

今天我们来一个浅度测评，仅仅以多次发送 POST 请求这个角度来对比这三个库的性能。

测试使用的 HTTP 服务地址为http://122.51.39.219:8000/query，向它发送 POST 请求的格式如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d584397154)

请求发送的 ts 字段日期距离今天大于10天，那么返回`{"success": false}`，如果小于等于10天，那么返回`{"success": true}`。

首先我们通过各个客户端使用相同的参数只发送一次请求，看看效果。

## 发送一次请求

### requests

```python
import requests

resp = requests.post('http://122.51.39.219:8000/query',
                     json={'ts': '2020-01-20 13:14:15'}).json()
print(resp)
复制代码
```

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d586236937)

### httpx

#### 使用 httpx 发送同步请求：

```
import httpx

resp = httpx.post('http://122.51.39.219:8000/query',
                  json={'ts': '2020-01-20 13:14:15'}).json()
print(resp)

复制代码
```

httpx 的同步模式与 requests 代码重合度99%，只需要把`requests`改成`httpx`即可正常运行。如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d588515541)

#### 使用 httpx 发送异步请求：

```
import httpx
import asyncio


async def main():
    async with httpx.AsyncClient() as client:
        resp = await client.post('http://122.51.39.219:8000/query',
                                 json={'ts': '2020-01-20 13:14:15'})
        result = resp.json()
        print(result)


asyncio.run(main())
复制代码
```

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d58884ccc0)

### aiohttp

```
import aiohttp
import asyncio


async def main():
    async with aiohttp.ClientSession() as client:
        resp = await client.post('http://122.51.39.219:8000/query',
                                 json={'ts': '2020-01-20 13:14:15'})
        result = await resp.json()
        print(result)


asyncio.run(main())
复制代码
```

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d58887fcad)

aiohttp 的代码与 httpx 异步模式的代码重合度90%，只不过把`AsyncClient`换成了`ClientSession`，另外，在使用 httpx 时，当你`await client.post`时就已经发送了请求。但是当使用`aiohttp`时，只有在`awiat resp.json()` 时才会真正发送请求。

## 发送100次请求

我们现在随机生成一个距离今天在5-15天的日期，发送到 HTTP接口中。如果日期距离今天超过10天，那么返回的数据的 False，如果小于等于10天，那么返回的数据是 True。

我们发送100次请求，计算总共耗时。

### requests

在前几天的文章中，我们提到，使用`requests.post`每次都会创建新的连接，速度较慢。而如果首先初始化一个 Session，那么 requests 会保持连接，从而大大提高请求速度。所以在这次测评中，我们分别对两种情况进行测试。

#### 不保持连接

```
import random
import time
import datetime
import requests


def make_request(body):
    resp = requests.post('http://122.51.39.219:8000/query', json=body)
    result = resp.json()
    print(result)


def main():
    start = time.time()
    for _ in range(100):
        now = datetime.datetime.now()
        delta = random.randint(5, 15)
        ts = (now - datetime.timedelta(days=delta)).strftime('%Y-%m-%d %H:%M:%S')
        make_request({'ts': ts})
    end = time.time()
    print(f'发送100次请求，耗时：{end - start}')


if __name__ == '__main__':
    main()

复制代码
```

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d589c5cfcd)

**发送100次请求，requests 不保持连接时耗时2.7秒**

#### 保持连接

对代码稍作修改，使用同一个 Session 发送请求：

```
import random
import time
import datetime
import requests


def make_request(session, body):
    resp = session.post('http://122.51.39.219:8000/query', json=body)
    result = resp.json()
    print(result)


def main():
    session = requests.Session()
    start = time.time()
    for _ in range(100):
        now = datetime.datetime.now()
        delta = random.randint(5, 15)
        ts = (now - datetime.timedelta(days=delta)).strftime('%Y-%m-%d %H:%M:%S')
        make_request(session, {'ts': ts})
    end = time.time()
    print(f'发送100次请求，耗时：{end - start}')


if __name__ == '__main__':
    main()
复制代码
```

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d5c1a63fb9)

**发送100次请求，requests 保持连接耗时1.4秒**

### httpx

#### 同步模式

代码如下：

```
import random
import time
import datetime
import httpx


def make_request(client, body):
    resp = client.post('http://122.51.39.219:8000/query', json=body)
    result = resp.json()
    print(result)


def main():
    session = httpx.Client()
    start = time.time()
    for _ in range(100):
        now = datetime.datetime.now()
        delta = random.randint(5, 15)
        ts = (now - datetime.timedelta(days=delta)).strftime('%Y-%m-%d %H:%M:%S')
        make_request(session, {'ts': ts})
    end = time.time()
    print(f'发送100次请求，耗时：{end - start}')


if __name__ == '__main__':
    main()
```

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d5c601565a)

**发送100次请求，httpx 同步模式耗时1.5秒左右。**

#### 异步模式

代码如下：

```
import httpx
import random
import datetime
import asyncio
import time


async def request(client, body):
    resp = await client.post('http://122.51.39.219:8000/query', json=body)
    result = resp.json()
    print(result)


async def main():
    async with httpx.AsyncClient() as client:
        start = time.time()
        task_list = []
        for _ in range(100):
            now = datetime.datetime.now()
            delta = random.randint(5, 15)
            ts = (now - datetime.timedelta(days=delta)).strftime('%Y-%m-%d %H:%M:%S')
            req = request(client, {'ts': ts})
            task = asyncio.create_task(req)
            task_list.append(task)
        await asyncio.gather(*task_list)
        end = time.time()
    print(f'发送100次请求，耗时：{end - start}')

asyncio.run(main())
```

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d5c9fbc80d)

**发送100次请求，httpx 异步模式耗时0.6秒左右。**

### aiohttp

测试代码如下：

```
import aiohttp
import random
import datetime
import asyncio
import time


async def request(client, body):
    resp = await client.post('http://122.51.39.219:8000/query', json=body)
    result = await resp.json()
    print(result)


async def main():
    async with aiohttp.ClientSession() as client:
        start = time.time()
        task_list = []
        for _ in range(100):
            now = datetime.datetime.now()
            delta = random.randint(5, 15)
            ts = (now - datetime.timedelta(days=delta)).strftime('%Y-%m-%d %H:%M:%S')
            req = request(client, {'ts': ts})
            task = asyncio.create_task(req)
            task_list.append(task)
        await asyncio.gather(*task_list)
        end = time.time()
    print(f'发送100次请求，耗时：{end - start}')

asyncio.run(main())
```

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d5ca11f318)

**发送100次请求，使用 aiohttp 耗时0.3秒左右**

## 发送1000次请求

由于 request 保持连接的速度比不保持连接快，所以我们这里只用保持连接的方式来测试。并且不打印返回的结果。

### requests

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d5cd185e16)

**发送1000次请求，requests 耗时16秒左右**

### httpx

#### 同步模式

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d5cf8e2aa5)

**发送1000次请求，httpx 同步模式耗时18秒左右**

#### 异步模式

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d5f97dbb18)

**发送1000次请求，httpx 异步模式耗时5秒左右**

### aiohttp

运行效果如下图所示：

![img](浅度测评：requests、aiohttp、httpx 我应该用哪一个？.assets/16fc83d6017fc862)

**发送1000次请求，aiohttp 耗时4秒左右**

## 总结

如果你只发几条请求。那么使用 requests 或者 httpx 的同步模式，代码最简单。

如果你要发送很多请求，但是有些地方要发送同步请求，有些地方要发送异步请求，那么使用 httpx 最省事。

如果你要发送很多请求，并且越快越好，那么使用 aiohttp 最快。

这篇测评文章只是一个非常浅度的评测，只考虑了请求速度这一个角度。如果你要在生产环境使用，那么你可以做更多实验来看是不是符合你的实际使用情况。
# Flask 蓝图(Blueprint)的引入及应用

我们都知道 flask 是一个轻量级的 web 框架，相对于其他同类型框架更为灵活、轻便、安全且容易上手。开发者可以随意编写自己想要的项目结构，同时还有很多的第三方库供君选择。但是灵活的同时也带来了相应的问题，比如对很多初学者来说，建的项目结构混乱，不易维护，还有经典的循环导入等问题

### 循环导入问题

很多初学者喜欢将启动文件和多个路由写在同一个文件中，例如以下代码:

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d1e31c878)

这样一旦随着视图函数的增多，代码的可维护性会变得越来越差。所以有必要对视图函数进行拆分。 我们将其拆分成两个文件： run.py 和 view.py，其中 run.py 作为程序的启动文件，因为要将路由注册到 flask 核心对象上，所以在 view.py 中需要导入核心对象，同时 run.py 中要导入 view 最终，代码和项目结构如下所示：

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d1778414e)

run.py

```
# coding: utf-8
__author__ = 'Jerry'

from flask import Flask

app = Flask(__name__)

from app.web import view


@app.route("/")
def index():
    return 'Hello World'


if __name__ == '__main__':
    app.run()
```

view.py

```
# coding: utf-8
__author__ = 'Jerry'
from flask import request

from run import app

@app.route("/add")
def add():
    name = request.args.get("name")
    return f"Add {name}"

@app.route("/delete")
def delete():
    name = request.args.get("name")
    return f"Delete {name}"

@app.route("/change",methods=["POST"])
def change():
    num = request.json
    return f"change {num}"
```

但是当我启动程序，在浏览器中输入地址 http://127.0.0.1:5000/add?name=Jerry 之后，报 404 Not Found。

![404](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d17c2aea9)

而 127.0.0.1:5000/ 却能正常响应！

![200](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d2d4b2489)

这是为什么呢? 为什么在 view.py 中注册的视图函数找不到呢? 原因就是因为循环导入，我们在 run.py 中导入了 view, 同时又在 view.py 中导入了 run.py 中的 app！最终导致什么结果呢，我们通过调试来发现吧！

### 详解循环导入

- 启动 run, 第一次实例化一个 Flask 的核心对象 app，执行到导入 view 代码，这是第一个关键点1！

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d3c4adb1a)

- 在 view.py 中，执行到导入 app 的代码，转而去执行 run.py 中的代码

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d2d40ea14)

- 可以看到程序又再一次执行了 run.py 中的代码，同时又一次实例化了一个 Flask 的核心对象，为了区分，我们将它称为 app2，这是第二个关键点 2 ！

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d44c7ddd2)

由于，本次执行 __ name __ != __ main __，所以并不会执行 app.run()。那么程序将继续执行 view.py 中剩余代码

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d51935c51)

可以看到，程序将相关的路由函数注册到了 app2 上面！ 当 view.py 中代码执行完成之后，将继续执行 run.py 中后面的代码。也就是前面提到的关键点 1 之后的相关代码！

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d8e2cdb47)

此时将路由函数 index 注册到了 app 上面，同时 __ name __ == __ main __，所以 app.run() 启动成功！

通过上面的分析，总结如下：

- 整个过程初始化了两次 Flask 核心对象，app 和 app2
- view 中的视图函数注册在 app2 上面
- index 视图函数注册了两次，第一次注册到 app2，最后一次注册 app 上面
- 最终启动的核心对象是 app

现在你应该明白了，view.py 中的视图函数注册的 app 和最终启动的 app 不是同一个，所以最终导致找不到视图函数！为了能有个更直观的感受，我们可以将 app 的内存地址打印出来，这样更加一目了然！

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d60b43367)

### 蓝图

解决方案：Flask 给我们提供一种机制，蓝图(Blueprint)。 蓝图就是一个存储操作路由映射方法的容器，主要用来实现客户端请求和 URL 相互关联的功能。 蓝图类似 Django 中的 app，两者的功能非常相似，帮助我们实现模块化应用的功能。

在 Flask 中可以创建多个蓝图，代表不同的功能模块。比如，上面代码中的 web 文件夹就能作为一个蓝图，另外用户相关的功能模块 user 也能创建一个相应的蓝图

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d6095c376)

### 解决问题

了解蓝图的相关功能之后，我们正式通过蓝图来解决以上问题。

- 首先在web/__ init __.py 文件中创建蓝图：

```
# coding: utf-8
__author__ = 'Jerry'
from flask import Blueprint

api = Blueprint('api', __name__)

from app.web import view

if __name__ == '__main__':
    pass
```

- 蓝图创建完之后，在 view.py 中导入对应的蓝图，并将视图函数注册到蓝图中，改造完之后如下：

```
# coding: utf-8
__author__ = 'Jerry'
from flask import request

from . import api

print(f"注册view中的app: {id(api)}")

@api.route("/add")
def add():
    name = request.args.get("name")
    return f"Add {name}"

@api.route("/delete")
def delete():
    name = request.args.get("name")
    return f"Delete {name}"

@api.route("/change", methods=["POST"])
def change():
    num = request.json
    return f"change {num}"
```

- 我们自始至终都要清楚，Flask 中的相关功能插件以及视图函数最终都要注册到核心对象上面，所以蓝图也是要注册到核心对象上面。同时要了解到，当项目功能较多，用的插件也很多的时候，核心对象的初始化以及蓝图、第三方插件的注册的相关代码会很多，这时候也要对代码进行分离。

所以在 app/__ init __.py 文件中做核心对象的初始化，同时将蓝图注册到核心对象上。代码如下：

 ![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d79d89644) 

同时在启动文件 run.py 记得导入 app 核心对象，修改后代码如下：

![run.py](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d77405424)

### 验证结果

运行之后，我们在浏览器中或者用代码来验证一下 GET 请求

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d837c6708)

POST 请求

![img](Flask 蓝图(Blueprint)的引入及应用.assets/16caa22d80a00e9b)

完美！
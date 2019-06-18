使用 Python 和 Flask 设计 RESTful API
=============================================

近些年来 REST (REpresentational State Transfer) 已经变成了 web services 和 web APIs 的标配。

在本文中我将向你展示如何简单地使用 Python 和 Flask 框架来创建一个 RESTful 的 web service。

什么是 REST？
--------------

六条设计规范定义了一个 REST 系统的特点:

* **客户端-服务器**: 客户端和服务器之间隔离，服务器提供服务，客户端进行消费。
* **无状态**: 从客户端到服务器的每个请求都必须包含理解请求所必需的信息。换句话说， 服务器不会存储客户端上一次请求的信息用来给下一次使用。
* **可缓存**: 服务器必须明示客户端请求能否缓存。
* **分层系统**: 客户端和服务器之间的通信应该以一种标准的方式，就是中间层代替服务器做出响应的时候，客户端不需要做任何变动。
* **统一的接口**: 服务器和客户端的通信方法必须是统一的。
* **按需编码**: 服务器可以提供可执行代码或脚本，为客户端在它们的环境中执行。这个约束是唯一一个是可选的。


什么是一个 RESTful 的 web service？
------------------------------------

REST 架构的最初目的是适应万维网的 HTTP 协议。

RESTful web services 概念的核心就是“资源”。 资源可以用 `URI <https://en.wikipedia.org/wiki/Uniform_resource_identifier>`_ 来表示。客户端使用 HTTP 协议定义的方法来发送请求到这些 URIs，当然可能会导致这些被访问的”资源“状态的改变。

HTTP 标准的方法有如下::

    ==========  =====================  ==================================
    HTTP 方法    行为                    示例
    ==========  =====================  ==================================
    GET         获取资源的信息           http://example.com/api/orders
    GET         获取某个特定资源的信息    http://example.com/api/orders/123
    POST        创建新资源               http://example.com/api/orders
    PUT         更新资源                 http://example.com/api/orders/123
    DELETE      删除资源                 http://example.com/api/orders/123
    ==========  ====================== ==================================

REST 设计不需要特定的数据格式。在请求中数据可以以 `JSON <http://en.wikipedia.org/wiki/JSON>` 形式, 或者有时候作为 url 中查询参数项。


设计一个简单的 web service
----------------------------

坚持 REST 的准则设计一个 web service 或者 API 的任务就变成一个标识资源被展示出来以及它们是怎样受不同的请求方法影响的练习。

比如说，我们要编写一个待办事项应用程序而且我们想要为它设计一个 web service。要做的第一件事情就是决定用什么样的根 URL 来访问该服务。例如，我们可以通过这个来访问::

http://[hostname]/todo/api/v1.0/

在这里我已经决定在 URL 中包含应用的名称以及 API 的版本号。在 URL 中包含应用名称有助于提供一个命名空间以便区分同一系统上的其它服务。在 URL 中包含版本号能够帮助以后的更新，如果新版本中存在新的和潜在不兼容的功能，可以不影响依赖于较旧的功能的应用程序。

下一步骤就是选择将由该服务暴露(展示)的资源。这是一个十分简单地应用，我们只有任务，因此在我们待办事项中唯一的资源就是任务。

我们的任务资源将要使用 HTTP 方法如下::

    ==========  ===============================================  =============================
    HTTP 方法   URL                                              动作
    ==========  ===============================================  ==============================
    GET         http://[hostname]/todo/api/v1.0/tasks            检索任务列表
    GET         http://[hostname]/todo/api/v1.0/tasks/[task_id]  检索某个任务
    POST        http://[hostname]/todo/api/v1.0/tasks            创建新任务
    PUT         http://[hostname]/todo/api/v1.0/tasks/[task_id]  更新任务
    DELETE      http://[hostname]/todo/api/v1.0/tasks/[task_id]  删除任务
    ==========  ================================================ =============================

我们定义的任务有如下一些属性:

* **id**: 任务的唯一标识符。数字类型。
* **title**: 简短的任务描述。字符串类型。
* **description**: 具体的任务描述。文本类型。
* **done**: 任务完成的状态。布尔值。

目前为止关于我们的 web service 的设计基本完成。剩下的事情就是实现它！

Flask 框架的简介
----------------------------

如果你读过 `Flask Mega-Tutorial 系列 <http://www.pythondoc.com/flask-mega-tutorial/index.html>`_，就会知道 Flask 是一个简单却十分强大的 Python web 框架。

在我们深入研究 web services 的细节之前，让我们回顾一下一个普通的 Flask Web 应用程序的结构。

我会首先假设你知道 Python 在你的平台上工作的基本知识。 我将讲解的例子是工作在一个类 Unix 操作系统。简而言之，这意味着它们能工作在 Linux，Mac OS X 和 Windows(如果你使用Cygwin)。
如果你使用 Windows 上原生的 Python 版本的话，命令会有所不同。

让我们开始在一个虚拟环境上安装 Flask。如果你的系统上没有 virtualenv，你可以从 https://pypi.python.org/pypi/virtualenv 上下载::

    $ mkdir todo-api
    $ cd todo-api
    $ virtualenv flask
    New python executable in flask/bin/python
    Installing setuptools............................done.
    Installing pip...................done.
    $ flask/bin/pip install flask

既然已经安装了 Flask，现在开始创建一个简单地网页应用，我们把它放在一个叫 app.py 的文件中::

    #!flask/bin/python
    from flask import Flask
    
    app = Flask(__name__)
    
    @app.route('/')
    def index():
        return "Hello, World!"
    
    if __name__ == '__main__':
        app.run(debug=True)

为了运行这个程序我们必须执行 app.py::

    $ chmod a+x app.py
    $ ./app.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader

现在你可以启动你的网页浏览器，输入 http://localhost:5000 看看这个小应用程序的效果。

简单吧？现在我们将这个应用程序转换成我们的 RESTful service！


使用 Python 和 Flask 实现 RESTful services
-------------------------------------------

使用 Flask 构建 web services 是十分简单地，比我在 `Mega-Tutorial <http://www.pythondoc.com/flask-mega-tutorial/index.html>`_ 中构建的完整的服务端的应用程序要简单地多。

在 Flask 中有许多扩展来帮助我们构建 RESTful services，但是在我看来这个任务十分简单，没有必要使用 Flask 扩展。

我们 web service 的客户端需要添加、删除以及修改任务的服务，因此显然我们需要一种方式来存储任务。最直接的方式就是建立一个小型的数据库，但是数据库并不是本文的主体。学习在 Flask 中使用合适的数据库，我强烈建议阅读 `Mega-Tutorial <http://www.pythondoc.com/flask-mega-tutorial/index.html>`_。

这里我们直接把任务列表存储在内存中，因此这些任务列表只会在 web 服务器运行中工作，在结束的时候就失效。 这种方式只是适用我们自己开发的 web 服务器，不适用于生产环境的 web 服务器， 这种情况一个合适的数据库的搭建是必须的。

我们现在来实现 web service 的第一个入口::

    #!flask/bin/python
    from flask import Flask, jsonify
    
    app = Flask(__name__)
    
    tasks = [
        {
            'id': 1,
            'title': u'Buy groceries',
            'description': u'Milk, Cheese, Pizza, Fruit, Tylenol',
            'done': False
        },
        {
            'id': 2,
            'title': u'Learn Python',
            'description': u'Need to find a good Python tutorial on the web',
            'done': False
        }
    ]
    
    @app.route('/todo/api/v1.0/tasks', methods=['GET'])
    def get_tasks():
        return jsonify({'tasks': tasks})
    
    if __name__ == '__main__':
        app.run(debug=True)

正如你所见，没有多大的变化。我们创建一个任务的内存数据库，这里无非就是一个字典和数组。数组中的每一个元素都具有上述定义的任务的属性。

取代了首页，我们现在拥有一个 get_tasks 的函数，访问的 URI 为 /todo/api/v1.0/tasks，并且只允许 GET 的 HTTP 方法。

这个函数的响应不是文本，我们使用 JSON 数据格式来响应，Flask 的 jsonify 函数从我们的数据结构中生成。

使用网页浏览器来测试我们的 web service 不是一个最好的注意，因为网页浏览器上不能轻易地模拟所有的 HTTP 请求的方法。相反，我们会使用 curl。如果你还没有安装 curl 的话，请立即安装它。

通过执行 app.py，启动 web service。接着打开一个新的控制台窗口，运行以下命令::

    $ curl -i http://localhost:5000/todo/api/v1.0/tasks
    HTTP/1.0 200 OK
    Content-Type: application/json
    Content-Length: 294
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 04:53:53 GMT
    
    {
      "tasks": [
        {
            "description": "Milk, Cheese, Pizza, Fruit, Tylenol",
            "done": false,
            "id": 1,
            "title": "Buy groceries"
        },
        {
            "description": "Need to find a good Python tutorial on the web",
            "done": false,
            "id": 2,
            "title": "Learn Python"
        }
      ]
    }

我们已经成功地调用我们的 RESTful service 的一个函数！

现在我们开始编写 GET 方法请求我们的任务资源的第二个版本。这是一个用来返回单独一个任务的函数::

    from flask import abort
    
    @app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['GET'])
    def get_task(task_id):
        task = filter(lambda t: t['id'] == task_id, tasks)
        if len(task) == 0:
            abort(404)
        return jsonify({'task': task[0]})

第二个函数有些意思。这里我们得到了 URL 中任务的 id，接着 Flask 把它转换成 函数中的 task_id 的参数。

我们用这个参数来搜索我们的任务数组。如果我们的数据库中不存在搜索的 id，我们将会返回一个类似 404 的错误，根据 HTTP 规范的意思是 “资源未找到”。

如果我们找到相应的任务，那么我们只需将它用 jsonify 打包成 JSON 格式并将其发送作为响应，就像我们以前那样处理整个任务集合。

调用 curl 请求的结果如下::

    $ curl -i http://localhost:5000/todo/api/v1.0/tasks/2
    HTTP/1.0 200 OK
    Content-Type: application/json
    Content-Length: 151
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 05:21:50 GMT
    
    {
        "task": {
        "description": "Need to find a good Python tutorial on the web",
        "done": false,
        "id": 2,
        "title": "Learn Python"
        }
    }
    
    $ curl -i http://localhost:5000/todo/api/v1.0/tasks/3
    HTTP/1.0 404 NOT FOUND
    Content-Type: text/html
    Content-Length: 238
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 05:21:52 GMT
    
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
    <title>404 Not Found</title>
    <h1>Not Found</h1>
    <p>The requested URL was not found on the server.</p><p>If you     entered the URL manually please check your spelling and try again.</p>


当我们请求 id #2 的资源时候，我们获取到了，但是当我们请求 #3 的时候返回了 404 错误。有关错误奇怪的是返回的是 HTML 信息而不是 JSON，这是因为 Flask 按照默认方式生成 404 响应。由于这是一个 Web service 客户端希望我们总是以 JSON 格式回应，所以我们需要改善我们的 404 错误处理程序::

    from flask import make_response
    
    @app.errorhandler(404)
    def not_found(error):
        return make_response(jsonify({'error': 'Not found'}), 404)

我们会得到一个友好的错误提示::

    $ curl -i http://localhost:5000/todo/api/v1.0/tasks/3
    HTTP/1.0 404 NOT FOUND
    Content-Type: application/json
    Content-Length: 26
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 05:36:54 GMT
    
    {
        "error": "Not found"
    }

接下来就是 POST 方法，我们用来在我们的任务数据库中插入一个新的任务::

    from flask import request
    
    @app.route('/todo/api/v1.0/tasks', methods=['POST'])
    def create_task():
        if not request.json or not 'title' in request.json:
            abort(400)
        task = {
            'id': tasks[-1]['id'] + 1,
            'title': request.json['title'],
            'description': request.json.get('description', ""),
            'done': False
        }
        tasks.append(task)
        return jsonify({'task': task}), 201

添加一个新的任务也是相当容易地。只有当请求以 JSON 格式形式，request.json 才会有请求的数据。如果没有数据，或者存在数据但是缺少 title 项，我们将会返回 400，这是表示请求无效。

接着我们会创建一个新的任务字典，使用最后一个任务的 id + 1 作为该任务的 id。我们允许 description 字段缺失，并且假设 done 字段设置成 False。

我们把新的任务添加到我们的任务数组中，并且把新添加的任务和状态 201 响应给客户端。

使用如下的 curl 命令来测试这个新的函数::

    $ curl -i -H "Content-Type: application/json" -X POST -d '{"title":"Read a book"}' http://localhost:5000/todo/api/v1.0/tasks
    HTTP/1.0 201 Created
    Content-Type: application/json
    Content-Length: 104
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 05:56:21 GMT
    
    {
        "task": {
        "description": "",
        "done": false,
        "id": 3,
        "title": "Read a book"
        }
    }

注意：如果你在 Windows 上并且运行 Cygwin 版本的 curl，上面的命令不会有任何问题。然而，如果你使用原生的 curl，命令会有些不同::

    curl -i -H "Content-Type: application/json" -X POST -d "{"""title""":"""Read a book"""}" http://localhost:5000/todo/api/v1.0/tasks

当然在完成这个请求后，我们可以得到任务的更新列表::

    $ curl -i http://localhost:5000/todo/api/v1.0/tasks
    HTTP/1.0 200 OK
    Content-Type: application/json
    Content-Length: 423
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 05:57:44 GMT
    
    {
        "tasks": [
        {
            "description": "Milk, Cheese, Pizza, Fruit, Tylenol",
            "done": false,
            "id": 1,
            "title": "Buy groceries"
        },
        {
            "description": "Need to find a good Python tutorial on the web",
            "done": false,
            "id": 2,
            "title": "Learn Python"
        },
        {
            "description": "",
            "done": false,
            "id": 3,
            "title": "Read a book"
        }
        ]
    }

剩下的两个函数如下所示::

    @app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['PUT'])
    def update_task(task_id):
        task = filter(lambda t: t['id'] == task_id, tasks)
        if len(task) == 0:
            abort(404)
        if not request.json:
            abort(400)
        if 'title' in request.json and type(request.json['title']) != unicode:
            abort(400)
        if 'description' in request.json and type(request.json['description']) is not unicode:
            abort(400)
        if 'done' in request.json and type(request.json['done']) is not bool:
            abort(400)
        task[0]['title'] = request.json.get('title', task[0]['title'])
        task[0]['description'] = request.json.get('description', task[0]['description'])
        task[0]['done'] = request.json.get('done', task[0]['done'])
        return jsonify({'task': task[0]})
    
    @app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['DELETE'])
    def delete_task(task_id):
        task = filter(lambda t: t['id'] == task_id, tasks)
        if len(task) == 0:
            abort(404)
        tasks.remove(task[0])
        return jsonify({'result': True})

delete_task 函数没有什么特别的。对于 update_task 函数，我们需要严格地检查输入的参数以防止可能的问题。我们需要确保在我们把它更新到数据库之前，任何客户端提供我们的是预期的格式。

更新任务 #2 的函数调用如下所示::

    $ curl -i -H "Content-Type: application/json" -X PUT -d '{"done":true}' http://localhost:5000/todo/api/v1.0/tasks/2
    HTTP/1.0 200 OK
    Content-Type: application/json
    Content-Length: 170
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 07:10:16 GMT
    
    {
        "task": [
        {
            "description": "Need to find a good Python tutorial on the web",
            "done": true,
            "id": 2,
            "title": "Learn Python"
        }
        ]
    }


优化 web service 接口
------------------------

目前 API 的设计的问题就是迫使客户端在任务标识返回后去构造 URIs。这对于服务器是十分简单的，但是间接地迫使客户端知道这些 URIs 是如何构造的，这将会阻碍我们以后变更这些 URIs。

不直接返回任务的 ids，我们直接返回控制这些任务的完整的 URI，以便客户端可以随时使用这些 URIs。为此，我们可以写一个小的辅助函数生成一个 “公共” 版本任务发送到客户端::

    from flask import url_for
    
    def make_public_task(task):
        new_task = {}
        for field in task:
            if field == 'id':
                new_task['uri'] = url_for('get_task', task_id=task['id'], _external=True)
            else:
                new_task[field] = task[field]
        return new_task

这里所有做的事情就是从我们数据库中取出任务并且创建一个新的任务，这个任务的 id 字段被替换成通过 Flask 的  url_for 生成的 uri 字段。

当我们返回所有的任务列表的时候，在发送到客户端之前通过这个函数进行处理::

    @app.route('/todo/api/v1.0/tasks', methods=['GET'])
    def get_tasks():
        return jsonify({'tasks': map(make_public_task, tasks)})

这里就是客户端获取任务列表的时候得到的数据::

    $ curl -i http://localhost:5000/todo/api/v1.0/tasks
    HTTP/1.0 200 OK
    Content-Type: application/json
    Content-Length: 406
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 18:16:28 GMT
    
    {
      "tasks": [
        {
            "title": "Buy groceries",
            "done": false,
            "description": "Milk, Cheese, Pizza, Fruit, Tylenol",
            "uri": "http://localhost:5000/todo/api/v1.0/tasks/1"
        },
        {
            "title": "Learn Python",
            "done": false,
            "description": "Need to find a good Python tutorial on the web",
            "uri": "http://localhost:5000/todo/api/v1.0/tasks/2"
        }
      ]
    }

我们将会把上述的方式应用到其它所有的函数上以确保客户端一直看到 URIs 而不是 ids。


加强 RESTful web service 的安全性
--------------------------------------

我们已经完成了我们 web service 的大部分功能，但是仍然有一个问题。我们的 web service 对任何人都是公开的，这并不是一个好主意。

我们有一个可以管理我们的待办事项完整的 web service，但在当前状态下的 web service 是开放给所有的客户端。 如果一个陌生人弄清我们的 API 是如何工作的，他或她可以编写一个客户端访问我们的 web service 并且毁坏我们的数据。

大部分初级的教程会忽略这个问题并且到此为止。在我看来这是一个很严重的问题，我必须指出。

确保我们的 web service 安全服务的最简单的方法是要求客户端提供一个用户名和密码。在常规的 web 应用程序会提供一个登录的表单用来认证，并且服务器会创建一个会话为登录的用户以后的操作使用，会话的 id 以 cookie 形式存储在客户端浏览器中。然而 REST 的规则之一就是 “无状态”， 因此我们必须要求客户端在每一次请求中提供认证的信息。

我们一直试着尽可能地坚持 HTTP 标准协议。既然我们需要实现认证我们需要在 HTTP 上下文中去完成，HTTP 协议提供了两种认证机制: `Basic 和 Digest <http://www.ietf.org/rfc/rfc2617.txt>`_。

有一个小的 Flask 扩展能够帮助我们，我们可以先安装 Flask-HTTPAuth::

    $ flask/bin/pip install flask-httpauth

比方说，我们希望我们的 web service 只让访问用户名 miguel 和密码 python 的客户端访问。 我们可以设置一个基本的 HTTP 验证如下::

    from flask.ext.httpauth import HTTPBasicAuth
    auth = HTTPBasicAuth()
    
    @auth.get_password
    def get_password(username):
        if username == 'miguel':
            return 'python'
        return None
    
    @auth.error_handler
    def unauthorized():
        return make_response(jsonify({'error': 'Unauthorized access'}), 401)

get_password 函数是一个回调函数，Flask-HTTPAuth 使用它来获取给定用户的密码。在一个更复杂的系统中，这个函数是需要检查一个用户数据库，但是在我们的例子中只有单一的用户因此没有必要。

error_handler 回调函数是用于给客户端发送未授权错误代码。像我们处理其它的错误代码，这里我们定制一个包含 JSON 数据格式而不是 HTML 的响应。

随着认证系统的建立，所剩下的就是把需要认证的函数添加 @auth.login_required 装饰器。例如::

    @app.route('/todo/api/v1.0/tasks', methods=['GET'])
    @auth.login_required
    def get_tasks():
        return jsonify({'tasks': tasks})

如果现在要尝试使用 curl 调用这个函数我们会得到::

    $ curl -i http://localhost:5000/todo/api/v1.0/tasks
    HTTP/1.0 401 UNAUTHORIZED
    Content-Type: application/json
    Content-Length: 36
    WWW-Authenticate: Basic realm="Authentication Required"
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 06:41:14 GMT
    
    {
        "error": "Unauthorized access"
    }

为了能够调用这个函数我们必须发送我们的认证凭据::

    $ curl -u miguel:python -i http://localhost:5000/todo/api/v1.0/tasks
    HTTP/1.0 200 OK
    Content-Type: application/json
    Content-Length: 316
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Mon, 20 May 2013 06:46:45 GMT


    {
      "tasks": [
        {
          "title": "Buy groceries",
          "done": false,
          "description": "Milk, Cheese, Pizza, Fruit, Tylenol",
          "uri": "http://localhost:5000/todo/api/v1.0/tasks/1"
        },
        {
          "title": "Learn Python",
          "done": false,
          "description": "Need to find a good Python tutorial on the web",
          "uri": "http://localhost:5000/todo/api/v1.0/tasks/2"
        }
      ]
    }

认证扩展给予我们很大的自由选择哪些函数需要保护，哪些函数需要公开。

为了确保登录信息的安全应该使用 HTTP 安全服务器(例如: https://...)，这样客户端和服务器之间的通信都是加密的，以防止传输过程中第三方看到认证的凭据。

让人不舒服的是当请求收到一个 401 的错误，网页浏览都会跳出一个丑陋的登录框，即使请求是在后台发生的。因此如果我们要实现一个完美的 web 服务器的话，我们就需要禁止跳转到浏览器显示身份验证对话框，让我们的客户端应用程序自己处理登录。

一个简单的方式就是不返回 401 错误。403 错误是一个令人青睐的替代，403 错误表示 “禁止” 的错误::

    @auth.error_handler
    def unauthorized():
        return make_response(jsonify({'error': 'Unauthorized access'}), 403)


可能的改进
-------------------------------

我们编写的小型的 web service 还可以在不少的方面进行改进。

对于初学者来说，一个真正的 web service 需要一个真实的数据库进行支撑。我们现在使用的内存数据结构会有很多限制不应该被用于真正的应用。

另外一个可以提高的领域就是处理多用户。如果系统支持多用户的话，不同的客户端可以发送不同的认证凭证获取相应用户的任务列表。在这样一个系统中的话，我们需要第二个资源就是用户。在用户资源上的 POST 的请求代表注册换一个新用户。一个 GET 请求表示客户端获取一个用户的信息。一个 PUT 请求表示更新用户信息，比如可能是更新邮箱地址。一个 DELETE 请求表示删除用户账号。

GET 检索任务列表请求可以在几个方面进行扩展。首先可以携带一个可选的页的参数，以便客户端请求任务的一部分。另外，这种扩展更加有用：允许按照一定的标准筛选。比如，用户只想要看到完成的任务，或者只想看到任务的标题以 A 字母开头。所有的这些都可以作为 URL 的一个参数项。

使用 Flask-RESTful 设计 RESTful API
==========================================

前面我已经用 Flask 实现了一个 RESTful 服务器。今天我们将会使用 Flask-RESTful 来实现同一个 RESTful 服务器，Flask-RESTful 是一个可以简化 APIs 的构建的 Flask 扩展。


RESTful 服务器
-----------------

作为一个提醒， 这里就是待完成事项列表 web service 所提供的方法的定义::

    ==========  ===============================================  =============================
    HTTP 方法   URL                                              动作
    ==========  ===============================================  ==============================
    GET         http://[hostname]/todo/api/v1.0/tasks            检索任务列表
    GET         http://[hostname]/todo/api/v1.0/tasks/[task_id]  检索某个任务
    POST        http://[hostname]/todo/api/v1.0/tasks            创建新任务
    PUT         http://[hostname]/todo/api/v1.0/tasks/[task_id]  更新任务
    DELETE      http://[hostname]/todo/api/v1.0/tasks/[task_id]  删除任务
    ==========  ================================================ =============================

这个服务唯一的资源叫做“任务”，它有如下一些属性:

* **id**: 任务的唯一标识符。数字类型。
* **title**: 简短的任务描述。字符串类型。
* **description**: 具体的任务描述。文本类型。
* **done**: 任务完成的状态。布尔值。


路由
-------

在上一遍文章中，我使用了 Flask 的视图函数来定义所有的路由。

Flask-RESTful 提供了一个 Resource 基础类，它能够定义一个给定 URL 的一个或者多个 HTTP 方法。例如，定义一个可以使用 HTTP 的 GET, PUT 以及 DELETE 方法的 User 资源，你的代码可以如下::

    from flask import Flask
    from flask.ext.restful import Api, Resource
    
    app = Flask(__name__)
    api = Api(app)
    
    class UserAPI(Resource):
        def get(self, id):
            pass
    
        def put(self, id):
            pass
    
        def delete(self, id):
            pass
    
    api.add_resource(UserAPI, '/users/<int:id>', endpoint = 'user')

add_resource 函数使用指定的 endpoint 注册路由到框架上。如果没有指定 endpoint，Flask-RESTful 会根据类名生成一个，但是有时候有些函数比如 url_for 需要 endpoint，因此我会明确给 endpoint 赋值。

我的待办事项 API 定义两个 URLs：/todo/api/v1.0/tasks（获取所有任务列表），以及 /todo/api/v1.0/tasks/<int:id>（获取单个任务）。我们现在需要两个资源::

    class TaskListAPI(Resource):
        def get(self):
            pass
    
        def post(self):
            pass
    
    class TaskAPI(Resource):
        def get(self, id):
            pass
    
        def put(self, id):
            pass
    
        def delete(self, id):
            pass
    
    api.add_resource(TaskListAPI, '/todo/api/v1.0/tasks', endpoint = 'tasks')
    api.add_resource(TaskAPI, '/todo/api/v1.0/tasks/<int:id>', endpoint = 'task')


解析以及验证请求
-----------------

当我在以前的文章中实现此服务器的时候，我自己对请求的数据进行验证。例如，在之前版本中如何处理 PUT 的::

```python
@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods = ['PUT'])
@auth.login_required
def update_task(task_id):
    task = filter(lambda t: t['id'] == task_id, tasks)
    if len(task) == 0:
        abort(404)
    if not request.json:
        abort(400)
    if 'title' in request.json and type(request.json['title']) != unicode:
        abort(400)
    if 'description' in request.json and type(request.json['description']) is not unicode:
        abort(400)
    if 'done' in request.json and type(request.json['done']) is not bool:
        abort(400)
    task[0]['title'] = request.json.get('title', task[0]['title'])
    task[0]['description'] = request.json.get('description', task[0]['description'])
    task[0]['done'] = request.json.get('done', task[0]['done'])
    return jsonify( { 'task': make_public_task(task[0]) } )
```

在这里, 我必须确保请求中给出的数据在使用之前是有效，这样使得函数变得又臭又长。

Flask-RESTful 提供了一个更好的方式来处理数据验证，它叫做 RequestParser 类。这个类工作方式类似命令行解析工具 argparse。

首先，对于每一个资源需要定义参数以及怎样验证它们::

```python
from flask.ext.restful import reqparse

class TaskListAPI(Resource):
    def __init__(self):
        self.reqparse = reqparse.RequestParser()
        self.reqparse.add_argument('title', type = str, required = True,
            help = 'No task title provided', location = 'json')
        self.reqparse.add_argument('description', type = str, default = "", location = 'json')
        super(TaskListAPI, self).__init__()

    # ...

class TaskAPI(Resource):
    def __init__(self):
        self.reqparse = reqparse.RequestParser()
        self.reqparse.add_argument('title', type = str, location = 'json')
        self.reqparse.add_argument('description', type = str, location = 'json')
        self.reqparse.add_argument('done', type = bool, location = 'json')
        super(TaskAPI, self).__init__()

    # ...
```

在 TaskListAPI 资源中，POST 方法是唯一接收参数的。参数“标题”是必须的，因此我定义一个缺少“标题”的错误信息。当客户端缺少这个参数的时候，Flask-RESTful 将会把这个错误信息作为响应发送给客户端。“描述”字段是可选的，当缺少这个字段的时候，默认的空字符串将会被使用。一个有趣的方面就是 RequestParser 类默认情况下在 request.values 中查找参数，因此 location 可选参数必须被设置以表明请求过来的参数是 request.json 格式的。

TaskAPI 资源的参数处理是同样的方式，但是有少许不同。PUT 方法需要解析参数，并且这个方法的所有参数都是可选的。

当请求解析器被初始化，解析和验证一个请求是很容易的。 例如，请注意 TaskAPI.put() 方法变的多么地简单::

```python
def put(self, id):
    task = filter(lambda t: t['id'] == id, tasks)
    if len(task) == 0:
        abort(404)
    task = task[0]
    args = self.reqparse.parse_args()
    for k, v in args.iteritems():
        if v != None:
            task[k] = v
    return jsonify( { 'task': make_public_task(task) } )
```

使用 Flask-RESTful 来处理验证的另一个好处就是没有必要单独地处理类似 HTTP 400 错误，Flask-RESTful 会来处理这些。


生成响应
-----------

原来设计的 REST 服务器使用 Flask 的 jsonify 函数来生成响应。Flask-RESTful 会自动地处理转换成 JSON 数据格式，因此下面的代码需要替换::

```python
return jsonify( { 'task': make_public_task(task) } )
```

现在需要写成这样::

```python
return { 'task': make_public_task(task) }
```

Flask-RESTful 也支持自定义状态码，如果有必要的话::

```python
return { 'task': make_public_task(task) }, 201
```

Flask-RESTful 还有更多的功能。make_public_task 能够把来自原始服务器上的任务从内部形式包装成客户端想要的外部形式。最典型的就是把任务的 id 转成 uri。Flask-RESTful 就提供一个辅助函数能够很优雅地做到这样的转换，不仅仅能够把 id 转成 uri 并且能够转换其他的参数::

```python
from flask.ext.restful import fields, marshal

task_fields = {
    'title': fields.String,
    'description': fields.String,
    'done': fields.Boolean,
    'uri': fields.Url('task')
}

class TaskAPI(Resource):
    # ...

    def put(self, id):
        # ...
        return { 'task': marshal(task, task_fields) }
```

task_fields 结构用于作为 marshal 函数的模板。fields.Uri 是一个用于生成一个 URL 的特定的参数。
它需要的参数是 endpoint。


认证
------

在 REST 服务器中的路由都是由 HTTP 基本身份验证保护着。在最初的那个服务器是通过使用 Flask-HTTPAuth 扩展来实现的。

因为 Resouce 类是继承自 Flask 的 MethodView，它能够通过定义 decorators 变量并且把装饰器赋予给它::

```python
from flask.ext.httpauth import HTTPBasicAuth
# ...
auth = HTTPBasicAuth()
# ...

class TaskAPI(Resource):
    decorators = [auth.login_required]
    # ...

class TaskAPI(Resource):
    decorators = [auth.login_required]
    # ...
```

使用 Flask 设计 RESTful 的认证
======================================

今天我将要展示一个简单，不过很安全的方式用来保护使用 Flask 编写的 API，它是使用密码或者令牌认证的。


示例代码
----------

本文使用的代码能够在 github 上找到: `REST-auth <https://github.com/miguelgrinberg/REST-auth>`_ 。


用户数据库
----------

为了让给出的示例看起来像真实的项目，这里我将使用 Flask-SQLAlchemy 来构建用户数据库模型并且存储到数据库中。

用户的数据库模型是十分简单的。对于每一个用户，username 和 password_hash 将会被存储::

```python
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key = True)
    username = db.Column(db.String(32), index = True)
    password_hash = db.Column(db.String(128))
```

出于安全原因，用户的原始密码将不被存储，密码在注册时被散列后存储到数据库中。使用散列密码的话，如果用户数据库不小心落入恶意攻击者的手里，他们也很难从散列中解析到真实的密码。

密码 **决不能** 很明确地存储在用户数据库中。


密码散列
----------

为了创建密码散列，我将会使用 PassLib 库，一个专门用于密码散列的 Python 包。

PassLib 提供了多种散列算法供选择。custom_app_context 是一个易于使用的基于 sha256_crypt 的散列算法。

User 用户模型需要增加两个新方法来增加密码散列和密码验证功能::

```python
from passlib.apps import custom_app_context as pwd_context

    class User(db.Model):
        # ...

        def hash_password(self, password):
            self.password_hash = pwd_context.encrypt(password)

        def verify_password(self, password):
            return pwd_context.verify(password, self.password_hash)
```

hash_password() 函数接受一个明文的密码作为参数并且存储明文密码的散列。当一个新用户注册到服务器或者当用户修改密码的时候，这个函数将被调用。

verify_password() 函数接受一个明文的密码作为参数并且当密码正确的话返回 True 或者密码错误的话返回 False。这个函数当用户提供和需要验证凭证的时候调用。

你可能会问如果原始密码散列后如何验证原始密码的？

散列算法是单向函数，这就是意味着它们能够用于根据密码生成散列，但是无法根据生成的散列逆向猜测出原密码。然而这些算法是具有确定性的，给定相同的输入它们总会得到相同的输出。PassLib 所有需要做的就是验证密码，通过使用注册时候同一个函数散列密码并且同存储在数据库中的散列值进行比较。

用户注册
----------

在本文例子中，一个客户端可以使用 POST 请求到 /api/users 上注册一个新用户。请求的主体必须是一个包含 username 和 password 的 JSON 格式的对象。

Flask 中的路由的实现如下所示::

```python
@app.route('/api/users', methods = ['POST'])
def new_user():
    username = request.json.get('username')
    password = request.json.get('password')
    if username is None or password is None:
        abort(400) # missing arguments
    if User.query.filter_by(username = username).first() is not None:
        abort(400) # existing user
    user = User(username = username)
    user.hash_password(password)
    db.session.add(user)
    db.session.commit()
    return jsonify({ 'username': user.username }), 201, {'Location': url_for('get_user', id = user.id, _external = True)}
```

这个函数是十分简单地。参数 username 和 password 是从请求中携带的 JSON 数据中获取，接着验证它们。

如果参数通过验证的话，新的 User 实例被创建。username 赋予给 User，接着使用 hash_password 方法散列密码。用户最终被写入数据库中。

响应的主体是一个表示用户的 JSON 对象，201 状态码以及一个指向新创建的用户的 URI 的 HTTP 头信息：Location。

注意：get_user 函数可以在 github 上找到完整的代码。

这里是一个用户注册的请求，发送自 curl::

```shell
$ curl -i -X POST -H "Content-Type: application/json" -d '{"username":"miguel","password":"python"}' http://127.0.0.1:5000/api/users
HTTP/1.0 201 CREATED
Content-Type: application/json
Content-Length: 27
Location: http://127.0.0.1:5000/api/users/1
Server: Werkzeug/0.9.4 Python/2.7.3
Date: Thu, 28 Nov 2013 19:56:39 GMT

{
  "username": "miguel"
}
```

需要注意地是在真实的应用中这里可能会使用安全的的 HTTP (譬如：HTTPS)。如果用户登录的凭证是通过明文在网络传输的话，任何对 API 的保护措施是毫无意义的。


基于密码的认证
--------------

现在我们假设存在一个资源通过一个 API 暴露给那些必须注册的用户。这个资源是通过 URL: /api/resource 能够访问到。

为了保护这个资源，我们将使用 HTTP 基本身份认证，但是不是自己编写完整的代码来实现它，而是让 Flask-HTTPAuth 扩展来为我们做。

使用 Flask-HTTPAuth，通过添加 login_required 装饰器可以要求相应的路由必须进行认证::

```python
from flask.ext.httpauth import HTTPBasicAuth
auth = HTTPBasicAuth()

@app.route('/api/resource')
@auth.login_required
def get_resource():
    return jsonify({ 'data': 'Hello, %s!' % g.user.username })
```

但是，Flask-HTTPAuth 需要给予更多的信息来验证用户的认证，当然 Flask-HTTPAuth 有着许多的选项，它取决于应用程序实现的安全级别。

能够提供最大自由度的选择(可能这也是唯一兼容 PassLib 散列)就是选用 verify_password 回调函数，这个回调函数将会根据提供的 username 和 password 的组合的，返回 True(通过验证) 或者 Flase(未通过验证)。Flask-HTTPAuth 将会在需要验证 username 和 password 对的时候调用这个回调函数。

verify_password 回调函数的实现如下::

```python
@auth.verify_password
def verify_password(username, password):
    user = User.query.filter_by(username = username).first()
    if not user or not user.verify_password(password):
        return False
    g.user = user
    return True
```

这个函数将会根据 username 找到用户，并且使用 verify_password() 方法验证密码。如果认证通过的话，用户对象将会被存储在 Flask 的 g 对象中，这样视图就能使用它。

这里是用 curl 请求只允许注册用户获取的保护资源::

```shell
$ curl -u miguel:python -i -X GET http://127.0.0.1:5000/api/resource
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 30
Server: Werkzeug/0.9.4 Python/2.7.3
Date: Thu, 28 Nov 2013 20:02:25 GMT

{
  "data": "Hello, miguel!"
}
```

如果登录失败的话，会得到下面的内容::

```shell
$ curl -u miguel:ruby -i -X GET http://127.0.0.1:5000/api/resource
HTTP/1.0 401 UNAUTHORIZED
Content-Type: text/html; charset=utf-8
Content-Length: 19
WWW-Authenticate: Basic realm="Authentication Required"
Server: Werkzeug/0.9.4 Python/2.7.3
Date: Thu, 28 Nov 2013 20:03:18 GMT

Unauthorized Access
```

这里我再次重申在实际的应用中，请使用安全的 HTTP。


基于令牌的认证
--------------

每次请求必须发送 username 和 password 是十分不方便，即使是通过安全的 HTTP 传输的话还是存在风险，因为客户端必须要存储不加密的认证凭证，这样才能在每次请求中发送。

一种基于之前解决方案的优化就是使用令牌来验证请求。

我们的想法是客户端应用程序使用认证凭证交换了认证令牌，接下来的请求只发送认证令牌。

令牌是具有有效时间，过了有效时间后，令牌变成无效，需要重新获取新的令牌。令牌的潜在风险在于生成令牌的算法比较弱，但是有效期较短可以减少风险。

有很多的方法可以加强令牌。一个简单的强化方式就是根据存储在数据库中的用户以及密码生成一个随机的特定长度的字符串，可能过期日期也在里面。令牌就变成了明文密码的重排，这样就能很容易地进行字符串对比，还能对过期日期进行检查。

更加完善的实现就是不需要服务器端进行任何存储操作，使用加密的签名作为令牌。这种方式有很多的优点，能够根据用户信息生成相关的签名，并且很难被篡改。

Flask 使用类似的方式处理 cookies 的。这个实现依赖于一个叫做 itsdangerous 的库，我们这里也会采用它。

令牌的生成以及验证将会被添加到 User 模型中，其具体实现如下::

```python
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer

class User(db.Model):
    # ...

    def generate_auth_token(self, expiration = 600):
        s = Serializer(app.config['SECRET_KEY'], expires_in = expiration)
        return s.dumps({ 'id': self.id })

    @staticmethod
    def verify_auth_token(token):
        s = Serializer(app.config['SECRET_KEY'])
        try:
            data = s.loads(token)
        except SignatureExpired:
            return None # valid token, but expired
        except BadSignature:
            return None # invalid token
        user = User.query.get(data['id'])
        return user
```

generate_auth_token() 方法生成一个以用户 id 值为值，'id' 为关键字的字典的加密令牌。令牌中同时加入了一个过期时间，默认为十分钟(600 秒)。

验证令牌是在 verify_auth_token() 静态方法中实现的。静态方法被使用在这里，是因为一旦令牌被解码了用户才可得知。如果令牌被解码了，相应的用户将会被查询出来并且返回。

API 需要一个获取令牌的新函数，这样客户端才能申请到令牌::

```python
@app.route('/api/token')
@auth.login_required
def get_auth_token():
    token = g.user.generate_auth_token()
    return jsonify({ 'token': token.decode('ascii') })
```

注意：这个函数是使用了 auth.login_required 装饰器，也就是说需要提供 username 和 password。

剩下来的就是决策客户端怎样在请求中包含这个令牌。

HTTP 基本认证方式不特别要求 usernames 和 passwords 用于认证，在 HTTP 头中这两个字段可以用于任何类型的认证信息。基于令牌的认证，令牌可以作为 username 字段，password 字段可以忽略。

这就意味着服务器需要同时处理 username 和 password 作为认证，以及令牌作为 username 的认证方式。verify_password 回调函数需要同时支持这两种方式::

```python
@auth.verify_password
def verify_password(username_or_token, password):
    # first try to authenticate by token
    user = User.verify_auth_token(username_or_token)
    if not user:
        # try to authenticate with username/password
        user = User.query.filter_by(username = username_or_token).first()
        if not user or not user.verify_password(password):
            return False
    g.user = user
    return True
```

新版的 verify_password 回调函数会尝试认证两次。首先它会把 username 参数作为令牌进行认证。如果没有验证通过的话，就会像基于密码认证的一样，验证 username 和 password。

如下的 curl 请求能够获取一个认证的令牌::

```shell
$ curl -u miguel:python -i -X GET http://127.0.0.1:5000/api/token
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 139
Server: Werkzeug/0.9.4 Python/2.7.3
Date: Thu, 28 Nov 2013 20:04:15 GMT

{
  "token": "eyJhbGciOiJIUzI1NiIsImV4cCI6MTM4NTY2OTY1NSwiaWF0IjoxMzg1NjY5MDU1fQ.eyJpZCI6MX0.XbOEFJkhjHJ5uRINh2JA1BPzXjSohKYDRT472wGOvjc"
}
```

现在可以使用令牌获取资源::

```shell
$ curl -u eyJhbGciOiJIUzI1NiIsImV4cCI6MTM4NTY2OTY1NSwiaWF0IjoxMzg1NjY5MDU1fQ.eyJpZCI6MX0.XbOEFJkhjHJ5uRINh2JA1BPzXjSohKYDRT472wGOvjc:unused -i -X GET http://127.0.0.1:5000/api/resource
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 30
Server: Werkzeug/0.9.4 Python/2.7.3
Date: Thu, 28 Nov 2013 20:05:08 GMT

{
  "data": "Hello, miguel!"
}
```

需要注意的是这里并没有使用密码。


OAuth 认证
--------------

当我们讨论 RESTful 认证的时候，OAuth 协议经常被提及到。

那么什么是 OAuth？

OAuth 可以有很多的含义。最通常就是一个应用程序允许其它应用程序的用户的接入或者使用服务，但是用户必须使用应用程序提供的登录凭证。我建议阅读者可以浏览 `OAuth <http://en.wikipedia.org/wiki/OAuth>`_ 了解更多知识。


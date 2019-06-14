# 在 Flask 中使用 Celery

后台运行任务的话题是有些复杂，因为围绕这个话题会让人产生困惑。为了简单起见，在以前我所有的例子中，我都是在线程中执行后台任务，但是我一直注意到更具有扩展性以及具备生产解决方案的任务队列像 [Celery](http://www.celeryproject.org/) 应该可以替代线程中执行后台任务。

不断有读者问我关于 Celery 问题，以及怎样在 Flask 应用中使用它，因此今天我将会向你们展示两个例子，我希望能够覆盖大部分的应用需求。

## 什么是 Celery?

Celery 是一个异步任务队列。你可以使用它在你的应用上下文之外执行任务。总的想法就是你的应用程序可能需要执行任何消耗资源的任务都可以交给任务队列，让你的应用程序自由和快速地响应客户端请求。

使用 Celery 运行后台任务并不像在线程中这样做那么简单。但是好处多多，Celery 具有分布式架构，使你的应用易于扩展。一个 Celery 安装有三个核心组件：

1. Celery 客户端: 用于发布后台作业。当与 Flask 一起工作的时候，客户端与 Flask 应用一起运行。
2. Celery workers: 这些是运行后台作业的进程。Celery 支持本地和远程的 workers，因此你就可以在 Flask 服务器上启动一个单独的 worker，随后随着你的应用需求的增加而新增更多的 workers。
3. 消息代理: 客户端通过消息队列和 workers 进行通信，Celery 支持多种方式来实现这些队列。最常用的队列就是 [RabbitMQ](http://www.rabbitmq.com/) 和 [Redis](http://redis.io/)。

## Flask 和 Celery 一起工作

Flask 与 Celery 整合是十分简单，不需要任何插件。一个 Flask 应用需要使用 Celery 的话只需要初始化 Celery 客户端像这样:

```
from flask import Flask
from celery import Celery

app = Flask(__name__)
app.config['CELERY_BROKER_URL'] = 'redis://localhost:6379/0'
app.config['CELERY_RESULT_BACKEND'] = 'redis://localhost:6379/0'

celery = Celery(app.name, broker=app.config['CELERY_BROKER_URL'])
celery.conf.update(app.config)
```

正如你所见，Celery 通过创建一个 Celery 类对象来初始化，传入应用名称以及消息代理的连接 URL，这个 URL 我把它放在 app.config 中的 CELERY_BROKER_URL 的键值。URL 告诉 Celery 代理服务在哪里运行。如果你运行的不是 Redis，或者代理服务运行在一个不同的机器上，相应地你需要改变 URL。

Celery 其它任何配置可以直接用 celery.conf.update() 通过 Flask 的配置直接传递。CELERY_RESULT_BACKEND 选项只有在你必须要 Celery 任务的存储状态和运行结果的时候才是必须的。展示的第一个示例是不需要这个功能的，但是第二个示例是需要的，因此最好从一开始就配置好。

任何你需要作为后台任务的函数需要用 celery.task 装饰器装饰。例如:

```
@celery.task
def my_background_task(arg1, arg2):
    # some long running task here
    return result
```

接着 Flask 应用能够请求这个后台任务的执行，像这样:

```
task = my_background_task.delay(10, 20)
```

delay() 方法是强大的 apply_async() 调用的快捷方式。这样相当于使用 apply_async():

```
task = my_background_task.apply_async(args=[10, 20])
```

当使用 apply_async()，你可以给 Celery 后台任务如何执行的更详细的说明。一个有用的选项就是要求任务在未来的某一时刻执行。例如，这个调用将安排任务运行在大约一分钟后:

```
task = my_background_task.apply_async(args=[10, 20], countdown=60)
```

delay() 和 apply_async() 的返回值是一个表示任务的对象，这个对象可以用于获取任务状态。我将会在本文的后面展示如何获取任务状态等信息，但现在让我们保持简单些，不用担心任务的执行结果。

更多可用的选项请参阅 [Celery 文档](http://docs.celeryproject.org/en/latest/index.html) 。

## 简单例子：异步发送邮件

我要举的第一个示例是应用程序非常普通的需求：能够发送邮件但是不阻塞主应用。

在这个例子中我会用到 [Flask-Mail](https://pythonhosted.org/Flask-Mail/) 扩展，我会假设你们熟悉这个扩展。

我用来说明的示例应用是一个只有一个输入文本框的简单表单。要求用户在此文本框中输入一个电子邮件地址，并在提交，服务器会发送一个测试电子邮件到这个邮件地址。表单中包含两个提交按钮，一个立即发送邮件，一个是一分钟后发送邮件。表单的截图在文章开始。

这里就是支持这个示例的 HTML 模板:

```
<html>
  <head>
    <title>Flask + Celery Examples</title>
  </head>
  <body>
    <h1>Flask + Celery Examples</h1>
    <h2>Example 1: Send Asynchronous Email</h2>
    {% for message in get_flashed_messages() %}
    <p style="color: red;">{{ message }}</p>
    {% endfor %}
    <form method="POST">
      <p>Send test email to: <input type="text" name="email" value="{{ email }}"></p>
      <input type="submit" name="submit" value="Send">
      <input type="submit" name="submit" value="Send in 1 minute">
    </form>
  </body>
</html>
```

这里没有什么特别的东西。只是一个普通的 HTML 表单，再加上 Flask 闪现消息。

Flask-Mail 扩展需要一些配置，尤其是电子邮件服务器发送邮件的时候会用到一些细节。为了简单我使用我的 Gmail 账号作为邮件服务器:

```
# Flask-Mail configuration
app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME')
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD')
app.config['MAIL_DEFAULT_SENDER'] = 'flask@example.com'
```

注意为了避免我的账号丢失的风险，我将其设置在系统的环境变量，这是我从应用中导入的。

有一个单一的路由来支持这个示例:

```
@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'GET':
        return render_template('index.html', email=session.get('email', ''))
    email = request.form['email']
    session['email'] = email

    # send the email
    msg = Message('Hello from Flask',
                  recipients=[request.form['email']])
    msg.body = 'This is a test email sent from a background Celery task.'
    if request.form['submit'] == 'Send':
        # send right away
        send_async_email.delay(msg)
        flash('Sending email to {0}'.format(email))
    else:
        # send in one minute
        send_async_email.apply_async(args=[msg], countdown=60)
        flash('An email will be sent to {0} in one minute'.format(email))

    return redirect(url_for('index'))
```

再次说明，这是一个很标准的 Flask 应用。由于这是一个非常简单的表单，我决定在没有扩展的帮助下处理它，因此我用 request.method 和 request.form 来完成所有的管理。我保存用户在文本框中输入的值在 session 中，这样在页面重新加载后就能记住它。

在这个函数中让人有兴趣的是发送邮件的时候是通过调用一个叫做 send_async_email 的 Celery 任务，该任务调用 delay() 或者 apply_async() 方法。

这个应用的最后一部分就是能够完成作业的异步任务:

```
@celery.task
def send_async_email(msg):
    """Background task to send an email with Flask-Mail."""
    with app.app_context():
        mail.send(msg)
```

这个任务使用 celery.task 装饰使得成为一个后台作业。这个函数唯一值得注意的就是 Flask-Mail 需要在应用的上下文中运行，因此需要在调用 send() 之前创建一个应用上下文。

重点注意在这个示例中从异步调用返回值并不保留，因此应用不能知道调用成功或者失败。当你运行这个示例的时候，需要检查 Celery worker 的输出来排查发送邮件的问题。

## 复杂例子：显示状态更新和结果

上面的示例过于简单，后台作业启动然后应用忘记它。大部分 Celery 针对网页开发的教程就到此为止，但是事实上许多应用程序有必要监控它的后台任务并且获取运行结果。

我现在将要做的就是扩展上面的应用程序成为第二个示例，这个示例展示一个虚构的长时间运行的任务。用户点击按钮启动一个或者更多的长时间运行的任务，在浏览器上的页面使用 ajax 轮询服务器更新所有任务的状态。每一个任务，页面都会显示一个图形的状态栏，进度条，一个状态消息，并且当任务完成的时候，也会显示任务的执行结果。示例的截图在本文的最开始。

### 状态更新的后台任务

让我向你们展示我在第二个示例中使用的后台任务:

```
@celery.task(bind=True)
def long_task(self):
    """Background task that runs a long function with progress reports."""
    verb = ['Starting up', 'Booting', 'Repairing', 'Loading', 'Checking']
    adjective = ['master', 'radiant', 'silent', 'harmonic', 'fast']
    noun = ['solar array', 'particle reshaper', 'cosmic ray', 'orbiter', 'bit']
    message = ''
    total = random.randint(10, 50)
    for i in range(total):
        if not message or random.random() < 0.25:
            message = '{0} {1} {2}...'.format(random.choice(verb),
                                              random.choice(adjective),
                                              random.choice(noun))
        self.update_state(state='PROGRESS',
                          meta={'current': i, 'total': total,
                                'status': message})
        time.sleep(1)
    return {'current': 100, 'total': 100, 'status': 'Task completed!',
            'result': 42}
```

对于这个任务，我在 Celery 装饰器中添加了 bind=True 参数。这个参数告诉 Celery 发送一个 self 参数到我的函数，我能够使用它(self)来记录状态更新。

因为这个任务真没有干什么有用的事情，我决定使用随机的动词，形容词和名词组合的幽默状态信息。你可以在代码上看到我用来生成上述信息的毫无意义的列表。

self.update_state() 调用是 Celery 如何接受这些任务更新。有一些内置的状态，比如 STARTED, SUCCESS 等等，但是 Celery 也支持自定义状态。这里我使用一个叫做 PROGRESS 的自定义状态。连同状态，还有一个附件的元数据，该元数据是 Python 字典形式，包含目前和总的迭代数以及随机生成的状态消息。客户端可以使用这些元素来显示一个漂亮的进度条。每迭代一次休眠一秒，以模拟正在做一些工作。

当循环退出，一个 Python 字典作为函数结果返回。这个字典包含了更新迭代计数器，最后的状态消息和幽默的结果。

上面的 long_task() 函数在一个 Celery worker 进程中运行。下面你能看到启动这个后台作业的 Flask 应用路由:

```
@app.route('/longtask', methods=['POST'])
def longtask():
    task = long_task.apply_async()
    return jsonify({}), 202, {'Location': url_for('taskstatus',
                                                  task_id=task.id)}
```

正如你所见，客户端需要发起一个 POST 请求到 /longtask 来掀开这些任务中的一个的序幕。服务器启动任务，并且存储返回值。对于响应我使用状态码 202，这个状态码通常是在 REST APIs 中使用用来表明一个请求正在进行中。我也添加了 Location 头，值为一个客户端用来获取状态信息的 URL。这个 URL 指向另一个叫做 taskstatus 的 Flask 路由，并且有 task.id 作为动态的要素。

### 从 Flask 应用中访问任务状态

上面提及到 taskstatus 路由负责报告有后台任务提供的状态更新。这里就是这个路由的实现:

```
@app.route('/status/<task_id>')
def taskstatus(task_id):
    task = long_task.AsyncResult(task_id)
    if task.state == 'PENDING':
        // job did not start yet
        response = {
            'state': task.state,
            'current': 0,
            'total': 1,
            'status': 'Pending...'
        }
    elif task.state != 'FAILURE':
        response = {
            'state': task.state,
            'current': task.info.get('current', 0),
            'total': task.info.get('total', 1),
            'status': task.info.get('status', '')
        }
        if 'result' in task.info:
            response['result'] = task.info['result']
    else:
        # something went wrong in the background job
        response = {
            'state': task.state,
            'current': 1,
            'total': 1,
            'status': str(task.info),  # this is the exception raised
        }
    return jsonify(response)
```

这个路由生成一个 JSON 响应，该响应包含任务的状态以及设置在 update_state() 调用中作为 meta 的参数的所有值，客户端可以使用这些构建一个进度条。遗憾地是这个函数需要检查一些条件，因此代码有些长。为了能够访问任务的数据，我重新创建了任务对象，该对象是 AsyncResult 类的实例，使用了 URL 中给的任务 id。

第一个 if 代码块是当任务还没有开始的时候(PENDING 状态)。在这种情况下暂时没有状态信息，因此我人为地制造了些数据。接下来的 elif 代码块返回后台的任务的状态信息。任务提供的信息可以通过访问 task.info 获得。如果数据中包含键 result ，这就意味着这是最终的结果并且任务已经结束，因此我把这些信息也加到响应中。最后的 else 代码块是任务执行失败的情况，这种情况下 task.info 中会包含异常的信息。

不管你是否相信，服务器所有要做的事情已经完成了。剩下的部分就是需要客户端需要实现的，在这里也就是用 JavaScript 脚本的网页来实现。

### 客户端的 Javascript

这一部分就不是本文的重点，如果你有兴趣的话，可以自己研究研究。

对于图形进度条我使用 [nanobar.js](http://nanobar.micronube.com/)，我从 CDN 上引用它。同样还需要引入 jQuery，它能够简化 ajax 的调用。

```
<script src="//cdnjs.cloudflare.com/ajax/libs/nanobar/0.2.1/nanobar.min.js"></script>
<script src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
```

启动连接后台作业的按钮的 Javascript 处理程序如下:

```
function start_long_task() {
    // add task status elements
    div = $('<div class="progress"><div></div><div>0%</div><div>...</div><div>&nbsp;</div></div><hr>');
    $('#progress').append(div);

    // create a progress bar
    var nanobar = new Nanobar({
        bg: '#44f',
        target: div[0].childNodes[0]
    });

    // send ajax POST request to start background job
    $.ajax({
        type: 'POST',
        url: '/longtask',
        success: function(data, status, request) {
            status_url = request.getResponseHeader('Location');
            update_progress(status_url, nanobar, div[0]);
        },
        error: function() {
            alert('Unexpected error');
        }
    });
}
```

div 的代码:

```
<div class="progress">
    <div></div>         <-- Progress bar
    <div>0%</div>       <-- Percentage
    <div>...</div>      <-- Status message
    <div>&nbsp;</div>   <-- Result
</div>
<hr>
```

最后 Javascript 的 update_progress 函数代码如下:

```
function update_progress(status_url, nanobar, status_div) {
      // send GET request to status URL
      $.getJSON(status_url, function(data) {
          // update UI
          percent = parseInt(data['current'] * 100 / data['total']);
          nanobar.go(percent);
          $(status_div.childNodes[1]).text(percent + '%');
          $(status_div.childNodes[2]).text(data['status']);
          if (data['state'] != 'PENDING' && data['state'] != 'PROGRESS') {
              if ('result' in data) {
                  // show result
                  $(status_div.childNodes[3]).text('Result: ' + data['result']);
              }
              else {
                  // something unexpected happened
                  $(status_div.childNodes[3]).text('Result: ' + data['state']);
              }
          }
          else {
              // rerun in 2 seconds
              setTimeout(function() {
                  update_progress(status_url, nanobar, status_div);
              }, 2000);
          }
      });
  }
```

这一部分的代码就不一一解释了。

## 运行示例

首先下载代码，代码的位于 [Github repository](https://github.com/miguelgrinberg/flask-celery-example)，接着执行以下的命令:

```
$ git clone https://github.com/miguelgrinberg/flask-celery-example.git
$ cd flask-celery-example
$ virtualenv venv
$ source venv/bin/activate
(venv) $ pip install -r requirements.txt
```

接着，启动 redis，关于 redis 的安装，启动以及配置，请参阅 [Redis 文档](http://redis.io/documentation/)。

最后，执行如下命令运行示例:

```
$ export MAIL_USERNAME=<your-gmail-username>
$ export MAIL_PASSWORD=<your-gmail-password>
$ source venv/bin/activate
(venv) $ celery worker -A app.celery --loglevel=info
```

运行你的 Flask 应用来感受 Flask 和 Celery 一起工作的快乐:

```
$ source venv/bin/activate
(venv) $ python app.py
```
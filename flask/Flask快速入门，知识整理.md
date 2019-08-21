# Flask快速入门，知识整理

# 一、Flask介绍

　　Flask是一个基于Python开发并且依赖jinja2模板和Werkzeug WSGI服务的一个微型框架，对于Werkzeug本质是Socket服务端，其用于接收http请求并对请求进行预处理，然后触发Flask框架，开发人员基于Flask框架提供的功能对请求进行相应的处理，并返回给用户，如果要返回给用户复杂的内容时，需要借助jinja2模板来实现对模板的处理，即：将模板和数据进行渲染，将渲染后的字符串返回给用户浏览器。

“微”(micro) 并不表示你需要把整个 Web 应用塞进单个 Python 文件（虽然确实可以 ），也不意味着 Flask 在功能上有所欠缺。微框架中的“微”意味着 Flask 旨在保持核心简单而易于扩展。Flask 不会替你做出太多决策——比如使用何种数据库。而那些 Flask 所选择的——比如使用何种模板引擎——则很容易替换。除此之外的一切都由可由你掌握。如此，Flask 可以与您珠联璧合。

默认情况下，Flask 不包含数据库抽象层、表单验证，或是其它任何已有多种库可以胜任的功能。然而，Flask 支持用扩展来给应用添加这些功能，如同是 Flask 本身实现的一样。众多的扩展提供了数据库集成、表单验证、上传处理、各种各样的开放认证技术等功能。Flask 也许是“微小”的，但它已准备好在需求繁杂的生产环境中投入使用。

```
pip3 install flask
```

```python
#Flask依赖一个实现wsgi协议的模块：werkzeug
from werkzeug.wrappers import Request, Response

@Request.application
def hello(request):
    return Response('Hello World!')

if __name__ == '__main__':
    from werkzeug.serving import run_simple
    run_simple('localhost', 4000, hello)
```

flask依赖wsgi，实现wsgi模块：wsgiref,werkzeug,uwsgi

##  与Django的简单比较


- Django：无socket，依赖第三方模块wsgi，中间件，路由系统（CBV，FBV），视图函数，ORM、cookie、session、Admin、Form、缓存、信号、序列化。
- Flask：无socket、中间件（需要扩展），路由系统，视图（CBV）、第三方模块（依赖jinja2）、cookie、session。


# 二、基本使用

```
from flask import Flask
app = Flask(__name__)
 
@app.route('/')
def hello_world():
    return 'Hello World!'
 
if __name__ == '__main__':
    app.run()
```

##  1. 实例化Flask对象时，可选的参数

```
app = Flask(__name__)    # 这是实例化一个Flask对象，最基本的写法
# 但是Flask中还有其他参数，以下是可填的参数，及其默认值

def __init__(self, import_name, static_path=None, static_url_path=None,
                 static_folder='static', template_folder='templates',
                 instance_path=None, instance_relative_config=False,
                 root_path=None):
```

**template_folder**：模板所在文件夹的名字

**root_path**：可以不用填，会自动找到，当前执行文件，所在目录地址

在return render_template时会将上面两个进行拼接，找到对应的模板地址

**static_folder**：静态文件所在文件的名字，默认是static，可以不用填

**static_url_path**：静态文件的地址前缀，写成什么，访问静态文件时，就要在前面加上这个

```
app = Flask(__name__,template_folder='templates',static_url_path='/xxxxxx')
```

 如：在根目录下创建目录，templates和static，则return render_template时，可以找到里面的模板页面；如在static文件夹里存放11.png，在引用该图片时，静态文件地址为：/xxxxxx/11.png

 **instance_path**和**instance_relative_config**是配合来用的、

这两个参数是用来找配置文件的，当用app.config.from_pyfile('settings.py')这种方式导入配置文件的时候会用到

**instance_relative_config：**默认为False，当设置为True时**，**from_pyfile会从**instance_path**指定的地址下查找文件。

**instsnce_path**：指定from_pyfile查询文件的路径，不设置时，默认寻找和app.run()的执行文件同级目录下的instance文件夹；如果配置了instance_path（**注意需要是绝对路径**），就会从指定的地址下里面的文件

### 绑定路由关系的两种方式

```
#方式一
    @app.route('/index.html',methods=['GET','POST'],endpoint='index')
    def index():
        return 'Index'
        
#方式二

def index():
    return "Index"

self.add_url_rule(rule='/index.html', endpoint="index", view_func=index, methods=["GET","POST"])    #endpoint是别名
or
app.add_url_rule(rule='/index.html', endpoint="index", view_func=index, methods=["GET","POST"])
app.view_functions['index'] = index
```

**添加路由关系的本质：**将url和视图函数封装成一个Rule对象，添加到Flask的url_map字段中

##  2. Flask中装饰器应用

```
from flask import Flask,render_template,request,redirect,session
app = Flask(__name__)
app.secret_key = "sdsfdsgdfgdfgfh"   # 设置session时，必须要加盐，否则报错

def wrapper(func):
    def inner(*args,**kwargs):
        if not session.get("user_info"):
            return redirect("/login")
        ret = func(*args,**kwargs)
        return ret
    return inner

@app.route("/login",methods=["GET","POST"])  # 指定该路由可接收的请求方式，默认为GET
def login():
    if request.method=="GET":
        return render_template("login.html")
    else:
        # print(request.values)   #这个里面什么都有，相当于body
        username = request.form.get("username")
        password = request.form.get("password")
        if username=="haiyan" and password=="123":
            session["user_info"] = username
            # session.pop("user_info")  #删除session
            return redirect("/index")
        else:
            # return render_template("login.html",**{"msg":"用户名或密码错误"})
            return render_template("login.html",msg="用户名或者密码错误")

@app.route("/index",methods=["GET","POST"])
@wrapper    #自己定义装饰器时，必须放在路由的装饰器下面
def index():
    # if not session.get("user_info"):
    #     return redirect("/login")
    return render_template("index.html")


if __name__ == '__main__':
    app.run(debug=True) 
```

debug = True 是指进入调试模式，服务器会在 我们的代码修改后， 自动重新载入，有错误的话会提醒，每次修改代码后就不需要再手动重启

[点击查看详情](http://www.cnblogs.com/huchong/p/8227888.html#_label1)

## 3. 请求响应相关

### 1.获取请求数据，及相应

```
　　　　- request
            - request.form   #POST请求的数据
            - request.args   #GET请求的数据，不是完全意义上的字典，通过.to_dict可以转换成字典
            - request.querystring  #GET请求，bytes形式的
        - response
            - return render_tempalte()    
            - return redirect()
            - return ""
            v = make_response(返回值)  #可以把返回的值包在了这个函数里面，然后再通过.set_cookie绑定cookie等
        - session
            - 存在浏览器上，并且是加密的
            - 依赖于：secret_key
```

### 2.flask中获取URL后面的参数(from urllib.parse import urlencode,quote,unquote)

GET请求：

URL为:　 http://127.0.0.1:5000/login?name=%27%E8%83%A1%E5%86%B2%27&nid=2 

```
from urllib.parse import urlencode,quote,unquote

def login():
    if request.method == 'GET':
        s1 = request.args
        s2 = request.args.to_dict()
        s3 = urlencode(s1)
        s4 = urlencode(s2)
        s5 = unquote(s3)
        s6 = unquote(s4)
        s7 = quote("胡冲")
        print('s1',s1)
        print('s2',s2)
        print('s3',s3)
        print('s4',s4)
        print('s5',s5)
        print('s6',s6)
        print('s7',s7)

        return render_template('login.html')

#############结果如下####################

s1 ImmutableMultiDict([('name', "'胡冲'"), ('nid', '2')])
s2 {'name': "'胡冲'", 'nid': '2'}
s3 name=%27%E8%83%A1%E5%86%B2%27&nid=2
s4 name=%27%E8%83%A1%E5%86%B2%27&nid=2
s5 name='胡冲'&nid=2
s6 name='胡冲'&nid=2
s7 %E8%83%A1%E5%86%B2
```

# 三、配置文件

[ 点击查看](http://www.cnblogs.com/huchong/p/8227888.html#_label0) 

```python
flask中的配置文件是一个flask.config.Config对象（继承字典）,默认配置为：
    {
        'DEBUG':                                get_debug_flag(default=False),  是否开启Debug模式
        'TESTING':                              False,                          是否开启测试模式
        'PROPAGATE_EXCEPTIONS':                 None,                          
        'PRESERVE_CONTEXT_ON_EXCEPTION':        None,
        'SECRET_KEY':                           None,
        'PERMANENT_SESSION_LIFETIME':           timedelta(days=31),
        'USE_X_SENDFILE':                       False,
        'LOGGER_NAME':                          None,
        'LOGGER_HANDLER_POLICY':               'always',
        'SERVER_NAME':                          None,
        'APPLICATION_ROOT':                     None,
        'SESSION_COOKIE_NAME':                  'session',
        'SESSION_COOKIE_DOMAIN':                None,
        'SESSION_COOKIE_PATH':                  None,
        'SESSION_COOKIE_HTTPONLY':              True,
        'SESSION_COOKIE_SECURE':                False,
        'SESSION_REFRESH_EACH_REQUEST':         True,
        'MAX_CONTENT_LENGTH':                   None,
        'SEND_FILE_MAX_AGE_DEFAULT':            timedelta(hours=12),
        'TRAP_BAD_REQUEST_ERRORS':              False,
        'TRAP_HTTP_EXCEPTIONS':                 False,
        'EXPLAIN_TEMPLATE_LOADING':             False,
        'PREFERRED_URL_SCHEME':                 'http',
        'JSON_AS_ASCII':                        True,
        'JSON_SORT_KEYS':                       True,
        'JSONIFY_PRETTYPRINT_REGULAR':          True,
        'JSONIFY_MIMETYPE':                     'application/json',
        'TEMPLATES_AUTO_RELOAD':                None,
    }
 
方式一：
    app.config['DEBUG'] = True
 
    PS： 由于Config对象本质上是字典，所以还可以使用app.config.update(...)
 
方式二：
    app.config.from_pyfile("python文件名称")
        如：
            settings.py
                DEBUG = True
 
            app.config.from_pyfile("settings.py")
 
    app.config.from_envvar("环境变量名称")
        环境变量的值为python文件名称名称，内部调用from_pyfile方法
 
 
    app.config.from_json("json文件名称")
        JSON文件名称，必须是json格式，因为内部会执行json.loads
 
    app.config.from_mapping({'DEBUG':True})
        字典格式
 
    app.config.from_object("python类或类的路径")
 
        app.config.from_object('pro_flask.settings.TestingConfig')
 
        settings.py
 
            class Config(object):
                DEBUG = False
                TESTING = False
                DATABASE_URI = 'sqlite://:memory:'
 
            class ProductionConfig(Config):
                DATABASE_URI = 'mysql://user@localhost/foo'
 
            class DevelopmentConfig(Config):
                DEBUG = True
 
            class TestingConfig(Config):
                TESTING = True
 
        PS: 从sys.path中已经存在路径开始写
     
 
    PS: settings.py文件默认路径要放在程序root_path目录，如果instance_relative_config为True，则就是instance_path目录
```

#  四、路由系统

##  1.可传入参数：

```
@app.route('/user/<username>')   #常用的   不加参数的时候默认是字符串形式的
@app.route('/post/<int:post_id>')  #常用的   #指定int，说明是整型的
@app.route('/post/<float:post_id>')
@app.route('/post/<path:path>')
@app.route('/login', methods=['GET', 'POST'])
```

 

```python
DEFAULT_CONVERTERS = {
    'default':          UnicodeConverter,
    'string':           UnicodeConverter,
    'any':              AnyConverter,
    'path':             PathConverter,
    'int':              IntegerConverter,
    'float':            FloatConverter,
    'uuid':             UUIDConverter,
}
```

##  2.反向生成URL： url_for

endpoint("name")   #别名，相当于django中的name

```python
from flask import Flask, url_for

@app.route('/index',endpoint="xxx")  #endpoint是别名
def index():
    v = url_for("xxx")
    print(v)
    return "index"

@app.route('/zzz/<int:nid>',endpoint="aaa")  #endpoint是别名
def zzz(nid):
    v = url_for("aaa",nid=nid)
    print(v)
    return "index2"
```

## 3.  @app.route和app.add_url_rule参数

```
@app.route和app.add_url_rule参数：
            rule,                       URL规则
            view_func,                  视图函数名称
            defaults=None,              默认值,当URL中无参数，函数需要参数时，使用defaults={'k':'v'}为函数提供参数
            endpoint=None,              名称，用于反向生成URL，即： url_for('名称')
            methods=None,               允许的请求方式，如：["GET","POST"]
            

            strict_slashes=None,        对URL最后的 / 符号是否严格要求，
                                        如：
                                            @app.route('/index',strict_slashes=False)，　#当为False时，url上加不加斜杠都行
                                                访问 http://www.xx.com/index/ 或 http://www.xx.com/index均可
                                            @app.route('/index',strict_slashes=True)　　#当为True时，url后面必须不加斜杠
                                                仅访问 http://www.xx.com/index 
            redirect_to=None,           由原地址直接重定向到指定地址，原url有参数时，跳转到的新url也得传参，注意：新url中不用指定参数类型，直接用旧的参数的类型
                                        如：
                                            @app.route('/index/<int:nid>', redirect_to='/home/<nid>') # 访问index时，会直接自动跳转到home，执行home的函数，
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　不执行index的
                                            
　　　　　　　　　　　　　　　　　　　　　　　　　　或
                                            def func(adapter, nid):
                                                return "/home/888"
                                            @app.route('/index/<int:nid>', redirect_to=func)

            subdomain=None,             子域名访问
                                                from flask import Flask, views, url_for

                                                app = Flask(import_name=__name__)
                                                app.config['SERVER_NAME'] = 'haiyan.com:5000'


                                                @app.route("/", subdomain="admin")
                                                def static_index():
                                                    """Flask supports static subdomains
                                                    This is available at static.your-domain.tld"""
                                                    return "admin.xxx.com"

　　　　　　　　　　　　　　　　　　　　　　　　　　　　#动态生成
                                                @app.route("/dynamic", subdomain="<username>")
                                                def username_index(username):
                                                    """Dynamic subdomains are also supported
                                                    Try going to user1.your-domain.tld/dynamic"""
                                                    return username + ".your-domain.tld"


                                                if __name__ == '__main__':
                                                    app.run()
        所有的域名都得与IP做一个域名解析：
　　　　　　　　如果你想通过域名去访问，有两种解决方式：
　　　　　　　　　　方式一：
　　　　　　　　　　　　1、租一个域名   haiyan.lalala
　　　　　　　　　　　　2、租一个公网IP  49.8.5.62
　　　　　　　　　　　　3、域名解析：
                           haiyan.com    49.8.5.62
　　　　　　　　　　　　4、吧代码放在49.8.5.62这个服务器上，程序运行起来
　　　　　　　　　　　　　 用户可以通过IP进行访问
　　　　　　　　　　方式二：如果是自己测试用的就可以用这种方式。先在自己本地的文件中找
　　　　　　　　　　　　 C:\Windows\System32\drivers\etc  找到HOST，修改配置
　　　　　　　　　　　　然后吧域名修改成自己的本地服务器127.0.0.1
　　　　　　　　　　　　加上配置：app.config["SERVER_NAME"] = "haiyan.com:5000"
```

 

```
# =============== 子域名访问============
@app.route("/static_index", subdomain="admin")
def static_index():
    return "admin.bjg.com"

# ===========动态生成子域名===========
@app.route("/index",subdomain='<xxxxx>')
def index(xxxxx):
    return "%s.bjg.com" %(xxxxx,)
```

## 4.自定制正则路由匹配

扩展Flask的路由系统，让他支持正则,这个类必须这样写，必须去继承BaseConverter

```python
from flask import Flask,url_for
from werkzeug.routing import BaseConverter
    app = Flask(__name__)

    # 定义转换的类  class RegexConverter(BaseConverter):
        """
        自定义URL匹配正则表达式
        """

        def __init__(self, map, regex):
            super(RegexConverter, self).__init__(map)
            self.regex = regex

        def to_python(self, value):
            """
            路由匹配时，匹配成功后传递给视图函数中参数的值
            :param value: 
            :return: 
            """
            return int(value)

        def to_url(self, value):
            """
            使用url_for反向生成URL时，传递的参数经过该方法处理，返回的值用于生成URL中的参数
            :param value: 
            :return: 
            """
            val = super(RegexConverter, self).to_url(value)
            return val

    # 添加到converts中
    app.url_map.converters['regex'] = RegexConverter

    # 进行使用
    @app.route('/index/<regex("\d+"):nid>',endpoint='xx')
    def index(nid):
        url_for('xx',nid=123) 
        return "Index"

    if __name__ == '__main__':
        app.run()
```

http://docs.jinkan.org/docs/flask/views.html

https://aliang.org/Python/Flask-route.html

# 五、视图函数

## 1.Django中的CBV模式

![img](https://images2017.cnblogs.com/blog/1195739/201801/1195739-20180107160456565-1909212451.png)

##  2.Flask中的CBV和FBV

```python
def auth(func):
    def inner(*args, **kwargs):
        result = func(*args, **kwargs)
        return result
    return inner

class IndexView(views.MethodView):
    # methods = ['POST']  #只允许POST请求访问
    decorators = [auth,]  # 如果想给所有的get,post请求加装饰器，就可以这样来写，也可以单个指定

    def get(self):   #如果是get请求需要执行的代码
        v = url_for('index')
        print(v)
        return "GET"

    def post(self):  #如果是post请求执行的代码
        return "POST"

app.add_url_rule('/index', view_func=IndexView.as_view(name='index'))  #name即FBV中的endpoint，指别名

if __name__ == '__main__':
    app.run()
```

###  类视图及其使用

　　视图函数不能面向对象编程，利用类视图来代替视图函数来解决这个问题

#### 导入视图类 View

　from flask.views import View

#### 编写一个视图子类

```
class MyView(View): # MyView继承于View

    def test(self):  #  自定义的方法
        return '测试类视图'

    def dispatch_request(self):   # 必须重写这个方法
        resp = self.test()
        return resp
```

#### 利用View子类获取到一个视图方法

　　　　MyView.as_view('test')

　　　　　　注意：.as_view方法的放回值是一个方法，而且该方法的名字就是传进去的参数

#### 将获取到的视图方法和路径对应起来

　　　　app.add_url_rule('/test/', view_func=MyView.as_view('test')) # MyView.as_view('test') 返回的是一个方法

#### 类视图的原理

　　　　把as_view方法返回的结果赋值给view_func
　　　　as_view方法返回的是一个方法(注意：as_view方法传入的参数就是as_view返回的那个方法的名字)，该方法会调用dispatch_request方法
　　　　一旦路由进来，就会调用 dispatch_request 方法
　　　　类视图的目的就是实现逻辑分离、方便管理

 

```python
from flask import Flask
from flask.views import View

app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello World'

class MyView(View): # MyView继承于View

    def test(self):  #  自定义的方法
        return '测试类视图'

    def dispatch_request(self):   # 必须重写这个方法
        resp = self.test()
        return resp


app.add_url_rule('/test/', view_func=MyView.as_view('test')) # MyView.as_view('test') 返回的是一个方法

print(app.url_map)

if __name__ == '__main__':
    app.run(debug=True)

# 把as_view方法返回的结果赋值给view_func
# as_view方法返回的是一个方法(注意：as_view方法传入的参数就是as_view返回的那个方法的名字)，该方法会调用dispatch_request方法
# 一旦路由进来，就会调用 dispatch_request 方法
# 类视图的目的就是实现逻辑分离、方便管理
```

### 方法视图及其使用

　　利用视图函数实现不同的请求执行不同的逻辑时比较复杂，需要在视图函数函数中进行判断；如果利用方法视图实现就比较简单

```python
@app.route('/test/', methods=['GET', 'POST'])
def test():
    if request.method == 'GET':
        # 做GET的事情
        pass
    elif request.method == 'POST':
        # 做POST的事情
        pass
    return '测试'
```

#### 导入方法视图类　　

　　　　from flask.views import MethodView

#### 创建方法视图子类

```python
class TestMethodView(MethodView):
    def get(self):
        # 处理Get请求
        return 'GET请求'
    def post(self):
        # 处理post请求
        return 'POST请求'
```

注意：视图类中的方法就是支持的请求类型![img](https://images2017.cnblogs.com/blog/1057762/201709/1057762-20170918163803337-478209396.png)

#### 利用方法视图子类创建一个视图函数

　　　　TestMethodView.as_view('testMethodView')

 　　　　　　注意：as_view返回的是一个视图函数，而且该视图函数逇名称就是传进去的参数

####  将获取到的视图方法和路径对应起来

　　　　　　app.add_url_rule('/test02/', view_func=TestMethodView.as_view('testMethodView'))

```python
from flask import Flask
from flask import request
from flask.views import MethodView

app = Flask(__name__)

@app.route('/')
def index():
    return '测试主页面'

@app.route('/test/', methods=['GET', 'POST'])
def test():
    if request.method == 'GET':
        # 做GET的事情
        pass
    elif request.method == 'POST':
        # 做POST的事情
        pass
    return '测试'

class TestMethodView(MethodView):
    def get(self):
        # 处理Get请求
        return 'GET请求'
    def post(self):
        # 处理post请求
        return 'POST请求'

app.add_url_rule('/test02/', view_func=TestMethodView.as_view('testMethodView'))
# method = TestMethodView.as_view('testMethodView');
# app.add_url_rule('/test02/<name>/', view_func=method, methods=['GET'])

print(app.url_map)

if __name__ == '__main__':
    app.run(debug=True)
```

#### 虽然在方法视图中定义的函数就是支持的请求类型，但是我们可以在配置路径时指定哪个路径对应哪中类型的请求

#### 利用方法视图子类获取一个名字为testMethodView02的视图函数，该视图函数只能支持GET请求，而且支持转换器

```python
method02 = TestMethodView.as_view('testMethodView02');
app.add_url_rule('/test02/<name>/', view_func=method02, methods=['GET'])
```

#### 利用利用方法视图子类获取一个名字为testMethodView03的视图函数，该视图函数只能支持POST请求

```python
method03 = TestMethodView.as_view('testMethodView03')
app.add_url_rule('/test03/', view_func=method03, methods=['POST'])
```

　模拟POST请求

　　　　　　　　![img](https://images2017.cnblogs.com/blog/1057762/201709/1057762-20170918165632165-2077598826.png)

 

```python
from flask import Flask
from flask import request
from flask.views import MethodView

app = Flask(__name__)

@app.route('/')
def index():
    return '测试主页面'

@app.route('/test/', methods=['GET', 'POST'])
def test():
    if request.method == 'GET':
        # 做GET的事情
        pass
    elif request.method == 'POST':
        # 做POST的事情
        pass
    return '测试'

class TestMethodView(MethodView):
    def get(self, name):
        # 处理Get请求, 也可以在这些方法中调用其他的方法
        return 'GET请求' + name
    def post(self):
        # 处理post请求, 也可以在这些方法中调用其他的方法
        return 'POST请求'

# app.add_url_rule('/test02/', view_func=TestMethodView.as_view('testMethodView'))
method02 = TestMethodView.as_view('testMethodView02');
app.add_url_rule('/test02/<name>/', view_func=method02, methods=['GET'])
method03 = TestMethodView.as_view('testMethodView03')
app.add_url_rule('/test03/', view_func=method03, methods=['POST'])

print(app.url_map)

if __name__ == '__main__':
    app.run(debug=True)
```

# 六、请求与响应

```python
 from flask import Flask
 from flask import request
 from flask import render_template
 from flask import redirect
 from flask import make_response

    app = Flask(__name__)


    @app.route('/login.html', methods=['GET', "POST"])
    def login():

        # 请求相关信息
        # request.method
        # request.args
        # request.form
        # request.values
        # request.cookies
        # request.headers
        # request.path
        # request.full_path
        # request.script_root
        # request.url
        # request.base_url
        # request.url_root
        # request.host_url
        # request.host
        # request.files
        # obj = request.files['the_file_name']
        # obj.save('/var/www/uploads/' + secure_filename(f.filename))

        # 响应相关信息
        # return "字符串"
        # return render_template('html模板路径',**{})
        # return redirect('/index.html')

        # response = make_response(render_template('index.html'))
        # response是flask.wrappers.Response类型
        # response.delete_cookie('key')
        # response.set_cookie('key', 'value')
        # response.headers['X-Something'] = 'A value'
        # return response


        return "内容"

    if __name__ == '__main__':
        app.run()
```



```python
from flask import Flask,url_for,request,redirect,render_template,jsonify,make_response
from urllib.parse import urlencode,quote,unquote
app = Flask(__name__)

@app.route('/index',endpoint='xx')
def index():
    from werkzeug.datastructures import ImmutableMultiDict
　　=================
    # get_data = request.args
    # get_dict = get_data.to_dict()
    # get_dict['xx'] = '18'
    # url = urlencode(get_dict)
    # print(url)
　　====================
    # print(request.query_string)
    # print(request.args)
　　==========================
    # val = "%E6%8A%8A%E5%87%A0%E4%B8%AA"
    # print(unquote(val))   #把上面这样的数据转换成中文
    #
    # return "Index"

    # return "Index"
    # return redirect()
    # return render_template()
    # return jsonify(name='alex',age='18')  #相当于JsonResponse
　　=======================
    response = make_response('xxxxx')   ##如果是返回更多的值，cookie，headers，或者其他的就可用它
    response.headers['xxx'] = '123123'
    return response


if __name__ == '__main__':
    # app.__call__
    app.run()
```

# 七、模板语法

## 1、模板的使用

Flask使用的是Jinja2模板，所以其语法和Django无太大差别

Flask中模板里面，执行函数时，需要带（）才执行

### 1.为了防止xss攻击，加了验证，所以页面上显示字符串的形式，解决办法，有两种方法

**方法一：**在后端使用Markup，等价于Django里的mark_safe

 **方法二**：在前端使用safe

```
{{ v1|safe }}
```

 2.静态文件的两种导入方式 

**目录结构：**

**![img](https://images2017.cnblogs.com/blog/1195739/201801/1195739-20180112002129551-1702350524.png)**

 **方式一：**

 ![img](https://images2017.cnblogs.com/blog/1195739/201801/1195739-20180112002150441-1692454105.png)

**方式二：**

**![img](https://images2017.cnblogs.com/blog/1195739/201801/1195739-20180112002214519-340547811.png)**

 3.flask中模板语法不提示，解决办法

![img](https://images2017.cnblogs.com/blog/1195739/201801/1195739-20180112002339004-478741395.png)

 **点击选择jinja2** 

## 2、自定义模板方法

Flask中自定义模板方法的方式和Bottle相似，创建一个函数并通过参数的形式传入render_template，

**run.py**

```python
from flask import Flask,url_for,render_template,Markup
app = Flask(__name__)

def test(a,b):    #自定义的标签，此方法在使用时，需要在render_temlate中传入到指定以页面使用
    return a+b

@app.template_global()   # 不需要传入，可直接在页面使用
def sb(a1, a2):
    return a1 + a2 + 100


@app.template_filter()    #不需要传入，使用时要在一个值（此值作为第一个参数传入到过滤器中）的后面加入|，然后再加参数
def db(a1, a2, a3):
    return a1 + a2 + a3

@app.route('/index')
def index():
    v1 = "字符串"
    v2 = [11,22,33]
    v3 = {"k1":"v3","sdf":"sdgfgf"}
    v4 = "<input type='text' />"
    v5 = Markup("<input type='text' />")
    return render_template("index.html",v1=v1,v2=v2,v3=v3,v4=v4,v5=v5,test=test)

if __name__ == '__main__':
    app.run(debug=True)
```

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width">
    <title>Title</title>
</head>
<body>
{{ v1 }}
<ul>
    {% for foo in v2 %}
       <li>{{ foo }}</li>
    {% endfor %}
    {{ v2.1 }}

    {% for k,v in v3.items() %}
    <li>{{ k }} {{ v }}</li>
    {% endfor %}
    {{ v3.k1 }}
    {{ v3.get("k1") }}

    {{ v4|safe }}
    {{ v5 }}

    <h1>{{ test(1,2) }}</h1>
    <p>{{ sb(1,2) }}</p>
    <p>{{ 1| db(2,3) }}</p>
</ul>
</body>
</html>
```

PS:模板继承的方法和django的一样。 

## 3.宏

只有定义的东西在很多地方去使用的时候才去用它,

**html**

```
{% macro xx(name, type='text', value='') %}
    <input type="{{ type }}" name="{{ name }}" value="{{ value }}">
　　<input type="{{ type }}" name="{{ name }}" value="{{ value }}">　　<input type="{{ type }}" name="{{ name }}" value="{{ value }}">
{% endmacro %}

{{ xx('n1') }} 
```

相当于在页面上定义了一个名为xx的'函数'，这个函数接收3个参数，我们给type和value写上了默认值，此时调用，我们还需要传入一个参数，我们此时传入了一个n1，则

页面上会生成3个input框，name都为n1

# 八、Session

除请求对象之外，还有一个 session 对象。它允许你在不同请求间存储特定用户的信息。它是在 Cookies 的基础上实现的，并且对 Cookies 进行密钥签名要使用会话，你需要设置一个密钥。

- 设置：session['username'] ＝ 'xxx'
- 删除：session.pop('username', None)

```
from flask import Flask,url_for,session

app = Flask(__name__)
app.secret_key = "sdsfdgdgdgd"
app.config['SESSION_COOKIE_NAME'] = 'session_lvning'  #设置session的名字

@app.route('/index/')
def index(nid):
    #session本质上操作的是字典， 所有对session操作的方法与字典方法相同
    #session的原理：如果下一次访问的时候带着随机字符串，会把session里面对应的
    # 值拿到内存，假设session保存在数据库，每执行一次链接一次数据库，每次都要时时更新的话，会非常损耗数据库的效率
    session["xxx"] = 123
    session["xxx2"] = 123
    session["xxx3"] = 123
    session["xxx4"] = 123
    del session["xxx2"]  #在这删除了，真正存储的时候是没有xxx2的
    return "ddsf"

if __name__ == '__main__':
    app.run()
```

## 关于session的配置

```
 app.config['SESSION_COOKIE_NAME'] = 'session_lvning'
```

```
- session超时时间如何设置？      'PERMANENT_SESSION_LIFETIME':           timedelta(days=31)
 以下是跟session相关的配置文件
"""
            'SESSION_COOKIE_NAME':                  'session',
            'SESSION_COOKIE_DOMAIN':                None,
            'SESSION_COOKIE_PATH':                  None,
            'SESSION_COOKIE_HTTPONLY':              True,
            'SESSION_COOKIE_SECURE':                False,
            'SESSION_REFRESH_EACH_REQUEST':         True,  #是否每次都跟新
            'PERMANENT_SESSION_LIFETIME':           timedelta(days=31)
```

基本使用

```python
from flask import Flask, session, redirect, url_for, escape, request
 
app = Flask(__name__)
 
@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'
 
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form action="" method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''
 
@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))
 
# set the secret key.  keep this really secret:
app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'
```

自定义Session

```
pip3 install Flask-Session
        
        run.py
            from flask import Flask
            from flask import session
            from pro_flask.utils.session import MySessionInterface
            app = Flask(__name__)

            app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'
            app.session_interface = MySessionInterface()

            @app.route('/login.html', methods=['GET', "POST"])
            def login():
                print(session)
                session['user1'] = 'alex'
                session['user2'] = 'alex'
                del session['user2']

                return "内容"

            if __name__ == '__main__':
                app.run()

        session.py
            #!/usr/bin/env python
            # -*- coding:utf-8 -*-
            import uuid
            import json
            from flask.sessions import SessionInterface
            from flask.sessions import SessionMixin
            from itsdangerous import Signer, BadSignature, want_bytes


            class MySession(dict, SessionMixin):
                def __init__(self, initial=None, sid=None):
                    self.sid = sid
                    self.initial = initial
                    super(MySession, self).__init__(initial or ())


                def __setitem__(self, key, value):
                    super(MySession, self).__setitem__(key, value)

                def __getitem__(self, item):
                    return super(MySession, self).__getitem__(item)

                def __delitem__(self, key):
                    super(MySession, self).__delitem__(key)



            class MySessionInterface(SessionInterface):
                session_class = MySession
                container = {}

                def __init__(self):
                    import redis
                    self.redis = redis.Redis()

                def _generate_sid(self):
                    return str(uuid.uuid4())

                def _get_signer(self, app):
                    if not app.secret_key:
                        return None
                    return Signer(app.secret_key, salt='flask-session',
                                  key_derivation='hmac')

                def open_session(self, app, request):
                    """
                    程序刚启动时执行，需要返回一个session对象
                    """
                    sid = request.cookies.get(app.session_cookie_name)
                    if not sid:
                        sid = self._generate_sid()
                        return self.session_class(sid=sid)

                    signer = self._get_signer(app)
                    try:
                        sid_as_bytes = signer.unsign(sid)
                        sid = sid_as_bytes.decode()
                    except BadSignature:
                        sid = self._generate_sid()
                        return self.session_class(sid=sid)

                    # session保存在redis中
                    # val = self.redis.get(sid)
                    # session保存在内存中
                    val = self.container.get(sid)

                    if val is not None:
                        try:
                            data = json.loads(val)
                            return self.session_class(data, sid=sid)
                        except:
                            return self.session_class(sid=sid)
                    return self.session_class(sid=sid)

                def save_session(self, app, session, response):
                    """
                    程序结束前执行，可以保存session中所有的值
                    如：
                        保存到resit
                        写入到用户cookie
                    """
                    domain = self.get_cookie_domain(app)
                    path = self.get_cookie_path(app)
                    httponly = self.get_cookie_httponly(app)
                    secure = self.get_cookie_secure(app)
                    expires = self.get_expiration_time(app, session)

                    val = json.dumps(dict(session))

                    # session保存在redis中
                    # self.redis.setex(name=session.sid, value=val, time=app.permanent_session_lifetime)
                    # session保存在内存中
                    self.container.setdefault(session.sid, val)

                    session_id = self._get_signer(app).sign(want_bytes(session.sid))

                    response.set_cookie(app.session_cookie_name, session_id,
                                        expires=expires, httponly=httponly,
                                        domain=domain, path=path, secure=secure)
```

第三方session

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
"""
pip3 install redis
pip3 install flask-session

"""


from flask import Flask, session, redirect
from flask.ext.session import Session


app = Flask(__name__)
app.debug = True
app.secret_key = 'asdfasdfasd'


app.config['SESSION_TYPE'] = 'redis'
from redis import Redis
app.config['SESSION_REDIS'] = Redis(host='192.168.0.94',port='6379')
Session(app)


@app.route('/login')
def login():
    session['username'] = 'alex'
    return redirect('/index')


@app.route('/index')
def index():
    name = session['username']
    return name


if __name__ == '__main__':
    app.run()
```

## Django和Flask中session的区别

```
'''
Django中，session保存在服务端的数据库中，数据库中保存请求用户的所有数据，服务端数据中{'随机字符串'：加密后的客户相关信息}
请求完成后，把随机字符串作为值，返回给客户端，保存在客户端的cookie中，键为：sessionid，值为：服务端返回的随机字符串；即{'sessionid':'随机字符串'}
    

        
Flask中，服务端什么都不存，用户第一次请求时，在内存中生成一个空字典，将这个空字典加密后，返回给客户端，保存在客户端的cookie中，键为’session',值为:加密后的字典
下次访问时，读取客户端cookie中key为session对应的值
然后进行解密（如果不能按之前的的加密方式对应个解密方式解密，即认为第一次请求，重新生成空字典），解密成功后，可以对字典进行操作，保存新数据在字典中，请求完成后，会重新加密这个字典,返回个客户端保存

'''
```

# 九、蓝图（flask中多py文件拆分都要用到蓝图）

如果代码非常多，要进行归类。不同的功能放在不同的文件，吧相关的视图函数也放进去。蓝图也就是对flask的目录结构进行分配（应用于小，中型的程序）

蓝图用于为应用提供目录划分：

## 小中型：

![img](https://images2017.cnblogs.com/blog/1195739/201801/1195739-20180109153505097-1906265054.png)

  **manage.py**

```
import fcrm
if __name__ == '__main__':
    fcrm.app.run()
```

**__init__.py**(只要一导入fcrm就会执行__init__.py文件)

```
from flask import Flask
#导入accout 和order
from fcrm.views import accout
from fcrm.views import order
app = Flask(__name__)
print(app.root_path)  #根目录

app.register_blueprint(accout.accout)  #把蓝图注册到app里面，accout.accout是创建的蓝图对象
app.register_blueprint(order.order)
```

**accout.py**

```
from flask import  Blueprint,render_template
accout = Blueprint("accout",__name__)

@accout.route('/accout')
def xx():
    return "accout"

@accout.route("/login")
def login():
    return render_template("login.html")
```

**order.py

```
from flask import Blueprint
order = Blueprint("order",__name__)

@order.route('/order')
def register():   
    return "order
```

小型应用程序：[代码示例](https://files.cnblogs.com/files/huchong/pro_flask_简单应用程序目录示例.zip) 

## 大型：

![img](https://images2017.cnblogs.com/blog/1195739/201801/1195739-20180109153849004-1739492250.png)

  ![img](https://images2017.cnblogs.com/blog/1195739/201801/1195739-20180109153920738-2108982455.png)

![img](https://images2017.cnblogs.com/blog/1195739/201801/1195739-20180109153947363-940880815.png)

 

 大型应用程序：[代码示例](https://files.cnblogs.com/files/huchong/pro_flask_大型应用目录示例.zip)

## 注意：

**蓝图中的视图函数的名字不能和蓝图对象的名字一样！！！**

## 其他：

- 蓝图URL前缀：xxx = Blueprint('account', __name__,url_prefix='/xxx')
- 蓝图子域名：xxx = Blueprint('account', __name__,subdomain='admin')
    \# 前提需要给配置SERVER_NAME： app.config['SERVER_NAME'] = 'hc.com:5000'
    \# 访问时：admin.hc.com:5000/login.html 

# 十、闪现（flash）

session存在在服务端的一个字典里面，session保存起来，取一次里面还是有的，直到你删除之后才没有了

## 1、本质

flash是基于session创建的，flash支持往里边放值，只要你取一下就没有了，相当于pop了一下。不仅可以拿到值，而且可以把其从session里的去掉，

基于Session实现的用于保存数据的集合，其特点是：使用一次就删除。

## 2、闪现的用途

某个数据仅需用一次时，可以使用闪现

```
from flask import Flask,session,Session,flash,get_flashed_messages,redirect,render_template,request
app = Flask(__name__)
app.secret_key ='sdfsdfsdf'

@app.route('/users')
def users():
    # 方式一
    # msg = request.args.get('msg','')
    # 方式二
    # msg = session.get('msg')
    # if msg:
    #     del session['msg']
    # 方式三
    v = get_flashed_messages()　　# 获取flash中的值
    print(v)
    msg = ''
    return render_template('users.html',msg=msg)

@app.route('/useradd')
def user_add():
    # 在数据库中添加一条数据
    # 假设添加成功，在跳转到列表页面时，显示添加成功
    # 方式一
    # return redirect('/users?msg=添加成功')
    # 方式二
    # session['msg'] = '添加成功'
    # 方式三
    flash('添加成功')
    return redirect('/users')


if __name__ == '__main__':
    app.run(debug=True)
```

# 十一、中间件

在函数执行之前或函数执行之后想做点事情，有2种方式

第一种：装饰器

第二种：flask里面的扩展，相当于django中的中间件

```
from flask import Flask,session,Session,flash,get_flashed_messages,redirect,render_template,request
app = Flask(__name__)
app.secret_key ='sdfsdfsdf'

@app.before_request
def process_request1():
    print('process_request1')

@app.after_request
def process_response1(response):
    print('process_response1')
    return response


@app.before_request
def process_request2():
    print('process_request2')

@app.after_request
def process_response2(response):   #参数也得有
    print('process_response2')
    return response   #必须有返回值


@app.route('/index')
def index():
    print('index')
    return 'Index'

@app.route('/order')
def order():
    print('order')
    return 'order'

@app.route('/test')
def test():
    print('test')
    return 'test'

if __name__ == '__main__':
    app.run()
```

运行结果：

![img](https://images2017.cnblogs.com/blog/1184802/201801/1184802-20180104233933237-2120353157.png)

还有一个@app.before_first_request：表示，当程序运行起来，第一个请求来的时候就只执行一次，下次再来就不会在执行了

# 十二、请求扩展

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from flask import Flask, Request, render_template

app = Flask(__name__, template_folder='templates')
app.debug = True


@app.before_first_request    # 只在第一次请求到来时执行一次，后面不会再执行
def before_first_request1():
    print('before_first_request1')


@app.before_first_request
def before_first_request2():
    print('before_first_request2')


@app.before_request    # 每次请求到来时，都会执行
def before_request1():
    Request.nnn = 123
    print('before_request1')


@app.before_request
def before_request2():
    print('before_request2')


@app.after_request    # 每次响应时执行
def after_request1(response):
    print('before_request1', response)
    return response


@app.after_request
def after_request2(response):
    print('before_request2', response)
    return response


@app.errorhandler(404)
def page_not_found(error):
    return 'This page does not exist', 404


@app.template_global()    # 自定义标签，所有页面都直接使用
def sb(a1, a2):
    return a1 + a2


@app.template_filter()    # 自定义过滤器，所有页面都直接使用
def db(a1, a2, a3):
    return a1 + a2 + a3


@app.route('/')    # 访问的url，不加其他后缀时，也要有/
def hello_world():
    return render_template('hello.html')


if __name__ == '__main__':
    app.run()
```

> 自定义标签和过滤器在页面上的调用方式：{{sb(1,2)}}  {{ 1|db(2,3)}}

#  十三、编写统一的404页面



## 导入abort方法

　　　　from flask import abort

## 设置一个站位符，当404错误出现时，自己编写的404页面就会在占位符的位置进行显示

　　　　abort(404)　　

## 利用钩子编写自己的404页面

```
@app.errorhandler(404)  # 404页面钩子
def page_404(er):  # 参数是原始的404页面提示信息
    print(er)
    return '这是统一的错误页面', 404, {}  # 返回自己编写的404页面信息
```

![img](https://images2018.cnblogs.com/blog/1195739/201804/1195739-20180425102513168-1283642565.png)

```python
from flask import Flask
from flask import abort

app = Flask(__name__)

@app.route('/')
def index():
    return '测试主页面'

movies = [1,2,3,4,5]

@app.route('/movie/<int:num>/')
def movie(num):
    if num in movies:
        return '电影 {} 的详细信息为：...... '.format(num)
    abort(404)  # 自己编写的404页面会显示在这里

@app.errorhandler(404)  # 404页面钩子
def page_404(er):  # 参数是原始的404页面提示信息
    print(er)
    return '这是统一的错误页面', 404, {}  # 返回自己编写的404页面信息


print(app.url_map)

if __name__ == '__main__':
    app.run(debug=True)
```
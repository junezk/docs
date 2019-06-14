# Flask-Login

Flask-Login 为 Flask 提供了用户会话管理。它处理了日常的登入，登出并且长时间记住用户的会话。

它会:

- 在会话中存储当前活跃的用户 ID，让你能够自由地登入和登出。
- 让你限制登入(或者登出)用户可以访问的视图。
- 处理让人棘手的 “记住我” 功能。
- 帮助你保护用户会话免遭 cookie 被盗的牵连。
- 可以与以后可能使用的 Flask-Principal 或其它认证扩展集成。

但是，它不会:

- 限制你使用特定的数据库或其它存储方法。如何加载用户完全由你决定。
- 限制你使用用户名和密码，OpenIDs，或者其它的认证方法。
- 处理超越 “登入或者登出” 之外的权限。
- 处理用户注册或者账号恢复。

## 配置你的应用

对一个使用 Flask-Login 的应用最重要的一部分就是 [`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager) 类。你应该在你的代码的某处为应用创建一个，像这样:

```
login_manager = LoginManager()
```

登录管理(login manager)包含了让你的应用和 Flask-Login 协同工作的代码，比如怎样从一个 ID 加载用户，当用户需要登录的时候跳转到哪里等等。

一旦实际的应用对象创建后，你能够这样配置它来实现登录:

```
login_manager.init_app(app)
```

## 它是如何工作

你必须提供一个 [`user_loader`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.user_loader) 回调。这个回调用于从会话中存储的用户 ID 重新加载用户对象。它应该接受一个用户的 [`unicode`](http://docs.python.org/library/functions.html#unicode) ID 作为参数，并且返回相应的用户对象。比如:

```
@login_manager.user_loader
def load_user(userid):
    return User.get(userid)
```

如果 ID 无效的话，它应该返回 [`None`](http://docs.python.org/library/constants.html#None) (**而不是抛出异常**)。(在这种情况下，ID 会被手动从会话中移除且处理会继续)

## 你的用户类

你用来表示用户的类需要实现这些属性和方法:

- `is_authenticated`

  当用户通过验证时，也即提供有效证明时返回 [`True`](http://docs.python.org/library/constants.html#True) 。（只有通过验证的用户会满足 [`login_required`](http://www.pythondoc.com/flask-login/#flask.ext.login.login_required) 的条件。）

- `is_active`

  如果这是一个活动用户且通过验证，账户也已激活，未被停用，也不符合任何你 的应用拒绝一个账号的条件，返回 [`True`](http://docs.python.org/library/constants.html#True) 。不活动的账号可能不会登入（当然， 是在没被强制的情况下）。

- `is_anonymous`

  如果是一个匿名用户，返回 [`True`](http://docs.python.org/library/constants.html#True) 。（真实用户应返回 [`False`](http://docs.python.org/library/constants.html#False) 。）

- `get_id()`

  返回一个能唯一识别用户的，并能用于从 [`user_loader`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.user_loader) 回调中加载用户的 [`unicode`](http://docs.python.org/library/functions.html#unicode) 。注意着 **必须** 是一个 [`unicode`](http://docs.python.org/library/functions.html#unicode) —— 如果 ID 原本是 一个 [`int`](http://docs.python.org/library/functions.html#int) 或其它类型，你需要把它转换为 [`unicode`](http://docs.python.org/library/functions.html#unicode) 。

要简便地实现用户类，你可以从 [`UserMixin`](http://www.pythondoc.com/flask-login/#flask.ext.login.UserMixin) 继承，它提供了对所有这些方法的默认 实现。（虽然这不是必须的。）

## Login 示例

一旦用户通过验证，你可以使用 [`login_user`](http://www.pythondoc.com/flask-login/#flask.ext.login.login_user) 函数让用户登录。例如:

```
@app.route('/login', methods=['GET', 'POST'])
def login():
    # Here we use a class of some kind to represent and validate our
    # client-side form data. For example, WTForms is a library that will
    # handle this for us, and we use a custom LoginForm to validate.
    form = LoginForm()
    if form.validate_on_submit():
        # Login and validate the user.
        # user should be an instance of your `User` class
        login_user(user)

        flask.flash('Logged in successfully.')

        next = flask.request.args.get('next')
        # next_is_valid should check if the user has valid
        # permission to access the `next` url
        if not next_is_valid(next):
            return flask.abort(400)

        return flask.redirect(next or flask.url_for('index'))
    return flask.render_template('login.html', form=form)
```

*警告:* 你必须验证 [`next`](http://docs.python.org/library/functions.html#next) 参数的值。如果不验证的话，你的应用将会受到重定向的攻击。

就这么简单。你可用使用 [`current_user`](http://www.pythondoc.com/flask-login/#flask.ext.login.current_user) 代理来访问登录的用户，在每一个模板中都可以使用 [`current_user`](http://www.pythondoc.com/flask-login/#flask.ext.login.current_user):

```
{% if current_user.is_authenticated() %}
  Hi {{ current_user.name }}!
{% endif %}
```

需要用户登入 的视图可以用 [`login_required`](http://www.pythondoc.com/flask-login/#flask.ext.login.login_required) 装饰器来装饰:

```
@app.route("/settings")
@login_required
def settings():
    pass
```

当用户要登出时:

```
@app.route("/logout")
@login_required
def logout():
    logout_user()
    return redirect(somewhere)
```

他们会被登出，且他们会话产生的任何 cookie 都会被清理干净。

## 定制登入过程

默认情况下，当未登录的用户尝试访问一个 [`login_required`](http://www.pythondoc.com/flask-login/#flask.ext.login.login_required) 装饰的视图，Flask-Login 会闪现一条消息并且重定向到登录视图。(如果未设置登录视图，它将会以 401 错误退出。)

登录视图的名称可以设置成 [`LoginManager.login_view`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.login_view)。例如:

```
login_manager.login_view = "users.login"
```

默认的闪现消息是 `Please log in to access this page.`。要自定义该信息，请设置 [`LoginManager.login_message`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.login_message):

```
login_manager.login_message = u"Bonvolu ensaluti por uzi tio pa臐o."
```

要自定义消息分类的话，请设置 `LoginManager.login_message_category`:

```
login_manager.login_message_category = "info"
```

当重定向到登入视图，它的请求字符串中会有一个 `next` 变量，其值为用户之前访问的页面。

如果你想要进一步自定义登入过程，请使用 [`LoginManager.unauthorized_handler`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.unauthorized_handler) 装饰函数:

```
@login_manager.unauthorized_handler
def unauthorized():
    # do stuff
    return a_response
```

## 使用授权头(Authorization header)登录

Caution

该方法将会被弃用，使用下一节的 `request_loader` 来代替。

有些时候你要支持使用 `Authorization` 头的基本认证登录，比如 API 请求。为了支持通过头登录你需要提供一个 [`header_loader`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.header_loader) 回调。这个回调和 [`user_loader`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.user_loader) 回调作用一样，只是它接受一个 HTTP 头(Authorization)而不是用户 id。例如:

```
@login_manager.header_loader
def load_user_from_header(header_val):
    header_val = header_val.replace('Basic ', '', 1)
    try:
        header_val = base64.b64decode(header_val)
    except TypeError:
        pass
    return User.query.filter_by(api_key=header_val).first()
```

默认情况下 `Authorization` 的值传给 [`header_loader`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.header_loader) 回调。你可以使用 `AUTH_HEADER_NAME` 配置来修改使用的 HTTP 头(可以不使用 `Authorization`，使用 `Token`)。

## 使用 Request Loader 定制登录

有时你想要不使用 cookies 情况下登录用户，比如使用 HTTP 头或者一个作为查询参数的 api 密钥。这种情况下，你应该使用 `request_loader` 回调。这个回调和 [`user_loader`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.user_loader) 回调作用一样，但是 [`user_loader`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.user_loader) 回调只接受 Flask 请求而不是一个 user_id。

例如，为了同时支持一个 url 参数和使用 `Authorization` 头的基本用户认证的登录:

```
@login_manager.request_loader
def load_user_from_request(request):

    # first, try to login using the api_key url arg
    api_key = request.args.get('api_key')
    if api_key:
        user = User.query.filter_by(api_key=api_key).first()
        if user:
            return user

    # next, try to login using Basic Auth
    api_key = request.headers.get('Authorization')
    if api_key:
        api_key = api_key.replace('Basic ', '', 1)
        try:
            api_key = base64.b64decode(api_key)
        except TypeError:
            pass
        user = User.query.filter_by(api_key=api_key).first()
        if user:
            return user

    # finally, return None if both methods did not login the user
    return None
```

## 匿名用户

默认情况下，当一个用户没有真正地登录，[`current_user`](http://www.pythondoc.com/flask-login/#flask.ext.login.current_user) 被设置成一个 `AnonymousUserMixin` 对象。它由如下的属性和方法:

- `is_active` 和 `is_authenticated` 的值为 [`False`](http://docs.python.org/library/constants.html#False)
- `is_anonymous` 的值为 [`True`](http://docs.python.org/library/constants.html#True)
- `get_id()` 返回 [`None`](http://docs.python.org/library/constants.html#None)

如果需要为匿名用户定制一些需求(比如，需要一个权限域)，你可以向 [`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager)提供一个创建匿名用户的回调（类或工厂函数）:

```
login_manager.anonymous_user = MyAnonymousUser
```

## 记住我

“记住我”的功能很难实现。但是，Flask-Login 几乎透明地实现它 - 只要把 `remember=True` 传递给 [`login_user`](http://www.pythondoc.com/flask-login/#flask.ext.login.login_user)。一个 cookie 将会存储在用户计算机中，如果用户会话中没有用户 ID 的话，Flask-Login 会自动地从 cookie 中恢复用户 ID。cookie 是防纂改的，因此如果用户纂改过它(比如，使用其它的一些东西来代替用户的 ID)，它就会被拒绝，就像不存在。

该层功能是被自动实现的。但你能（且应该，如果你的应用处理任何敏感的数据）提供 额外基础工作来增强你记住的 cookie 的安全性。

### 可选令牌

使用用户 ID 作为记住的令牌值不一定是安全的。更安全的方法是使用用户名和密码联合的 hash 值，或类似的东西。要添加一个额外的令牌，向你的用户对象添加一个方法：

- `get_auth_token()`

  返回用户的认证令牌（返回为 [`unicode`](http://docs.python.org/library/functions.html#unicode) ）。这个认证令牌应能唯一识别用户，且不易通过用户的公开信息，如 UID 和名称来猜测出——同样也不应暴露这些信息。

相应地，你应该在 [`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager) 上设置一个 [`token_loader`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.token_loader) 函数， 它接受令牌（存储在 cookie 中）作为参数并返回合适的 User 对象。

[`make_secure_token`](http://www.pythondoc.com/flask-login/#flask.ext.login.make_secure_token) 函数用于便利创建认证令牌。它会连接所有的参数，然后用应用的密钥来 HMAC 它确保最大的密码学安全。（如果你永久地在数据库中存储用户令牌，那么你会希望向令牌中添加随机数据来阻碍猜测。）

如果你的应用使用密码来验证用户，在认证令牌中包含密码（或你应使用的加盐值的密码 hash ）能确保若用户更改密码，他们的旧认证令牌会失效。

### ”新鲜的“登录(Fresh Logins)

当用户登入，他们的会话被标记成“新鲜的”，就是说在这个会话只中用户实际上登录过。当会话销毁用户使用“记住我”的 cookie 重新登入，会话被标记成“非新鲜的”。[`login_required`](http://www.pythondoc.com/flask-login/#flask.ext.login.login_required) 并不在意它们之间的不同，这适用于大部分页面。然而，更改某人 的个人信息这样的敏感操作应需要一个“新鲜的”的登入。（像修改密码这样的操作总是需要 密码，无论是否重登入。）

[`fresh_login_required`](http://www.pythondoc.com/flask-login/#flask.ext.login.fresh_login_required)，除了验证用户登录，也将确保他们的登录是“新鲜的”。如果不是“新鲜的”，它会把用户送到可以重输入验证条件的页面。你可以定制 [`fresh_login_required`](http://www.pythondoc.com/flask-login/#flask.ext.login.fresh_login_required) 就像定制 [`login_required`](http://www.pythondoc.com/flask-login/#flask.ext.login.login_required) 那样，通过设置 [`LoginManager.refresh_view`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.refresh_view)，[`needs_refresh_message`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.needs_refresh_message)，和`needs_refresh_message_category`:

```
login_manager.refresh_view = "accounts.reauthenticate"
login_manager.needs_refresh_message = (
    u"To protect your account, please reauthenticate to access this page."
)
login_manager.needs_refresh_message_category = "info"
```

或者提供自己的回调来处理“非新鲜的”刷新：

```
@login_manager.needs_refresh_handler
def refresh():
    # do stuff
    return a_response
```

调用 [`confirm_login`](http://www.pythondoc.com/flask-login/#flask.ext.login.confirm_login) 函数可以重新标记会话为”新鲜“。

### Cookie 设置

cookie 的细节可以在应用设置中定义。

| `REMEMBER_COOKIE_NAME`     | 存储“记住我”信息的 cookie 名。 **默认值**： remember_token   |
| -------------------------- | ------------------------------------------------------------ |
| `REMEMBER_COOKIE_DURATION` | cookie 过期时间，为一个 [`datetime.timedelta`](http://docs.python.org/library/datetime.html#datetime.timedelta) 对象。 **默认值：** 365 天 (1 非闰阳历年) |
| `REMEMBER_COOKIE_DOMAIN`   | 如果“记住我” cookie 应跨域，在此处设置域名值 （即 .example.com 会允许 example 下所有子域 名）。 **默认值：** None |
| `REMEMBER_COOKIE_PATH`     | 限制”记住我“ cookie 存储到某一路径下。 **默认值：** `/`      |
| `REMEMBER_COOKIE_SECURE`   | 限制 “Remember Me” cookie 在某些安全通道下有用 （典型地 HTTPS）。**默认值：** [`None`](http://docs.python.org/library/constants.html#None) |
| `REMEMBER_COOKIE_HTTPONLY` | 保护 “Remember Me” cookie 不能通过客户端脚本访问。 **默认值：** [`False`](http://docs.python.org/library/constants.html#False) |

## 会话保护

当上述特性保护“记住我”令牌免遭 cookie 窃取时，会话 cookie 仍然是脆弱的。 Flask-Login 包含了会话保护来帮助阻止用户会话被盗用。

你可以在 [`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager) 上和应用配置中配置会话保护。如果它被启用，它可以在 `basic` 或 `strong` 两种模式中运行。要在 [`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager) 上设置它，设置 `session_protection` 属性为 `"basic"` 或 `"strong"`:

```
login_manager.session_protection = "strong"
```

或者，禁用它:

```
login_manager.session_protection = None
```

默认，它被激活为 `"basic"` 模式。它可以在应用配置中设定 `SESSION_PROTECTION` 为 [`None`](http://docs.python.org/library/constants.html#None) 、 `"basic"` 或 `"strong"` 来禁用。

当启用了会话保护，每个请求，它生成一个用户电脑的标识（基本上是 IP 地址和 User Agent 的 MD5 hash 值）。如果会话不包含相关的标识，则存储生成的。如果存在标识，则匹配生成的，之后请求可用。

在 `basic` 模式下或会话是永久的，如果该标识未匹配，会话会简单地被标记为非活 跃的，且任何需要活跃登入的东西会强制用户重新验证。（当然，你必须已经使用了活跃登入机制才能奏效。）

在 `strong` 模式下的非永久会话，如果该标识未匹配，整个会话（记住的令牌如果存在，则同样）被删除。

## 本地化

默认情况下，当用户需要登录，[`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager) 使用 `flash` 来显示信息。这些信息都是英文的。如果你需要本地化，设置 [`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager) 的 `localize_callback` 属性为一个函数，该函数在消息被发送到 `flash` 的时候被调用，比如，`gettext`。

## API 文档

这部分文档是从 Flask-Login 源码中自动生成的。

### 配置登录

- *class* `flask.ext.login.`**LoginManager**(*app=None*, *add_context_processor=True*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#LoginManager)

  This object is used to hold the settings used for logging in. Instances of[`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager) are *not* bound to specific apps, so you can create one in the main body of your code and then bind it to your app in a factory function.

  `setup_app`(*app*, *add_context_processor=True*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#LoginManager.setup_app)

  This method has been deprecated. Please use `LoginManager.init_app()`instead.

  `unauthorized`()[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#LoginManager.unauthorized)

  This is called when the user is required to log in. If you register a callback with [`LoginManager.unauthorized_handler()`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.unauthorized_handler), then it will be called. Otherwise, it will take the following actions:Flash [`LoginManager.login_message`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.login_message) to the user.If the app is using blueprints find the login view for the current blueprint using `blueprint_login_views`. If the app is not using blueprints or the login view for the current blueprint is not specified use the value of [`login_view`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.login_view). Redirect the user to the login view. (The page they were attempting to access will be passed in the `next` query string variable, so you can redirect there if present instead of the homepage.)If [`LoginManager.login_view`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.login_view) is not defined, then it will simply raise a HTTP 401 (Unauthorized) error instead.This should be returned from a view or before/after_request function, otherwise the redirect will have no effect.

  `needs_refresh`()[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#LoginManager.needs_refresh)

  This is called when the user is logged in, but they need to be reauthenticated because their session is stale. If you register a callback with [`needs_refresh_handler`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.needs_refresh_handler), then it will be called. Otherwise, it will take the following actions:

  - Flash [`LoginManager.needs_refresh_message`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.needs_refresh_message) to the user.
  - Redirect the user to [`LoginManager.refresh_view`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.refresh_view). (The page they were attempting to access will be passed in the `next`query string variable, so you can redirect there if present instead of the homepage.)

  If [`LoginManager.refresh_view`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.refresh_view) is not defined, then it will simply raise a HTTP 401 (Unauthorized) error instead.
  
  This should be returned from a view or before/after_request function, otherwise the redirect will have no effect.
  
  **General Configuration**
  
  `user_loader`(*callback*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#LoginManager.user_loader)
  
  This sets the callback for reloading a user from the session. The function you set should take a user ID (a `unicode`) and return a user object, or `None` if the user does not exist.
  
  Parameters:**callback** ([*callable*](http://docs.python.org/library/functions.html#callable)) – The callback for retrieving a user object.
  
  `header_loader`(*callback*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#LoginManager.header_loader)
  
  This sets the callback for loading a user from a header value. The function you set should take an authentication token and return a user object, or [`None`](http://docs.python.org/library/constants.html#None)if the user does not exist.
  
  Parameters:**callback** ([*callable*](http://docs.python.org/library/functions.html#callable)) – The callback for retrieving a user object.
  
  `token_loader`(*callback*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#LoginManager.token_loader)
  
  This sets the callback for loading a user from an authentication token. The function you set should take an authentication token (a `unicode`, as returned by a user’s `get_auth_token` method) and return a user object, or `None` if the user does not exist.
  
  Parameters:**callback** ([*callable*](http://docs.python.org/library/functions.html#callable)) – The callback for retrieving a user object.
  
  `anonymous_user`
  
  A class or factory function that produces an anonymous user, which is used when no one is logged in.
  
  [`unauthorized`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.unauthorized) Configuration
  
  `login_view`
  
  The name of the view to redirect to when the user needs to log in. (This can be an absolute URL as well, if your authentication machinery is external to your application.)
  
  `login_message`
  
  The message to flash when a user is redirected to the login page.
  
  `unauthorized_handler`(*callback*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#LoginManager.unauthorized_handler)
  
  This will set the callback for the [`unauthorized`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.unauthorized) method, which among other things is used by [`login_required`](http://www.pythondoc.com/flask-login/#flask.ext.login.login_required). It takes no arguments, and should return a response to be sent to the user instead of their normal view.Parameters:**callback** ([*callable*](http://docs.python.org/library/functions.html#callable)) – The callback for unauthorized users.[`needs_refresh`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.needs_refresh) Configuration`refresh_view`The name of the view to redirect to when the user needs to reauthenticate.`needs_refresh_message`The message to flash when a user is redirected to the reauthentication page.
  
  `needs_refresh_handler`(*callback*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#LoginManager.needs_refresh_handler)
  
  This will set the callback for the [`needs_refresh`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.needs_refresh) method, which among other things is used by [`fresh_login_required`](http://www.pythondoc.com/flask-login/#flask.ext.login.fresh_login_required). It takes no arguments, and should return a response to be sent to the user instead of their normal view.Parameters:**callback** ([*callable*](http://docs.python.org/library/functions.html#callable)) – The callback for unauthorized users.

### 登录机制

- `flask.ext.login.``current_user`

  A proxy for the current user.

- `flask.ext.login.``login_fresh`()[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#login_fresh)

  This returns `True` if the current login is fresh.

- `flask.ext.login.``login_user`(*user*, *remember=False*, *force=False*, *fresh=True*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#login_user)

  Logs a user in. You should pass the actual user object to this. If the user’s `is_active` property is `False`, they will not be logged in unless `force` is `True`.This will return `True` if the log in attempt succeeds, and `False` if it fails (i.e. because the user is inactive).

  Parameters:**user** ([*object*](http://docs.python.org/library/functions.html#object)) – The user object to log in.**remember** ([*bool*](http://docs.python.org/library/functions.html#bool)) – Whether to remember the user after their session expires. Defaults to `False`.**force** ([*bool*](http://docs.python.org/library/functions.html#bool)) – If the user is inactive, setting this to `True` will log them in regardless. Defaults to `False`.**fresh** – setting this to `False` will log in the user with a sessionmarked as not “fresh”. Defaults to `True`. :type fresh: bool

- `flask.ext.login.``logout_user`()[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#logout_user)

  Logs a user out. (You do not need to pass the actual user.) This will also clean up the remember me cookie if it exists.

- `flask.ext.login.``confirm_login`()[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#confirm_login)

  This sets the current session as fresh. Sessions become stale when they are reloaded from a cookie.

### 保护视图

- `flask.ext.login.`**login_required**(*func*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#login_required)

  If you decorate a view with this, it will ensure that the current user is logged in and authenticated before calling the actual view. (If they are not, it calls the [`LoginManager.unauthorized`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.unauthorized) callback.) For example:`@app.route('/post') @login_required def post():     pass `If there are only certain times you need to require that your user is logged in, you can do so with:`if not current_user.is_authenticated:     return current_app.login_manager.unauthorized() `...which is essentially the code that this function adds to your views.It can be convenient to globally turn off authentication when unit testing. To enable this, if the application configuration variable `LOGIN_DISABLED` is set to [`True`](http://docs.python.org/library/constants.html#True), this decorator will be ignored.Parameters:**func** (*function*) – The view function to decorate.

- `flask.ext.login.`**fresh_login_required**(*func*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#fresh_login_required)

  If you decorate a view with this, it will ensure that the current user’s login is fresh - i.e. there session was not restored from a ‘remember me’ cookie. Sensitive operations, like changing a password or e-mail, should be protected with this, to impede the efforts of cookie thieves.If the user is not authenticated, [`LoginManager.unauthorized()`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.unauthorized) is called as normal. If they are authenticated, but their session is not fresh, it will call [`LoginManager.needs_refresh()`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.needs_refresh) instead. (In that case, you will need to provide a [`LoginManager.refresh_view`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager.refresh_view).)Behaves identically to the [`login_required()`](http://www.pythondoc.com/flask-login/#flask.ext.login.login_required) decorator with respect to configutation variables.Parameters:**func** (*function*) – The view function to decorate.

### 用户对象助手

- *class* `flask.ext.login.`**UserMixin**[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#UserMixin)

  This provides default implementations for the methods that Flask-Login expects user objects to have.

### 工具

- `flask.ext.login.`**login_url**(*login_view*, *next_url=None*, *next_field='next'*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#login_url)

  Creates a URL for redirecting to a login page. If only `login_view` is provided, this will just return the URL for it. If `next_url` is provided, however, this will append a `next=URL` parameter to the query string so that the login view can redirect back to that URL.

  Parameters:**login_view** ([*str*](http://docs.python.org/library/functions.html#str)) – The name of the login view. (Alternately, the actual URL to the login view.)**next_url** ([*str*](http://docs.python.org/library/functions.html#str)) – The URL to give the login view for redirection.**next_field** ([*str*](http://docs.python.org/library/functions.html#str)) – What field to store the next URL in. (It defaults to `next`.)

- `flask.ext.login.`**make_secure_token**(**args*, ***options*)[[source\]](http://www.pythondoc.com/flask-login/_modules/flask_login.html#make_secure_token)

  This will create a secure token that you can use as an authentication token for your users. It uses heavy-duty HMAC encryption to prevent people from guessing the information. (To make it even more effective, if you will never need to regenerate the token, you can pass some random data as one of the arguments.)
  
  Parameters:
  
  ***args** – The data to include in the token.
  
  ***\*options** (*kwargs*) – To manually specify a secret key, pass `key=THE_KEY`. Otherwise, the `current_app` secret key will be used.

### 信号

如何在你的代码中使用这些信号请参阅 [Flask documentation on signals](http://flask.pocoo.org/docs/signals/)。

- `flask.ext.login.``user_logged_in`

  当一个用户登入的时候发出。除应用（信号的发送者）之外，它还传递正登入的用户 [`user`](http://docs.python.org/library/user.html#module-user) 。

- `flask.ext.login.``user_logged_out`

  当一个用户登出的时候发出。除应用（信号的发送者）之外，它还传递正登出的用户 [`user`](http://docs.python.org/library/user.html#module-user) 。

- `flask.ext.login.``user_login_confirmed`

  当用户的登入被证实，把它标记为活跃的。（它不用于常规登入的调用。） 它不接受应用以外的任何其它参数。

- `flask.ext.login.``user_unauthorized`

  当 [`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager) 上的 `unauthorized` 方法被调用时发出。它不接受应用以外的任何其它参数。

- `flask.ext.login.``user_needs_refresh`

  当 [`LoginManager`](http://www.pythondoc.com/flask-login/#flask.ext.login.LoginManager) 上的 `needs_refresh` 方法被调用时发出。它不接受应用以外的任何其它参数。

- `flask.ext.login.``session_protected`

  当会话保护起作用时，且会话被标记为非活跃或删除时发出。它不接受应用以外的任何其它参数。
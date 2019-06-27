# 使用VueJS，Flask和RethinkDB实现简单的文件存储服务

## Introduction

In this guide, I will be showing you how to build a simple file storage service. We shall be making use of VueJS to handle the front-end interactions, Flask for the back-end, and RethinkDB for database storage. I will be introducing a number of tools as we go, so stay tuned.

In the first part of this series of guides, I will be focusing on building out the back-end for the application. Later, I'll cover implementing some of the principles taught here in your current workflow either as a Python developer, Flask developer, or even as a programmer in general.

## Building the API

Let's start by building out our API. Using this file storage service, as a user, we should be able to:

1. Create an account
2. Login
3. Create and manage folders and subfolders
4. Upload a file into a folder
5. View file properties
6. Edit and Delete files

For the API, we should have the following endpoints:

- `POST /api/v1/auth/login` - This endpoint will be used to login users
- `POST /api/v1/auth/register` - This endpoint will be used to register users
- `GET /api/v1/user/<user_id>/files/` - This endpoint will be used to list files for a user with user id `user_id`
- `POST /api/v1/user/<user_id>/files/` - This endpoint will be used for creating new files for the user with user id `user_id`
- `GET /api/v1/user/<user_id>/files/<file_id>` - This endpoint will be used to get a single file with id `file_id`
- `PUT /api/v1/user/<user_id>/files/<file_id>` - This endpoint will be used to edit a single file with id `file_id`
- `DELETE /api/v1/user/<user_id>/files/<file_id>` - This endpoint will be used to delete a single file with id `file_id`

OK, we've figured out the API endpoints that we need to create this app. Now, we need to start the actual process of creating the endpoints. So let's get to it.

## Setup

You should start by creating a project directory. This is the recommended structure for our application:

```bash
-- /api
  -- /controllers
  -- /utils
  -- models.py
  -- __init__.py
-- /templates
-- /static
  -- /lib
  -- /js
  -- /css
  -- /img
-- index.html
-- config.py
-- run.py
```

The modules and packages for the API will be put into the `/api`directory with the models stored in the `models.py` module, and the controllers (mostly for routing purposes) stored as modules in the `/controllers` package.

We will add our routes and app creation function to `/api/__init__.py`. This way, we are able to use the `create_app()`function to create multiple app instances with different configurations. This is especially helpful when you are writing tests for your application.

```python
from flask import Flask, Blueprint
from flask_restful import Api

from config import config

def create_app(env):
  app = Flask(__name__)
  app.config.from_object(config[env])

  api_bp = Blueprint('api', __name__)
  api = Api(api_bp)

  # Code for adding Flask RESTful resources goes here

  app.register_blueprint(api_bp, url_prefix="/api/v1")

    return app
```

From what you can see here, we have created a function `create_app()` which takes an `env` parameter. Ideally, `env` should be one of `development`, `production`, and `testing`. Using the value supplied for `env`, we will be loading specific configurations. This configuration information is stored in `config.py` as a dictionary of classes.

```python
class Config(object):
  DEBUG = True
  TESTING = False
  DATABASE_NAME = "papers"

class DevelopmentConfig(Config):
  SECRET_KEY = "S0m3S3cr3tK3y"

config = {
  'development': DevelopmentConfig,
  'testing': DevelopmentConfig,
  'production': DevelopmentConfig
}
```

For now we've specified only a few configuration parameters like the `DEBUG` parameter which tells Flask whether or not to run in debug mode. We have also specified the `DATABASE_NAME` parameter, which we shall be referencing in our models and a `SECRET_KEY` parameter for JWT Token generation. We have used the same settings for all the environments so far.

As you can see, we are using Flask Blueprint to version our API, just in case we need to change implementation of core functionality without affecting the user. We have also initialized `api`, a Flask-RESTful API object. Later on, I will show you how to add new API endpoints to our app using this object.

Next, we put the code for running the server into `run.py` in the root directory. We will be using Flask-Script to create extra CLI commands for our application.

```python
from flask_script import Manager
from api import create_app

app = create_app('development')
manager = Manager(app)

@manager.command
def migrate():
  # Migration script
  pass

if __name__ == '__main__':
  manager.run()
```

Here, we have used the `flask_script.Manager` class to abstract out the running of the server and enable us to add new commands for the CLI. `migrate()` will be used to automatically create all the tables required by our models. This is a simplified solution for now. If all is well, you should not have any errors.

Now, we can go to the command line and run `python run.py runserver` to run the server on the default port 5000.

Continue on to the next guide in this series learn about [User Model and Authentication Controller](https://www.pluralsight.com/guides/user-model-authentication-controller-simple-file-storage-using-vuejs-flask-and-rethinkdb).

## 用户模型 User Model

我们需要两个模型来启动应用。在这里，我们先创建用户模型。

我们首先连接到 RethinkDB，并创建一个数据库的连接对象。

```python
import rethinkdb as r
from flask import current_app

conn = r.connect(db="papers")
class RethinkDBModel(object):
  pass
```

We referenced the database name from the application config. In flask, we have the `current_app` variable which holds a reference to the currently running application instance.

Why did I create a blank `RethinkDBModel` class? Well, there might be a couple of things we might want to share across model classes; this class is here in case cross-model sharing is necessary.

Our `User` class will inherit from this empty base class. In `User`, we will have a few functions which we'll use to interact with the database from the controllers.

We start with the `create()` function. This function will be called when we need to create a user document in the table.

```python
class User(RethinkDBModel):
  _table = 'users'

  @classmethod
  def create(cls, **kwargs):
      fullname = kwargs.get('fullname')
      email = kwargs.get('email')
      password = kwargs.get('password')
      password_conf = kwargs.get('password_conf')
      if password != password_conf:
        raise ValidationError("Password and Confirm password need to be the same value")
      password = cls.hash_password(password)
      doc = {
        'fullname': fullname,
        'email': email,
        'password': password,
        'date_created': datetime.now(r.make_timezone('+01:00')),
        'date_modified': datetime.now(r.make_timezone('+01:00'))
        }
      r.table(cls._table).insert(doc).run(conn)
```

Here, we are making use of the `classmethod` decorator. This decorator enables us have access to the class instance from within the method body. We will be using the class instance to access the `_table` property from within the method. The `_table` stores the table name for that model.

We have also added code here to make sure that the `password` and `password_conf` fields are the same. When this happens, a `ValidationError` will be thrown. The exceptions will be stored in the `/api/utils/errors.py` module. Find the definition of `ValidationError` below:

```python
class ValidationError(Exception):
    pass
```

We're using named exceptions because they are easier to track.

Notice how we used `datetime.now(r.make_timezone('+01:00'))`here? I faced some issues when I used `datetime.now()` without the timezone. **RethinkDB requires that time zone information be set on date fields in documents.** The Python function does not supply this for us by default unless we specify this as a parameter to the `now()`function (See [here](https://www.rethinkdb.com/docs/dates-and-times/python/) for more). Using the `r.make_timezone('+01:00')`we are able to create a timezone object that we can use for the `datetime.now()` function.

If all goes well and no exceptions are encountered, we call the `insert()` method on the table object that `r.table(table_name)`returns. This method takes a dictionary containing the data. This data will be stored as a new document in the table selected.

We have made a call to the `hash_password()` method in our class. This method makes use of the `hash.pbkdf2_sha256` module in the `passlib` package to hash the password fairly securely. In addition to that, we will need to create a method for verifying passwords.

```python
from passlib.hash import pbkdf2_sha256

class User(RethinkDBModel):
  _table = 'users'

  @classmethod
  def create(cls, **kwargs):
     fullname = kwargs.get('fullname')
     email = kwargs.get('email')
     password = kwargs.get('password')
     password_conf = kwargs.get('password_conf')
     if password != password_conf:
         raise ValidationError("Password and Confirm password need to be the same value")
     password = cls.hash_password(password)
     doc = {
         'fullname': fullname,
         'email': email,
         'password': password,
         'date_created': datetime.now(r.make_timezone('+01:00')),
         'date_modified': datetime.now(r.make_timezone('+01:00'))
     }
     r.table(cls._table).insert(doc).run(conn)

  @staticmethod
  def hash_password(password):
      return pbkdf2_sha256.encrypt(password, rounds=200000, salt_size=16)

  @staticmethod
  def verify_password(password, _hash):
      return pbkdf2_sha256.verify(password, _hash)
```

The `pbkdf2_sha256.encrypt()` method is called with the password and values for `rounds` and `salt_size`. See [here](https://pythonhosted.org/passlib/lib/passlib.hash.pbkdf2_digest.html) for details on how you can customize your encryption and how the library works. Just to give some context on the decision to use `PBKDF2`:

> Security-wise, PBKDF2 is currently one of the leading key derivation functions, and has no known security issues.

-- *Quotes from the passlib documentation*

The `verify_password` method will be called using the password string and a hash. It will return true or false if the password is valid.

We will now move on the the `validate()` function. This function will be called in the login method with the email address and password. The function will check that the document exists using the `email`field as index and then compare the password hash against the password supplied.

In addition to that, since we're going to be making use of JWT (JSON Web Token) for token-based authentication, we will be generating a token if the user supplies valid information. This is how the entire `models.py` will look like when we're done adding in the logic.

```python
import os
import rethinkdb as r
from jose import jwt
from datetime import datetime
from passlib.hash import pbkdf2_sha256

from flask import current_app

from api.utils.errors import ValidationError

conn = r.connect(db="papers")

class RethinkDBModel(object):
    pass


class User(RethinkDBModel):
  _table = 'users'

  @classmethod
  def create(cls, **kwargs):
    fullname = kwargs.get('fullname')
    email = kwargs.get('email')
    password = kwargs.get('password')
    password_conf = kwargs.get('password_conf')
    if password != password_conf:
        raise ValidationError("Password and Confirm password need to be the same value")
    password = cls.hash_password(password)
    doc = {
        'fullname': fullname,
        'email': email,
        'password': password,
        'date_created': datetime.now(r.make_timezone('+01:00')),
        'date_modified': datetime.now(r.make_timezone('+01:00'))
    }
    r.table(cls._table).insert(doc).run(conn)

  @classmethod
  def validate(cls, email, password):
    docs = list(r.table(cls._table).filter({'email': email}).run(conn))

    if not len(docs):
        raise ValidationError("Could not find the e-mail address you specified")

    _hash = docs[0]['password']

    if cls.verify_password(password, _hash):
        try:
            token = jwt.encode({'id': docs[0]['id']}, current_app.config['SECRET_KEY'], algorithm='HS256')
            return token
        except JWTError:
            raise ValidationError("There was a problem while trying to create a JWT token.")
        else:
            raise ValidationError("The password you inputted was incorrect.")

  @staticmethod
  def hash_password(password):
      return pbkdf2_sha256.encrypt(password, rounds=200000, salt_size=16)

  @staticmethod
  def verify_password(password, _hash):
      return pbkdf2_sha256.verify(password, _hash)
```

A couple of things to take note of in the `validate()` method. Firstly, the `filter()` function was used here on the table object. This command takes in a dictionary that is used to search through the table. This function can also take a predicate, in some cases. This predicate can be a lambda function and will be used similarly to what is done with the python `filter` function or any other function that takes a function as an argument. The function returns a cursor that can be used to access all the documents that are returned by the query. The cursor is iterable and as such we can iterate the cursor object using the `for...in` loop. In this case, we have chosen to convert the iterable object to a list using the Python list function.

As you would expect, we basically do two things here. We want to know if the email address exists at all and then if the password is correct. For the first part, we basically count the collection. If this is empty, we raise an error. For the second part, we call the `verify_password()` function to compare the password supplied with the hash in the database. We raise an exception if these don't match.

Also noteworthy is how we have used `jwt.encode()` to create a JWT token and return it to the controller. This method is fairly straightforward and you can see the documentation [here](https://github.com/mpdavis/python-jose).

That does it for the model. Let's move on to the controllers. We have tried to obey the principle of having **Fat models and Slim controllers**, in this model. Most of the logic is in the models. This way, our controllers only focus on routing and error reporting to the API end user.

## Authentication Controller

For the authentication controller, we need to add in Flask RESTful resource sub classes. Django web development is similar to class-based views. It's simply created as a subclass of the `flask_restful.Resource` class. Your subclasses will have methods that map to respective HTTP verbs. For instance, if we wanted to implement a GET action, we will be creating a `get()` method in our Resource subclass. The process is completed by mapping URLs to the respective classes using the `api.add_resource()` method.

Now let's add in two classes; one to take care of POST action to the login route and one to take care of POST action to the register route.

We'll start by creating the required classes. These should be stored in the `/api/controllers/auth.py` file.

```python
from flask_restful import Resource

class AuthLogin(Resource):
  def post(self):
    pass

class AuthRegister(Resource):
  def post(self):
    pass
```

```python
from flask import Flask, Blueprint
from flask_restful import Api

from api.controllers import auth
from config import config

def create_app(env):
  app = Flask(__name__)
  app.config.from_object(config[env])

  api_bp = Blueprint('api', __name__)
  api = Api(api_bp)

  api.add_resource(auth.AuthLogin, '/auth/login')
  api.add_resource(auth.AuthRegister, '/auth/register')

  app.register_blueprint(api_bp, url_prefix="/api/v1")

  return app
```

Now let's head back to the controller file to add in some logic. The logic required here is similar to what you will do with authentication systems in general.

```python
from flask_restful import reqparse, abort, Resource

from api.models import User
from api.utils.errors import ValidationError

class AuthLogin(Resource):
  def post(self):
     parser = reqparse.RequestParser()
     parser.add_argument('email', type=str, help='You need to enter your e-mail address', required=True)
     parser.add_argument('password', type=str, help='You need to enter your password', required=True)

     args = parser.parse_args()

     email = args.get('email')
     password = args.get('password')

     try:
        token = User.validate(email, password)
         return {'token': token}
     except ValidationError as e:
         abort(400, message='There was an error while trying to log you in -> {}'.format(e.message))

class AuthRegister(Resource):
  def post(self):
     parser = reqparse.RequestParser()
     parser.add_argument('fullname', type=str, help='You need to enter your full name', required=True)
     parser.add_argument('email', type=str, help='You need to enter your e-mail address', required=True)
     parser.add_argument('password', type=str, help='You need to enter your chosen password', required=True)
     parser.add_argument('password_conf', type=str, help='You need to enter the confirm password field', required=True)

     args = parser.parse_args()

     email = args.get('email')
     password = args.get('password')
     password_conf = args.get('password_conf')
     fullname = args.get('fullname')

     try:
         User.create(
             email=email,
             password=password,
             password_conf=password_conf,
             fullname=fullname
         )
         return {'message': 'Successfully created your account.'}
     except ValidationError as e:
         abort(400, message='There was an error while trying to create your account -> {}'.format(e.message))
```

As mentioned earlier, the majority of the logic and database interaction has been pushed to the model. The controller logic is relatively simple.

To summarize what was done, for the login controller `AuthLogin`, we created a `post()` function which accepts the e-mail address and password, validates the fields using `reqparse`, and calls `User.validate()` which validates the information sent and returns a token. If an error occurs, we catch it and respond with an error message.

Similarly, for the `AuthRegister`, we collect information from the user and call a model `create()` function. In this case, we create a collection for the email address, password, password confirm, and full name fields. We pass all these values to the `User.create()` function and, as before, this function will throw an error if anything goes wrong.

All things being equal, everything should work just fine. Run the server using `python run.py runserver` to test it out. You should be able to access the two endpoints that we've created here, and it should work very well.

Next up, we'll be creating the models for our files. Continue on to the next guide in this series - [File and Folder Models](https://www.pluralsight.com/guides/file-and-folder-models-simple-file-storage-using-vuejs-flask-and-rethinkdb).

## File and Folder Models

We'll be creating simple models for working with the files and folders similar to what we did with the User model. We'll create a `Folder`model as a child of the `File` model.

For more information about how to set up your workspace, checkout the first guide in this series: [Introduction and Setup to Building a Simple File Storage Service Using VueJS, Flask and RethinkDB](https://www.pluralsight.com/guides/introduction-and-setup-building-simple-file-storage-using-vuejs-flask-and-rethinkdb).

## File Model

What you will notice as we proceed is that the fact files are stored in a flat manner in the filesystem. All users have a folder where all their files are stored but the structure of the data is logical and stored in the database. This way, we have minimal writes on the file system. To do this we will be employing some pretty neat techniques that will probably be useful to you for future projects.

Create the base models in `/api/models.py`

```python
class File(RethinkDBModel):
  _table = 'files'

class Folder(File):
  pass
```

We start out by creating the `create()` method for the File model. This method will be called when we make a POST request to the `/users/<user_id>/files/<file_id>` endpoint used to create a file.

```python
@classmethod
def create(cls, **kwargs):
  name = kwargs.get('name')
  size = kwargs.get('size')
  uri = kwargs.get('uri')
  parent = kwargs.get('parent')
  creator = kwargs.get('creator')

  # Direct parent ID
  parent_id = '0' if parent is None else parent['id']

  doc = {
     'name': name,
     'size': size,
     'uri': uri,
     'parent_id': parent_id,
     'creator': creator,
     'is_folder': False,
     'status': True,
     'date_created': datetime.now(r.make_timezone('+01:00')),
     'date_modified': datetime.now(r.make_timezone('+01:00'))
  }

  res = r.table(cls._table).insert(doc).run(conn)
  doc['id'] = res['generated_keys'][0]

  if parent is not None:
     Folder.add_object(parent, doc['id'])

  return doc
```

First, we collect all the information we need from the keyword arguments dictionary like name, size, file URI, creator, and so on. We have collected a parameter which we are calling `parent`. This field points to the `id` of the folder in which we want to store this file. We can choose not to pass this parameter if we want to store the file in the root folder. As we go on, you will understand how we can make use of this to create complex nested folder structures.

Notice here how having a `parent` of `None` makes the `parent_id`field `0`. This takes care of cases when a file is created with no parent. This assumes that we are storing the file in the root folder which will have an ID of 0.

We collect all of this information about the file into a dictionary and call the `insert()` function to store it in the database. The returned dictionary from calling the insert function contains the IDs of the newly generated documents. After inserting the dictionary, we populate the ID information in the dictionary so that we can return it to the users.

In the last three lines of this method, we've added in a check to see if the `parent` is `None`. Since this file manager implementation has folders, whenever we create a file in a folder, we would have to logically add each newly created object to a folder. We do this by adding the object ID into an `objects` list in the corresponding record for the folder we're trying to store it in. This is done by calling a method which we will create in the Folder model called `add_object`.

Next up, we will go back to our base `RethinkDBModel` class to create a number of useful methods which we may or may not override in the child classes.

```python
class RethinkDBModel(object):
  @classmethod
  def find(cls, id):
     return r.table(cls._table).get(id).run(conn)

  @classmethod
  def filter(cls, predicate):
     return list(r.table(cls._table).filter(predicate).run(conn))

  @classmethod
  def update(cls, id, fields):
     status = r.table(cls._table).get(id).update(fields).run(conn)
     if status['errors']:
         raise DatabaseProcessError("Could not complete the update action")
     return True

  @classmethod
  def delete(cls, id):
     status = r.table(cls._table).get(id).delete().run(conn)
     if status['errors']:
         raise DatabaseProcessError("Could not complete the delete action")
     return True
```

Here we created wrapper methods for the RethinkDB `get()`, `filter()`, `update()`, and `delete()` functions. This way, subclasses can leverage on those functions for more complex interactions.

The next method we will be creating in our File model is a function that will be used to move files between folders.

```python
@classmethod
def move(cls, obj, to):
  previous_folder_id = obj['parent_id']
  previous_folder = Folder.find(previous_folder_id)
  Folder.remove_object(previous_folder, obj['id'])
  Folder.add_object(to, obj['id'])
```

The logic here is fairly simple. We call this method when we want to move a file `obj` into folder `to`.

We start by getting the current folder ID for the current parent directory of the file. This is stored in the `parent_id` field of `obj`. We call the Folder model `find` function to obtain the folder object as a dictionary called `previous_folder`. After getting this object, we do two things. We remove the file object from the previous folder `previous_folder`, and add the file object to the new folder `to`. We achieve this by calling the `remove_object()` and `add_object()`methods of the Folder class. These methods remove the file ID from and add the file ID to the `objects` list in the Folder document, respectively. I will be showing what the implementations for these looks like in a bit.

我们现在完成了对文件建模。我们可以对文件进行基本交互，例如创建，编辑，从数据库中删除等等。

接下来，我们继续讨论`Folder`模型的逻辑，这与我们为文件所做的非常相似。

## Folder Model

```python
@classmethod
def create(cls, **kwargs):
  name = kwargs.get('name')
  parent = kwargs.get('parent')
  creator = kwargs.get('creator')

  # Direct parent ID
  parent_id = '0' if parent is None else parent['id']

  doc = {
     'name': name,
     'parent_id': parent_id,
     'creator': creator,
     'is_folder': True,
     'last_index': 0,
     'status': True,
     'objects': None,
     'date_created': datetime.now(r.make_timezone('+01:00')),
     'date_modified': datetime.now(r.make_timezone('+01:00'))
  }

  res = r.table(cls._table).insert(doc).run(conn)
  doc['id'] = res['generated_keys'][0]

  if parent is not None:
     cls.add_object(parent, doc['id'], True)

  cls.tag_folder(parent, doc['id'])

  return doc

@classmethod
def tag_folder(cls, parent, id):
  tag = id if parent is None else '{}#{}'.format(parent['tag'], parent['last_index'])
  cls.update(id, {'tag': tag})
```

`create()`方法与File 类的类似。只需要文件夹的名称和创建者作为参数来创建文件夹对象。使用 `add_object()` 方法将文件夹添加到父文件夹中。

这里包含了 `is_folder` 字段，默认`True`为文件夹，`False` 为文件。

我们还创建了 `tag_folder()`方法，稍后用来处理移动文件夹。索引基于它们在树上的级别。存储在根级别的任何文件夹都有一个标记，`<id>`其中id是文件夹的ID。存储在该文件夹下的任何文件夹都将具有id，`<id>-n`其中n是一个连续递增的整数，随后嵌套文件夹将遵循相同的模式。

我们通过调用`insert()`函数插入我们创建的字典，以将所有这些数据存储到数据库中。文件夹根据它们在文件树上的位置进行标记。

接下来，我们将重写File类的find方法，以包含显示文件夹列表信息的功能。这对我们以后的前端非常有用。

```python
@classmethod
def find(cls, id, listing=False):
  file_ref = r.table(cls._table).get(id).run(conn)
  if file_ref is not None:
     if file_ref['is_folder'] and listing and file_ref['objects'] is not None:
         file_ref['objects'] = list(r.table(cls._table).get_all(r.args(file_ref['objects'])).run(conn))
  return file_ref
```

我们首先使用其ID获取对象。我们在满足三个条件时显示文件夹的列表：

- `listing`设置为True。我们使用此变量来了解我们是否确实需要有关文件夹中包含的文件的信息。
- `file_ref`对象实际上是一个文件夹。我们通过检查`is_folder`文档的字段来确定这一点。
- 文件夹文档列表中有`objects`对象。

如果满足所有这些条件，我们通过调用`get_all`方法获取所有嵌套对象。此方法接受多个关键字并返回所有具有相应关键字的对象。我们使用该`r.args`方法将对象列表转换为`get_all`方法的多个参数，用返回的列表替换文档的`objects`字段。此列表包含每个嵌套文件/文件夹的详细信息。

接下来，我们继续创建文件夹对象的`move`方法。这与之前为文件对象创建的`move`方法类似，其中包含用于处理标记的逻辑并确保我们可以正确地移动文件夹。

```python
@classmethod
def move(cls, obj, to):
  if to is not None:
     parent_tag = to['tag']
     child_tag = obj['tag']

     parent_sections = parent_tag.split("#")
     child_sections = child_tag.split("#")

     if len(parent_sections) > len(child_sections):
         matches = re.match(child_tag, parent_tag)
         if matches is not None:
             raise Exception("You can't move this object to the specified folder")

  previous_folder_id = obj['parent_id']
  previous_folder = cls.find(previous_folder_id)
  cls.remove_object(previous_folder, obj['id'])

  if to is not None:
     cls.add_object(to, obj['id'], True)
```

在这里，我们首先确保我们移动到的文件夹必须是指定的而不是`None`。如果没有指定，将会使此文件夹移动到根文件夹。

我们有了待移动文件夹和目的文件夹的标签，然后比较标签的组成，从而知道文件在文件树中的层级. 只有一种情况需要我们特别处理：当父节点比子节点层级还要低的情景。我们可以将文件夹移动到其级别及以上的任何文件夹，但如果`parent_sections`它超过了`child_sections`，就表示我们正在尝试将一个文件夹移动到自己的子文件夹中，而这种情况使不允许的。我们需要小心避免出现这种操作。

如果要移动的目的文件夹位于将移动文件夹的下面，我们必须确保前一个文件夹不嵌套在后者中。这可以通过检查目的文件夹的`child_tag` 不以 `parent_tag`字符串开始来实现。我们使用正则表达式来实现它，并在发生这种情况时引发异常。

我们差不多完成了！最后，我们将创建`add_object()`和`remove_object()`方法。

```python
@classmethod
def remove_object(cls, folder, object_id):
  update_fields = folder['objects'] or []
  while object_id in update_fields:
     update_fields.remove(object_id)
  cls.update(folder['id'], {'objects': update_fields})

@classmethod
def add_object(cls, folder, object_id, is_folder=False):
  p = {}
  update_fields = folder['objects'] or []
  update_fields.append(object_id)
  if is_folder:
     p['last_index'] = folder['last_index'] + 1
  p['objects'] = update_fields
  cls.update(folder['id'], p)
```

如前所述，我们将通过修改文件夹对象中的 `objects` 列表来执行添加和删除操作。

我们现在已经完成了这些模型。接下来，控制器！继续阅读本系列的下一个指南：[文件控制器](https://www.pluralsight.com/guides/file-controller-and-finishing-simple-file-storage-using-vuejs-flask-and-rethinkdb)。

## 文件控制器 File Controller

文件控制器用于处理文件和文件夹，因此，这里的逻辑将比文件夹控制器略多一些。我们首先为`/api/controllers/files.py`模块中的控制器创建一个样板。

```python
import os

from flask import request, g
from flask_restful import reqparse, abort, Resource
from werkzeug import secure_filename

from api.models import File

BASE_DIR = os.path.abspath(
  os.path.dirname(os.path.dirname(os.path.dirname(__file__)))
)


class CreateList(Resource):
  def get(self, user_id):
     pass

  def post(self, user_id):
     pass

class ViewEditDelete(Resource):
  def get(self, user_id, file_id):
     pass

  def put(self, user_id, file_id):
     pass

  def delete(self, user_id, file_id):
     pass
```

`CreateList`类，顾名思义，将用于管理当前用户的文件列表。`ViewEditDelete` 类用于浏览，编辑和删除文件。我们在类中使用的方法与相应的HTTP action相对应。

### 装饰 Decorators

我们将编写一组在Resource类中使用的装饰器。这部分代码将独立出来保存在 `/api/utils/decorators.py` 文件中。

```python
from jose import jwt
from jose.exceptions import JWTError
from functools import wraps

from flask import current_app, request, g
from flask_restful import abort

from api.models import User, File

def login_required(f):
  '''
  This decorator checks the header to ensure a valid token is set
  '''
  @wraps(f)
  def func(*args, **kwargs):
     try:
         if 'authorization' not in request.headers:
                abort(404, message="You need to be logged in to access this resource")
         token = request.headers.get('authorization')
         payload = jwt.decode(token, current_app.config['SECRET_KEY'], algorithms=['HS256'])
         user_id = payload['id']
         g.user = User.find(user_id)
         if g.user is None:
            abort(404, message="The user id is invalid")
         return f(*args, **kwargs)
     except JWTError as e:
         abort(400, message="There was a problem while trying to parse your token -> {}".format(e.message))
  return func

def validate_user(f):
  '''
  This decorate ensures that the user logged in is the actually the same user we're operating on
  '''
  @wraps(f)
  def func(*args, **kwargs):
     user_id = kwargs.get('user_id')
     if user_id != g.user['id']:
         abort(404, message="You do not have permission to the resource you are trying to access")
     return f(*args, **kwargs)
  return func

def belongs_to_user(f):
  '''
  This decorator ensures that the file we're trying to access actually belongs to us
  '''
  @wraps(f)
  def func(*args, **kwargs):
     file_id = kwargs.get('file_id')
     user_id = kwargs.get('user_id')
     file = File.find(file_id, True)
     if not file or file['creator'] != user_id:
        abort(404, message="The file you are trying to access was not found")
     g.file = file
     return f(*args, **kwargs)
  return func
```

`login_required`装饰器用来验证用户是否已登录。我们通过解码令牌来保护某些端点以确保其有效性。我们获取存储在令牌中的`id`字段并检索相应的用户对象。然后，将此对象存储在`g.user`中。

类似地，我们创建`validate_user`装饰器，确保没有其他登录用户可以访问标记有其他用户ID的URL模式。此验证完全基于URL中的信息。

Finally, the `belongs_to_user` decorator ensures that only the user who created a file can access it. This decorator actually checks the `creator` field in the file document against the `user_id` supplied.

最后，`belongs_to_user`装饰器确保只有创建文件的用户才能访问它。这个装饰器实际上是`creator`根据`user_id`提供的文件检查文件中的字段。

以下是创建文件和文件列表的视图：

```python
class CreateList(Resource):
  @login_required
  @validate_user
  @marshal_with(file_array_serializer)
  def get(self, user_id):
     try:
         return File.filter({'creator': user_id, 'parent_id': '0'})
     except Exception as e:
         abort(500, message="There was an error while trying to get your files --> {}".format(e.message))

  @login_required
  @validate_user
  @marshal_with(file_serializer)
  def post(self, user_id):
     try:
        parser = reqparse.RequestParser()
         parser.add_argument('name', type=str, help="This should be the folder name if creating a folder")
         parser.add_argument('parent_id', type=str, help='This should be the parent folder id')
         parser.add_argument('is_folder', type=bool, help="This indicates whether you are trying to create a folder or not")

         args = parser.parse_args()

         name = args.get('name', None)
         parent_id = args.get('parent_id', None)
         is_folder =  args.get('is_folder', False)

         parent = None

         # Are we adding this to a parent folder?
         if parent_id is not None:
             parent = File.find(parent_id)
             if parent is None:
                 raise Exception("This folder does not exist")
             if not parent['is_folder']:
                 raise Exception("Select a valid folder to upload to")

         # Are we creating a folder?
         if is_folder:
             if name is None:
                raise Exception("You need to specify a name for this folder")

             return Folder.create(
                 name=name,
                 parent=parent,
                 is_folder=is_folder,
                 creator=user_id
             )
         else:
             files = request.files['file']

             if files and is_allowed(files.filename):
                 _dir = os.path.join(BASE_DIR, 'upload/{}/'.format(user_id))

                 if not os.path.isdir(_dir):
                     os.mkdir(_dir)
                 filename = secure_filename(files.filename)
                 to_path = os.path.join(_dir, filename)
                 files.save(to_path)
                 fileuri = os.path.join('upload/{}/'.format(user_id), filename)
                 filesize = os.path.getsize(to_path)

                 return File.create(
                     name=filename,
                     uri=fileuri,
                     size=filesize,
                     parent=parent,
                     creator=user_id
                 )
             raise Exception("You did not supply a valid file in your request")
     except Exception as e:
         abort(500, message="There was an error while processing your request --> {}".format(e.message))
```

The listing method is pretty straightforward. We filter the table for all files that were created by a certain user and stored in the root directory. We return this data for this endpoint and throw an exception if there are any errors.

列表方法非常简单。我们过滤表格，查找由某个用户创建并存储在根目录中的所有文件。我们为此端点返回此数据，并在出现任何错误时抛出异常。

### 创建 Create

For the create action, it's a bit more involved. For this guide, we're assuming that files and folders will be created with the same endpoint. For files, we will need to supply the file as well as a `parent_id`, if we're uploading it in a folder. For folders, we will need a name and a `parent_id` value, again if we are creating this within another folder. For folders, we also need to send an `is_folder` field with our request to specify that we are creating a folder.

对于创建动作，它涉及更多。对于本指南，我们假设将使用相同的端点创建文件和文件夹。对于文件，我们将需要提供文件以及a `parent_id`，如果我们将其上传到文件夹中。对于文件夹，`parent_id`如果我们在另一个文件夹中创建它，我们将需要一个名称和一个值。对于文件夹，我们还需要发送一个`is_folder`字段，其中包含我们的请求，以指定我们正在创建文件夹。

If we are going to store this within a folder, we have to ensure that the folder exists and is a valid folder. We also ensure that we are supplying a name field if we are creating a folder.

如果我们要将其存储在一个文件夹中，我们必须确保该文件夹存在并且是一个有效的文件夹。如果我们要创建文件夹，我们还会确保提供名称字段。

For file creation, we upload the file into a folder specifically named for the different users as mentioned before. In our case, we are using the pattern `/upload/<user_id>` for the different user file directories. We have also used the file information to populate the document we're going to be storing in the table.

对于文件创建，我们将文件上传到专门为不同用户命名的文件夹，如前所述。在我们的例子中，我们将模式`/upload/<user_id>`用于不同的用户文件目录。我们还使用文件信息来填充我们将要存储在表中的文档。

We conclude by calling the methods for file and folder creation - `File.create()` and `Folder.create()` respectively.

最后，我们分别实现了创建文件和文件夹的方法- `File.create()`和`Folder.create()`。

### 串行器 Serializers

Notice that we have used the `marshal_with` decorator available with Flask-RESTful. This decorator is used to format the response object and to indicate the different field names and types that we'll be returning. See the definition of the `file_array_serializer` and `file_serializer` below:

请注意，我们使用了Flask-RESTful 提供的 `marshal_with` 装饰器。此装饰器用于格式化响应对象并指示我们将返回的不同字段名称和类型。`file_array_serializer` 及 `file_serializer` 的定义如下：

```python
file_array_serializer = {
  'id': fields.String,
  'name': fields.String,
  'size': fields.Integer,
  'uri': fields.String,
  'is_folder': fields.Boolean,
  'parent_id': fields.String,
  'creator': fields.String,
  'date_created': fields.DateTime(dt_format=  'rfc822'),
  'date_modified': fields.DateTime(dt_format='rfc822'),
}

file_serializer = {
  'id': fields.String,
  'name': fields.String,
  'size': fields.Integer,
  'uri': fields.String,
  'is_folder': fields.Boolean,
  'objects': fields.Nested(file_array_serializer, default=[]),
  'parent_id': fields.String,
  'creator': fields.String,
  'date_created': fields.DateTime(dt_format='rfc822'),
  'date_modified': fields.DateTime(dt_format='rfc822'),
}
```

这些代码可以添加在`/api/controllers/files.py`的前面或独自保存在`/api/utils/serializers.py`文件中。

两个串行器之间的区别在于文件串行器可以对象数组进行相应。`file_array_serializer` 可用在泪飙响应，`file_serializer` 用于对象序列化。

我们还使用了一个`is_allowed()` 函数来确保上传的文件类型。我们创建了一个名为`ALLOWED_EXTENSIONS`包含所有允许扩展名的列表。

```python
ALLOWED_EXTENSIONS = set(['txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'])

def is_allowed(filename):
  return '.' in filename and \
       filename.rsplit('.', 1)[1] in ALLOWED_EXTENSIONS
```

最后，我们通过`ViewEditDelete`在`/api/controllers/files.py`模块中声明资源类。

```python
class ViewEditDelete(Resource):
  @login_required
  @validate_user
  @belongs_to_user
  @marshal_with(file_serializer)
  def get(self, user_id, file_id):
     try:
         should_download = request.args.get('download', False)
         if should_download == 'true':
             parts = os.path.split(g.file['uri'])
             return send_from_directory(directory=parts[0], filename=parts[1])
         return g.file
     except Exception as e:
         abort(500, message="There was an while processing your request --> {}".format(e.message))

  @login_required
  @validate_user
  @belongs_to_user
  @marshal_with(file_serializer)
  def put(self, user_id, file_id):
     try:
         update_fields = {}
         parser = reqparse.RequestParser()

         parser.add_argument('name', type=str, help="New name for the file/folder")
         parser.add_argument('parent_id', type=str, help="New parent folder for the file/folder")

         args = parser.parse_args()

         name = args.get('name', None)
         parent_id = args.get('parent_id', None)

         if name is not None:
             update_fields['name'] = name

         if parent_id is not None and g.file['parent_id'] != parent_id:
             if parent_id != '0'
                 folder_access = Folder.filter({'id': parent_id, 'creator': user_id})
                 if not folder_access:
                     abort(404, message="You don't have access to the folder you're trying to move this object to")

             if g.file['is_folder']:
                 update_fields['tag'] = g.file['id'] if parent_id == '0' else '{}#{}'.format(folder_access['tag'], folder['last_index'])
                 Folder.move(g.file, folder_access)
             else:
                 File.move(g.file, folder_access)

             update_fields['parent_id'] = parent_id

         if g.file['is_folder']:
             Folder.update(file_id, update_fields)
         else:
             File.update(file_id, update_fields)

         return File.find(file_id)
     except Exception as e:
         abort(500, message="There was an while processing your request --> {}".format(e.message))

  @login_required
  @validate_user
  @belongs_to_user
  def delete(self, user_id, file_id):
     try:
         hard_delete = request.args.get('hard_delete', False)
         if not g.file['is_folder']:
             if hard_delete == 'true':
                 os.remove(g.file['uri'])
                 File.delete(file_id)
             else:
                 File.update(file_id, {'status': False})
         else:
             if hard_delete == 'true':
                 folders = Folder.filter(lambda folder: folder['tag'].startswith(g.file['tag']))
                 for folder in folders:
                     files = File.filter({'parent_id': folder['id'], 'is_folder': False })
                     File.delete_where({'parent_id': folder['id'], 'is_folder': False })
                     for f in files:
                         os.remove(f['uri'])
             else:
                 File.update(file_id, {'status': False})
                 File.update_where({'parent_id': file_id}, {'status': False})
         return "File has been deleted successfully", 204
     except:
         abort(500, message="There was an error while processing your request --> {}".format(e.message))
```

我们创建了`get()` 方法根据ID返回单个文件或文件夹对象。对于文件夹，它包括列表信息。如果你看`belongs_to_user`装饰器，你可以看到这是如何完成的。对于文件对象，如果我们要下载它，就需要把参数 `should_download`设置为`true`。

`put()` 方法负责更新文件和文件夹信息，也可以移动文件和文件夹。通过更新文件/文件夹的 `parent_id` 字段来移动文件。`move()` 方法也包含了这种操作的逻辑。

`delete()` 方法还有一个查询参数，它指定我们是否要执行硬删除。对于硬删除，将从数据库中删除记录，并从文件系统中删除文件。对于软删除，我们只将文件`status`字段更新为false。

我们在在`RethinkDBModel` 类中还新建了 `update_where()`，`delete_where()` 方法，用于从表中删除和更新过滤集：

```python
@classmethod
def update_where(cls, predicate, fields):
  status = r.table(cls._table).filter(predicate).update(fields).run(conn)
  if status['errors']:
     raise DatabaseProcessError("Could not complete the update action")
  return True

@classmethod
def delete_where(cls, predicate):
  status = r.table(cls._table).filter(predicate).delete().run(conn)
  if status['errors']:
     raise DatabaseProcessError("Could not complete the delete action")
  return True
```

## 结论

就是这样！我们已经完成了文件存储API。运行API以查看其运行情况。

您可以在[此处](https://github.com/andela-cnnadi/papers)查看项目的代码库。
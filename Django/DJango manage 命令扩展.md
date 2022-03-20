#Django 扩展自定义manage命令

在app目录中建立一个目录management，并且在下面建立一个目录叫commands，下面增加一个文件，叫hello.py即可。

**要注意一下几点**

1）说是目录，其实应该是包，所以在management下面和commands下面都要添加__init__.py。

2）app一定要添加在INSTALLED_APPS中，不然命令加载不到。

Django will register a `manage.py` command for each Python module in that directory whose name doesn’t begin with an underscore. 


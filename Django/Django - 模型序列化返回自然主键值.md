# Django - 模型序列化返回自然主键值

## 场景

在设计表结构时，难免需要建立一些外键关联。例如这样两个模型：

```
from django.db import models

class Person(models.Model):
    username = models.CharField(max_length=100)
    birthdate = models.DateField()

class Book(models.Model):
    name = models.CharField(max_length=100)
    author = models.ForeignKey(Person, on_delete=models.CASCADE)
```

表 `Book` 的字段 `author` 是表 `Person` 的外键，我们试用 Django 原生的 `Serializer` 模块来对 `Book` 实例序列化：

```
from django.core import serializers
book_json = serializers.serialize("json", Book.objects.get(pk=1))
```

JSON 序列化结果如下：

```
{
    "pk": 1,
    "model": "store.book",
    "fields": {
        "name": "Mostly Harmless",
        "author": 42
    }
}
```

这个 `"author": 42` 对用户来说相当于未知，我们需要的是 `Person` 表中主键为 42 的用户姓名，即 `username` 的值。

## 解决方案

在 Django 官方文档的「序列化」一节中提到了用 `models.Manager` 处理的方案；在搜索解决方案过程中，也接触到 `Django-REST-Framework`(DRF) ，了解到 DRF 中的 `Serializer` 模块也能解决这类问题。那我们不妨对比一下两种解决方案。

### 方案一：models.Manager

根据文档，要返回自然主键，我们需要定义一个模型管理器，创建一个 `get_by_natural_key` 方法，如下：

```
from django.db import models

class PersonManager(models.Manager):
    def get_by_natural_key(self, username):
        return self.get(username=username)

class Person(models.Model):
    username = models.CharField(max_length=100)
    birthdate = models.DateField()
    objects = PersonManager()
```

然后再次序列化 `Book` 实例：

```
from django.core import serializers
book_json = serializers.serialize("json", Book.objects.get(pk=1), use_natural_foreign_keys=True)
```

得到新的结果如下：

```
{
    "pk": 1,
    "model": "store.book",
    "fields": {
        "name": "Mostly Harmless",
        "author": ["DouglasAdams"]
    }
}
```

如果需要对其他应用的数据模型做修改，例如使用了 `django.auth.User`(默认认证后端)作为 `Book` 的外键，要想不修改 `User` 模型又使用新的模型管理器，可以使用代理模式完成：

```
from django.db import models

class NewManager(models.Manager):
    # ...
    pass

class MyPerson(Person):
    objects = NewManager()

    class Meta:
        proxy = True
```

总的来说，这个方案可以完美解决我所遇到的问题，代码量稍微大一些，但是也更灵活。

### 方案二：DRF 的 Serializer

下面我们试试用 Django-REST-Framework 的序列化模块：

```
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    author_name = serializers.CharField(source='author.username')

    class Meta:
        model = Book
        fields = '__all__'
```

这段代码表示，在序列化 `Book` 实例时，添加一个新的属性 `author_name`，该值的来源为 `source` 参数定义的外键 `author` 实例的自然主键 `username`。

然后是执行序列化的过程：

```
queryset = Book.objects.get(pk=1)
BookSerializer(instance=queryset)
```

序列化结果：

```
{
    "id": 1,
    "name": "Mostly Harmless",
    "author": 42,
    "author_name": "DouglasAdams"
}
复制代码
```

当然，序列化一批 `Book` 实例也是可以的：

```
queryset = Book.objects.all()
BookSerializer(instance=queryset, many=True)
```

序列化结果：

```
[
    {
        "id": 1,
        "name": "Mostly Harmless",
        "author": 42,
        "author_name": "DouglasAdams"
    },
    {
        "id": 2,
        "name": "Harry Potter",
        "author": 2,
        "author_name": "JKRowling"
    }
]
```

可以看到，使用 DRF 的序列化模块返回自然主键，不仅代码清晰改动少，而且效果也很不错，序列化数据少了一个层级，对前端也是十分友好的。

### 方案三：手动修改序列化后的外键

当然，还有一种最傻也是最容易想到的办法，就是在序列化后，手动修改 JSON 串中对应的外键值为自然主键值。

这种做法可以得到和方案一一样的效果，但是遇到查询结果为列表时我们需要遍历替换。同时试想一下，如果我们在每个视图中都这么处理，那代码会变得十分糟糕。不建议使用该方案。

## 总结

对比两种序列化方案，我个人更偏向于 DRF 优雅的处理方式。当然，除了序列化，DRF 还有很多功能，例如分页等，强烈建议学习学习。

当然，可能不存在最好的最好的技术方案，遇到这类问题选择最合适自己的就好。也可能还有更多的方法可以解决标题的问题，也欢迎留言探讨！

## 参考

- [docs.djangoproject.com/zh-hans/2.2…](https://docs.djangoproject.com/zh-hans/2.2/topics/serialization/#natural-keys)
- [docs.djangoproject.com/en/2.2/topi…](https://docs.djangoproject.com/en/2.2/topics/db/models/#proxy-models)
- [www.django-rest-framework.org/api-guide/f…](https://www.django-rest-framework.org/api-guide/fields/#using-source)
# Django ORM Cookbook精选摘录(下) 使用信号和重写save方法哪个好？

Django ORM Cookbook是一本不错的关于Django进阶的英文书籍，里面回答了很多我们在利用Django开发Web APP过程中经常遇到的问题，还有许多值得推荐的技巧。这是下篇，上篇文章见[Django ORM Cookbook精选摘录(上)](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMjM5OTMyODA4Nw%3D%3D%26mid%3D2247484366%26idx%3D1%26sn%3Dfda7882ce5e378325a5d9137d303e98d%26chksm%3Da73c63f6904beae01ad9048eb74ff5e616bd271eff3640fdb3b12f5486b72f70a85a121f7ec3%26scene%3D21%23wechat_redirect)。我们着重分享下在Django项目中保持不同模型数据同步更新，使用信号signals和重写save方法哪个更好？

**1. 使用refresh_from_db重新载入一个字段更改后过的对象**

由于Django对queryset自带缓存的属性，当你对一个对象或很多个对象的某些字段进行更新后，你还需要使用refresh_from_db()载入字段更改过后的对象或查询集，要不然内存里还是老的数据。

```python
class TestORM(TestCase):
    def test_update_result(self):
        userobject = User.objects.create(username='testuser', first_name='Test', last_name='user')
        User.objects.filter(username='testuser').update(username='test1user')
        # At this point userobject.val is still testuser, but the value in the database
        # was updated to test1user. The object's updated value needs to be reloaded
        # from the database.
        userobject.refresh_from_db()
        self.assertEqual(userobject.username, 'test1user')
```

**2. 如何根据现有数据库转化为Django模型**

如果你的现有数据库已经有了很多数据表，希望根据这些数据表生成Django模型，首先你应该在settings.py配置好数据库信息，然后使用如下命令：

```text
$ python manage.py inspectdb > models.py
```

输入的文件夹会位于当前工作目录，你只需将生成的模型文件移入每个app对应的文件夹即可。

**3. 保持不同模型数据同步更新**

假如你有如下3个模型，类别(Category), 英雄(Hero)和坏人(Villain), 你希望统计每个类别下英雄和坏人的数量，并保持这个数据实时更新。比如每次创建一个英雄时，对应category模型的hero_count也增加1。我们来看看应该如何实现。

```python
class Category(models.Model):
    name = models.CharField(max_length=100)
    hero_count = models.PositiveIntegerField()
    villain_count = models.PositiveIntegerField()

    class Meta:
        verbose_name_plural = "Categories"


class Hero(models.Model):
    name = models.CharField(max_length=100)
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    # ...


class Villain(models.Model):
    name = models.CharField(max_length=100)
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    # ...
```

这时你可以这么做，重写save()方法，同步更新Category模型数据。

```python
class Hero(models.Model):
    # ...

    def save(self, *args, **kwargs):
        if not self.pk:
            Category.objects.filter(pk=self.category_id).update(hero_count=F('hero_count')+1)
        super().save(*args, **kwargs)


class Villain(models.Model):
    # ...

    def save(self, *args, **kwargs):
        if not self.pk:
            Category.objects.filter(pk=self.category_id).update(villain_count=F('villain_count')+1)
        super().save(*args, **kwargs)
```

注意我们并没有使用 `self.category.hero_count` `+=` `1`, 而是使用了 `update` 和F方法来更新字段。这样做的好处是减少数据查询次数，节省内存。如果使用self.category.hero_count方法，我们至少需要进行两次数据库操作，先从数据库读category对象，载入内存，更新字段，然后在写入数据库。而我们使用update和F方法时，我们只需要对数据库进行一次写入操作即可，同时避免将整个category对象载入内存。

一个替代方法是使用Django信号 signals，比如：

```python
from django.db.models.signals import pre_save
from django.dispatch import receiver

@receiver(pre_save, sender=Hero, dispatch_uid="update_hero_count")
def update_hero_count(sender, **kwargs):
    hero = kwargs['instance']
    if hero.pk:
        Category.objects.filter(pk=hero.category_id).update(hero_count=F('hero_count')+1)

@receiver(pre_save, sender=Villain, dispatch_uid="update_villain_count")
def update_villain_count(sender, **kwargs):
    villain = kwargs['instance']
    if villain.pk:
        Category.objects.filter(pk=villain.category_id).update(villain_count=F('villain_count')+1)
```



Django项目中经常使用到信号，但使用信号的代码更难阅读和维护。在保持不同模型数据同步更新时，到底使用信号signals，还是重写save方法更好？作者也给出了建议：

- 当你的字段依赖于一个你可以控制的模型，推荐使用重写 `.save`
- 如果你的字段依赖于一个你不能控制第三方app的模型，使用信号。
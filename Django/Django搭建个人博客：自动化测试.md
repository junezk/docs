# Django搭建个人博客：自动化测试

测试是伴随着开发进行的，**开发有多久，测试就要多久**。本教程已经进行了30多章了，都是如何测试的？当然是`runserver`啦！每当开发新功能后，都需要运行服务器，假装自己就是用户，测试是否运行正常。

这样的人工测试优点是非常直观，你看到的和用户看到的是完全相同的。但是缺点也很明显：

- **效率低。**在开发时可能你需要反复的修改代码、测试功能，这样重复查看几十次甚至几百次网页时会相当的让人烦躁。
- **容易遗漏bug。**随着你的项目越来越复杂，组件之间的交互也更加复杂。修改某一个组件可能会导致另一个组件出现意想不到的bug，但是在人工测试时却很难检查出来，总不能每写几行代码就把整个网站统统检查一遍吧。过了很久之后你终于发现了这个bug，但此时你已经搞不清它来源于什么地方了。
- **有的测试不方便进行。**比如说有个功能，限制每个用户每天发表评论不能超过10条，人工测试就显得比较麻烦，特别是需要反复调试的时候。

为了解决人工测试的种种问题，`Django`引入了`Python`标准库的单元测试模块，也就是**自动化测试**了：你可以写一段代码，让代码帮你测试！（程序员是最会偷懒的职业..）代码会忠实的完成测试任务，帮助你从繁重的测试工作中解脱出来。除此之外，自动化测试还有以下优点：

- **预防错误。**当应用过于复杂时，代码的意图会变得非常不清晰，甚至你都看不懂自己写的代码，这是很常见的。而测试就好像是从内部审查代码一样，可以帮助你发现微小的错误。
- **有利于团队协作。**良好的测试保证其他人不会不小心破坏了你的代码（也保证你不会不小心弄坏别人的..）。现在已经不是单打独斗出英雄的年代了，想要成为优秀的Django程序员，你必须擅长编写测试！

虽然学习自动化测试不会让你的博客增加一丝丝的功能，但是可以**让代码更加强壮**，所以我觉得很有必要拿出一章来专门讲讲。

> [Django官方文档的第5部分](https://link.juejin.im/?target=https%3A%2F%2Fdocs.djangoproject.com%2Fzh-hans%2F2.2%2Fintro%2Ftutorial05%2F)讲测试讲得非常的好，并且有中文版本。本章节就大量借鉴了官方文档，也非常非常推荐读者去拜读。

## 第一个测试

### 给我bug！

为了演示测试是如何工作的，让我们首先在**文章模型**中写个有bug的方法：

```
article/models.py

from django.utils import timezone

class ArticlePost(models.Model):
    ...

    def was_created_recently(self):
        # 若文章是"最近"发表的，则返回 True
        diff = timezone.now() - self.created
        if diff.days <= 0 and diff.seconds < 60:
            return True
        else:
            return False
```

这个方法用于检测当前文章是否是最近发表的。

> 这个方法稍微扩展一下就会变得非常实用。比如可以将博文的发表日期显示为“刚刚”、“3分钟前”、“5小时前”等相对时间，用户体验将大有提升。

仔细看看，它是没办法正确判断“未来”的文章的：

```
>>> import datetime
>>> from django.utils import timezone
>>> from article.models import ArticlePost
>>> from django.contrib.auth.models import User

# 创建一篇"未来"的文章
>>> future_article = ArticlePost(author=User(username='user'), title='test',body='test', created=timezone.now() + datetime.timedelta(days=30))

# 是否是“最近”发表的？
>>> future_article.was_created_recently()
True
```

未来发生的肯定不是最近发生的，因此代码是错误的。

### 写个测试暴露bug

接下来就要写测试用例，将测试转为自动化。

还记得最初生成**文章app**时候的目录结构吗？

```
article
 │  admin.py
 │  apps.py
 │  models.py
 │  tests.py
 │  views.py
 │  __init__.py
 │
 └─migrations
       └─ __init__.py
```

这个`tests.py`就是留给你写测试用例的地方了：

```
article/tests.py

from django.test import TestCase

import datetime
from django.utils import timezone
from article.models import ArticlePost
from django.contrib.auth.models import User


class ArticlePostModelTests(TestCase):

    def test_was_created_recently_with_future_article(self):
        # 若文章创建时间为未来，返回 False
        author = User(username='user', password='test_password')
        author.save()

        future_article = ArticlePost(
            author=author,
            title='test',
            body='test',
            created=timezone.now() + datetime.timedelta(days=30)
            )

        self.assertIs(future_article.was_created_recently(), False)
```

基本就是把刚才在`Shell`中的测试代码抄了过来。有点不同的是末尾这个`assertIs`方法，了解**“断言”**的同学会对它很熟悉：它的作用是检测方法内的两个参数是否完全一致，如果不是则抛出异常，提醒你这个地方是有问题滴。

接下来运行测试：

```
(env) > python manage.py test
```

运行结果如下：

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_was_created_recently_with_future_article (article.tests.ArticlePostModelTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "E:\django_project\my_blog\article\tests.py", line 19, in test_was_created_recently_with_future_article
    self.assertIs(future_article.was_created_recently(), False)
AssertionError: True is not False

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

这里面名堂就很多了：

- 首先测试系统会在所有以`tests`开头的文件中寻找测试代码
- 所有`TestCase`的子类都被认为是测试代码
- 系统创建了一个特殊的数据库供测试使用，即所有测试产生的数据不会对你自己的数据库造成影响
- 类中所有以`test`开头的方法会被认为是测试用例
- 在运行测试用例时，`assertIs`抛出异常，因为`True is not False`
- 完成测试后，自动销毁测试数据库

测试系统明确指明了错误的数量、位置和种类等信息，请读者细细品尝。

### 修正bug

既然通过测试找到了bug，那接下来就要把代码进行修正：

```
article/models.py

from django.utils import timezone

class ArticlePost(models.Model):
    ...

    def was_created_recently(self):
        diff = timezone.now() - self.created
        
        # if diff.days <= 0 and diff.seconds < 60:
        if diff.days == 0 and diff.seconds >= 0 and diff.seconds < 60:
            return True
        else:
            return False
```

重新运行测试：

```
(env) > python manage.py test

Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
Destroying test database for alias 'default'...
```

这次代码顺利通过了测试。

可以肯定的是，在往后的开发中，这个bug不会再出现了，因为你只需要运行一遍测试，就会立即得到警告。**可以认为项目的这一小部分代码永远是安全的**。

### 更全面的测试

既然一个测试用例就可以保证一小段代码永远安全，那我写一堆测试岂不是可以保证整个项目永远安全吗？确实如此，这个买卖绝对是不亏的。

因此我们继续再增加几个测试，全面强化代码：

```python
article/tests.py

...

from django.test import TestCase

import datetime
from django.utils import timezone
from article.models import ArticlePost
from django.contrib.auth.models import User


class ArticlePostModelTests(TestCase):

    def test_was_created_recently_with_future_article(self):
        # 若文章创建时间为未来，返回 False
        ...

    def test_was_created_recently_with_seconds_before_article(self):
        # 若文章创建时间为 1 分钟内，返回 True
        author = User(username='user1', password='test_password')
        author.save()
        seconds_before_article = ArticlePost(
            author=author,
            title='test1',
            body='test1',
            created=timezone.now() - datetime.timedelta(seconds=45)
            )
        self.assertIs(seconds_before_article.was_created_recently(), True)

    def test_was_created_recently_with_hours_before_article(self):
        # 若文章创建时间为几小时前，返回 False
        author = User(username='user2', password='test_password')
        author.save()
        hours_before_article = ArticlePost(
            author=author,
            title='test2',
            body='test2',
            created=timezone.now() - datetime.timedelta(hours=3)
            )
        self.assertIs(hours_before_article.was_created_recently(), False)

    def test_was_created_recently_with_days_before_article(self):
        # 若文章创建时间为几天前，返回 False
        author = User(username='user3', password='test_password')
        author.save()
        months_before_article = ArticlePost(
            author=author,
            title='test3',
            body='test3',
            created=timezone.now() - datetime.timedelta(days=5)
            )
        self.assertIs(months_before_article.was_created_recently(), False)
```

现在我们拥有了4个测试，来保证`was_created_recently()`方法对于**过去**、**最近**、**未来**中的4种情况都返回正确的值。你还可以继续扩展，直到你觉得完全没有任何bug藏匿的可能性为止。

在实际的开发中，有些难缠的bug会把自己伪装得非常的好，而不是像教程这样明确的知道它就在那里。有了自动化测试，无论以后你的项目怎么变化、app交互多么的复杂，只要在测试中写好的逻辑就一定是符合预期的，而你所需要做的只是运行一条测试指令而已。

> 虽然教程中仅使用了`assertIs`，但实际上Django中的断言有大概几十种之多，比如`assertEqual`、`assertContains`等，并且还在不断更新。详见[Python标准断言](https://link.juejin.im/?target=https%3A%2F%2Fdocs.python.org%2F3%2Flibrary%2Funittest.html%23unittest.TestCase)和[Django扩展断言](https://link.juejin.im/?target=https%3A%2F%2Fdocs.djangoproject.com%2Fzh-hans%2F2.2%2Ftopics%2Ftesting%2Ftools%2F%23assertions)

## 测试视图

上面的测试都是针对模型的。**视图**该怎么测试？如何通过测试系统模拟出用户的请求呢？

答案是`TestCase`类提供了一个供测试使用的`Client`来模拟用户通过请求和视图层代码的交互。

以**文章详情视图**的**浏览量统计**为例，比较容易出现的潜在bug有：

- 增加的浏览量未能正常保存进数据库（即每次请求则浏览量+1）
- 增加浏览量的同时，`updated`字段也错误的一并更新

所以有针对的写2条测试。新写一个专门**测试视图的类**，与前面的**测试模型的类**区分开：

```python
article/tests.py

...
from time import sleep
from django.urls import reverse


class ArticlePostModelTests(TestCase):
    ...


class ArtitclePostViewTests(TestCase):

    def test_increase_views(self):
        # 请求详情视图时，阅读量 +1
        author = User(username='user4', password='test_password')
        author.save()
        article = ArticlePost(
            author=author,
            title='test4',
            body='test4',
            )
        article.save()
        self.assertIs(article.total_views, 0)

        url = reverse('article:article_detail', args=(article.id,))
        response = self.client.get(url)

        viewed_article = ArticlePost.objects.get(id=article.id)
        self.assertIs(viewed_article.total_views, 1)

    def test_increase_views_but_not_change_updated_field(self):
        # 请求详情视图时，不改变 updated 字段
        author = User(username='user5', password='test_password')
        author.save()
        article = ArticlePost(
            author=author,
            title='test5',
            body='test5',
            )
        article.save()

        sleep(0.5)

        url = reverse('article:article_detail', args=(article.id,))
        response = self.client.get(url)

        viewed_article = ArticlePost.objects.get(id=article.id)
        self.assertIs(viewed_article.updated - viewed_article.created < timezone.timedelta(seconds=0.1), True)
```

注意看代码是如何与**视图层**交互的：`response = self.client.get(url)`向视图发起请求并获得了响应，剩下的就是从数据库中取出更新后的数据，并用**断言**语句来判断代码是否符合预期了。

运行测试：

```
(env) > python manage.py test

Creating test database for alias 'default'...
System check identified no issues (0 silenced).
......
----------------------------------------------------------------------
Ran 6 tests in 0.617s

OK
Destroying test database for alias 'default'...
复制代码
```

6条测试用例全部通过。

## 越多越好的测试

仅仅是app中的两个非常小的功能，就已经写了6条测试用例了，并且还可以继续扩展。除此之外，其他的每个模型、视图都可以扩展出几十甚至上百条测试，这样下去代码总量很快就要失去控制了，并且相对于业务代码来说，测试代码显得繁琐且不够优雅。

**但是没关系！**就让测试代码继续肆意增长吧。大部分情况下，你写完一个测试之后就可以忘掉它了。在你继续开发的过程中，它会一直默默无闻地为你做贡献的。最坏的情况是当你继续开发的时候，发现之前的一些测试现在看来是多余的。但是这也不是什么问题，多做些测试也不错。

## 深入代码测试

在前面的测试中，我们已经从模型层和视图层的角度检查了应用的输入输出，但是模板呢？虽然可以用`assertInHTML`、`assertJSONEqual`等断言大致检查模板中的某些内容，但更加近似于浏览器的检查就要使用`Selenium`等测试工具（毕竟Django的重点是后端而不是前端）。

`Selenium`不仅可以测试 Django 框架里的代码，甚至还可以检查 JavaScript代码。它假装成是一个正在和你站点进行交互的浏览器，就好像有个真人在访问网站一样。Django 提供了`LiveServerTestCase`来和`Selenium`这样的工具进行交互。

关于测试的话题这里只是开了个头，读者可以继续阅读下面的内容进一步了解：

- [Django: Writing and running tests](https://link.juejin.im/?target=https%3A%2F%2Fdocs.djangoproject.com%2Fen%2F2.2%2Ftopics%2Ftesting%2Foverview%2F)
- [Django: Testing tools](https://link.juejin.im/?target=https%3A%2F%2Fdocs.djangoproject.com%2Fen%2F2.2%2Ftopics%2Ftesting%2Ftools%2F)
- [Django: Advanced testing topics](https://link.juejin.im/?target=https%3A%2F%2Fdocs.djangoproject.com%2Fen%2F2.2%2Ftopics%2Ftesting%2Fadvanced%2F)
- [Selenium官方文档](https://link.juejin.im/?target=https%3A%2F%2Fselenium-python.readthedocs.io%2Fgetting-started.html)

## 总结

有一帮崇尚“测试驱动”的开发者，他们开发时先写测试代码，然后才写业务代码。而普通开发者通常是先写业务代码，再写测试代码，这也是没问题的。但如果你已经写了很多业务代码了，再回头写测试确实有些无从下手，那么至少在以后写新功能时，记得加上测试。测试写得好不好，甚至比功能本身更能看出编程水平。

测试可以让代码更加强壮。项目没出bug时，皆大欢喜，有没有测试都一样；一旦出现难缠的bug，你就会无比想念一套完善的测试代码了。

博主写自己的网站时就没有对测试给与足够的重视，回想起来走了很多弯路。希望读者以前车之鉴，培养良好的编程习惯。

------

- 有疑问请在[杜赛的个人网站](https://link.juejin.im/?target=http%3A%2F%2Fwww.dusaiphoto.com)留言，我会尽快回复。
- 或Email私信我：dusaiphoto@foxmail.com
- 项目完整代码：[Django_blog_tutorial](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fstacklens%2Fdjango_blog_tutorial)
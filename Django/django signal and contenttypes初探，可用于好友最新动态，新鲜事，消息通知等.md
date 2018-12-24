# django signal and contenttypes初探，可用于好友最新动态，新鲜事，消息通知等

django\_signal\_contenttypes
============================

#### django signal and contenttypes 信号和内容类型 contenttypes framework初探，可用于好友最新动态，新鲜事，消息通知等  

githu地址:[https://github.com/pushiqiang/django\_signal\_contenttypes](https://github.com/pushiqiang/django_signal_contenttypes)  

django中得signals和操作系统（linux）中的signal完全是两会事，后者的signal是软件中断，提供一种处理异步事件得方法，信号是系统定义好的，可用作进程间传递消息得一种方法，而django中的信号只是一个普通的类，不能跨进程，看其代码更像一个回调函数。  

django的signal结合contenttypes可以实现好友最新动态，新鲜事，消息通知等功能。  
总体来说这个功能就是在用户发生某个动作的时候将其记录下来或者附加某些操作，比如通知好友。要实现这种功能可以在动作发生的代码里实现也可以通过数据库触发器等实现，但在django中，一个很简单的方法的就是使用signals。  

当django保存一个object的时候会发出一系列的signals，可以通过对这些signals注册listener，从而在相应的signals发出时执行一定的代码。  
使用signals来监听用户的动作有很多好处：

1. 不管这个动作是发生在什么页面，甚至在很多页面都可以发生这个动作，都只需要写一次代码来监听保存object这个动作就可以了。
2. 可以完全不修改原来的代码就可以添加监听signals的功能。
3. 你几乎可以在signals监听代码里写任何代码，包括做一些判断是不是第一次发生此动作还是一个修改行为等等。  

想要记录下每个操作，同时还能追踪到这个操作的具体动作。  
*首先用信号机制，监听信号，实现对信号的响应函数，在响应函数中记录发生的动作（记录在一张记录表，相当于下文的Event）。  
*其次就是为了能追踪到操作的具体动作，必须从这张表中得到相应操作的model，这就得用到下面说的ContentType framework

###### contenttypes framework

在django中，有一个记录了项目中所有model元数据的表，就是ContentType，表中一条记录对应着一个存在的model，所以可以通过一个ContentType表的id和一个具体表中的id找到任何记录，及先通过ContenType表的id可以得到某个model，再通过model的id得到具体的对象。  
ContentType表如下：  

    class ContentType(models.Model):
        name = models.CharField(max_length=100)
        app_label = models.CharField(max_length=100)
        model = models.CharField(_('python model class name'), max_length=100)
        objects = ContentTypeManager()

通过app\_label和model这2个字段，使用django.db.models.get\_model这个方法就可以找出原来所对应的Model。而有了原来的Model的定义，再通过使用主键，就可以找到这个Model所对应的某条记录了。  

什么是contenttypes framework（原文）：  
Django includes a “contenttypes” application that can track all of the models installed in your Django-powered project, providing a high-level, generic interface for working with your models.  
对于新鲜事这个功能来说就是使用Generic relations来产生一个特殊的外键，它不像models.ForeignKey那样，必须指定一个Model来作为它指向的对象。Generic relations可以指向任何Model对象，有点像C语言中 void* 指针。  
这样关于保存用户所产生的这个动作，比如用户写了一片日志，我们就可以使用Generic relations来指向某个Model实例比如Post，而那个Post实例才真正保存着关于用户动作的完整信息，即Post实例本身就是保存动作信息最好的地方。这样我们就可以通过存取Post实例里面的字段来描述用户的那个动作了，需要什么信息就往那里面去取。而且使用Generic relations的另外一个好处就是在删除了Post实例后，相应的新鲜事实例也会自动删除。  

怎么从这张操作记录表中得到相应操作的model呢，这就得用到generic.GenericForeignKey，它是一个特殊的外键，可以指向任何Model的实例，在这里就可以通过这个字段来指向类似Post这样保存着用户动作信息的Model实例。  
代码如下：  

    # -*- coding: utf-8 -*-
    from django.db import models
    from django.contrib.auth.models import User
    from django.contrib.contenttypes.models import ContentType
    from django.contrib.contenttypes import generic
    from django.db.models import signals
    
    class Post(models.Model):
        author = models.ForeignKey(User)
        title = models.CharField(max_length=255)
        content = models.TextField()
        created = models.DateTimeField(u'发表时间', auto_now_add = True)
        updated = models.DateTimeField(u'最后修改时间', auto_now = True)
    
        events = generic.GenericRelation('Event')
    
        def __unicode__(self):
            return self.title
        def description(self):
            return u'%s 发表了日志《%s》' % (self.author, self.title)
    
    class Event(models.Model):
        user = models.ForeignKey(User)
        #以下三个成员必须指定
        content_type = models.ForeignKey(ContentType)
        object_id = models.PositiveIntegerField()
        event = generic.GenericForeignKey('content_type', 'object_id')
    
        created = models.DateTimeField(u'事件发生时间', auto_now_add = True)
        def __unicode__(self):
            return  u"%s的事件: %s" % (self.user, self.description())
        def description(self):
            return self.event.description()
    
    def post_post_save(sender, instance, signal, *args, **kwargs):
        post = instance
        print sender,'\n', instance, '\n',signal
        print post.created ,post.updated
        event = Event(user=post.author,event = post)
        event.save()
    
    signals.post_save.connect(post_post_save, sender=Post)
    ###当然你也可以监听多个model的post_save信号
    ###signals.post_save.connect(post_post_save, sender=Book)
    只要model中有object的保存操作，都将执行post_post_save函数，故可以在这个接受函数中实现通知好友等功能

Event中使用 GenericForeignKey() 来指向其它的 Model 实例。为了使用它，还需要在 Model 中定义content\_type 和 object\_id 才可以。其中 content\_type来自 ContentType 这个Model，记录Event所指向的其他 Model 实例的名字。object\_id则是表示所指向的Model实例的id。  

实际上根据上面的解释它只要有 content\_type 和 object\_id 两个字段就够了，不过我们总是需要亲自指定两个字段的值。而 GenericForeignKey 出现的目的就是要把这个过程给自动化了，只要给 content\_object 赋一个对象，就会自动得根据这个对象的元数据 ，给content\_type 和 object_id 赋值了。  
上面示例可通过如下代码获得具体的对象事件：  

        entity = Event.objects.get(pk=1)
        object = entity.event

###### signal

前面说到django在保存一个object的时候会发出一系列signals，在这里我们所监听的是signals.post_save这个signal，这个signal是在django保存完一个对象后发出的，django中已定义好得一些signal, 在django/db/models/signal.py中可以查看，同时也可以自定义信号。  

利用connect这个函数来注册监听器， connect原型为：  
def connect(self, receiver, sender=None, weak=True, dispatch_uid=None):  
第一个参数是要执行的函数，第二个参数是指定发送信号的Class，这里指定为Post这个Model，对其他Model所发出的signal并不会执行注册的函数。  
instance这个参数，即刚刚保存完的Model对象实例。创建事件的时候看到可以将post这个instance直接赋给generic.GenericForeignKey类型的字段，从而event实例就可以通过它来获取事件的真正信息了。  

最后有一点需要的注意的是，Post的Model定义里现在多了一个字段：  
events = generic.GenericRelation(‘Event’)

通过这个字段可以得到与某篇post相关联的所有事件，最重要的一点是如果没有这个字段，那么当删除一篇post的时候，与该post关联的事件是不会自动删除的。反之有这个字段就会进行自动的级联删除。
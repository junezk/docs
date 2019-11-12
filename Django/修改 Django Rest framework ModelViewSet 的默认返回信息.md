# 修改 Django Rest framework ModelViewSet 的默认返回信息

drf API 接口默认返回的只是数据的JSON字符串，不包含其它的接口信息，甚至有时候格式也不一致，这对前端不太友好，所以我希望把这个返回修改一下，往返回信息里面添加一些额外的信息，如**状态**，**说明**等信息

首先，要知道在哪里修改这个返回，翻一下源码发现这些返回信息是写在**create()**、**list()**、**retrieve()**、**update()**、**destroy()\**这些方法里的，这些信息都放在\**Response**对象里面返回到前端页面去。

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264e821d7b52)

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264e63eb200d)

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264e60411936)

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264e645b0018)

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264e6f4efb21)

知道在哪里改后，接着就是改这些方法的返回信息了，我希望给前端返回这样格式的信息:

> {'code': 1, 'msg': '成功', 'errors': {}, 'data': []} # errors 放具体的错误信息，data 放返回去的数据

然后用这个 dict 直接替换 Response 里面的信息就行了，如下：

![这里就放一张，其它的类似修改](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264e7d299e3d)

替换后，尝试请求下接口，默认的返回格式就变成我们定义的样子了，但是，任务还没完成，在只有一个接口的情况下，这样改改没什么，但是如果有十几个，几十个接口的话，这样一个个接口去改，不但累人还不好维护，所以这里换种方法。

我们通过定义一个继承于 ModelViewSet 的类，然后所有的接口不再继承 ModelViewSet 而继承于这个 `CustomViewSet`，这就不用一个个接口去改了，还方便维护。

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264e9880f43b)

到此为止，任务依然没有完成，成功的返回我们是定义好了，但是接口发生异常的时候，返回的信息还是默认的那些信息，因为我们改的只是正常处理完成后的成功返回，异常返回我们是没有处理，那异常信息的返回应该在哪里改？

一顿调试后，我发现了当接口发生异常后，是由`exception_handler`这个方法去处理的。这个方法定义在`rest_framework/views.py`里，因为这个方法是独立出来的，不是定义在 ModelViewSet 里面，所以就没办法通过重写的方法来改了，我们把整个方法复制出来，放在`CustomViewSet`下面（当然没有规定要放在一起，放在哪里都行，方便管理就行）。

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264ea871ea16)

可以看出，第95行到98行就是我们需要改的地方，调试看一下这个对象是怎样的，然后进一步把里面我们需要的信息提取出来，改成这个样子：

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264e9c00940b)

最重要的一步！要在`settings.py`里面的 `REST_FRAMEWORK`（自己添加的）里添加多一条这样的配置：

```
REST_FRAMEWORK = {
    ······
    ······
    
    'EXCEPTION_HANDLER': 'common.views.exception_handler'
}
```

这里指定了使用哪个方法作为接口异常的处理器，这个设置成我们刚刚改那个。

PS: 虽然做了上面的修改后，就修改了系统上大部分的返回信息，但是有时候我们需要定一个接口不是继承 ModelViewSet 的，这样这些修改就不能继承到了，在这样的情况下，每写一次接口，都要自己手动敲一遍这些信息，难免会手快敲错了，在这里我定义了一个类:

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264eb689a091)

那上面的接口就可以改成这个样子了，更加的方便维护：

![img](修改 Django Rest framework ModelViewSet 的默认返回信息.assets/16ba264ecaef93bf)

### 总结：

1. 自定义一个继承于`ModelViewSet`的类，重写**create**、**list**、**update**、**retrieve**、**destroy**这些方法，修改Response
2. 复制`rest_framework/views.py/exception_handler`方法，修改里面处理异常信息的代码
3. 在`settings.py`的`REST_FRAMEWORK`配置里面添加`'EXCEPTION_HANDLER': 'common.views.exception_handler'`
4. （可选）新建一个返回信息类`ResturnMsg`，代替手动写返回信息，避免错误。


 
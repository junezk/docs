# django rest framework mixins小结

### 引言

  本篇对drf中的mixins进行简要的分析总结。
  mixins在drf中主要配合viewset共同使用，实现http方法与mixins的相关类与方法进行关联。关于[viewset](https://juejin.im/post/5a66d262f265da3e317e4cc5)可以看我另外一篇博客，这里不过多介绍。

```python
from rest_framework import viewsets
```

  在这个viewset中，只要有5类Minxin，他们与http方法对应如下：

![Mixins](https://user-gold-cdn.xitu.io/2018/1/23/1612249e5cba8220?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

下面，我们将逐个Mixins介绍!

### 1. CreateModelMixin

```python
# 源码
class CreateModelMixin(object):
    """
    Create a model instance ==>创建一个实例
    """
    def create(self, request, *args, **kwargs):
    
	# 获取相关serializer
        serializer = self.get_serializer(data=request.data)
        
        # 进行serializer的验证
        # raise_exception=True,一旦验证不通过，不再往下执行，直接引发异常
        serializer.is_valid(raise_exception=True)
        
        # 调用perform_create()方法，保存实例
        self.perform_create(serializer)
        
        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)

    def perform_create(self, serializer):
    # 保存实例
        serializer.save()

    def get_success_headers(self, data):
        try:
            return {'Location': str(data[api_settings.URL_FIELD_NAME])}
        except (TypeError, KeyError):
            return {}
```

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="835" height="777"></svg>)

  由上图可以看出这个类的一个逻辑，其中，perform_create( )对serializer直接进行save保存，当在一些情境下，我们需要对perform_create( )进行重写。 那么我们现在可能有一个下面的需要：

  假设现在有一个course课程model，里面维持了一个数，记录课程收藏数，还存在一个用户收藏userfav的model(应当有一个外键指向course)，当一个用户对课程进行收藏，理论上现在post进来的应该是userfav的instance，显然，我们还需要对相应course的收藏数进行+1。
  这个时候，我们就需要重写perform_create( )方法！

```python
def perform_create(self, serializer):
# 重写save的逻辑
	instance = serializer.save()
	course = instance.course
	course.fav_num += 1
	course.save()
```

  显然，这不是唯一的解决方法，我们还可以在seriliazer进行设置，我们还可以使用drf的信号量进行解决！

### 2. ListModelMixin

```python
# 源码
class ListModelMixin(object):
    """
    List a queryset.==> 列表页获取
    """
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())
        
        # 这是一个分页功能，如果在viewset中设置了pagination_class，那么这里就会起作用
        # 获取当前页的queryset，如果不存在分页，返回None
        page = self.paginate_queryset(queryset)
        
        if page is not None:
	    # 分页不为空，那么不能简单的执行Response(serializer.data)
	    # 还需要将相关的page信息序列化在进行响应
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
```

  ListModelMixin一般用来获取列表页，大多数情况下比较简单，不需要重写相关的方法。

### 3. RetrieveModelMixin

```python
# 源码
class RetrieveModelMixin(object):
    """
    Retrieve a model instance.==> 获取某一个对象的具体信息
    """
    def retrieve(self, request, *args, **kwargs):
    	# 一般访问的url都为/obj/id/这种新式
    	# get_object()可以获取到这个id的对象
    	# 注意在viewset中设置lookup_field获取重写get_object()方法可以指定id具体对象是什么~！
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        return Response(serializer.data)
```

  对retrieve这个方法的重写几率比较高，例如我们在增加点击数的时候，经常要对其进行一个重写。

### 4. RetrieveModelMixin

```python
# 源码
class UpdateModelMixin(object):
    """
    Update a model instance.==> 更新某个具体对象的内容
    """
    def update(self, request, *args, **kwargs):
        partial = kwargs.pop('partial', False)
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=partial)
        serializer.is_valid(raise_exception=True)
        self.perform_update(serializer)

        if getattr(instance, '_prefetched_objects_cache', None):
            # If 'prefetch_related' has been applied to a queryset, we need to
            # forcibly invalidate the prefetch cache on the instance.
            instance._prefetched_objects_cache = {}

        return Response(serializer.data)

    def perform_update(self, serializer):
        serializer.save()

    def partial_update(self, request, *args, **kwargs):
        kwargs['partial'] = True
        return self.update(request, *args, **kwargs)
```

  RetrieveModelMixin的实现逻辑基本整合了Create以及Retrieve，先得到具体的实例，再对其进行验证以及保存，如果需要对更新这个逻辑进行自定义，那么需要重写perform_update( )方法，而尽量少去重写update( )

### 5. DestroyModelMixin

```python
# 源码
class DestroyModelMixin(object):
    """
    Destroy a model instance.
    """
    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        self.perform_destroy(instance)
        return Response(status=status.HTTP_204_NO_CONTENT)

    def perform_destroy(self, instance):
        instance.delete()
```

  DestroyModelMixin的逻辑也相对比较简单，我们取CreateModelMixin下面的例子，当我们取消收藏，那么我们的DestroyModelMixin就发挥作用了。同理

```python
def perform_create(self, serializer):
	instance = serializer.save()
	course = instance.course
	if course.fav_num > 0:
		course.fav_num -= 1
	else:
		course.fav_num = 0
	course.save()
```

### 小结

  mixins相对比较好理解，本篇只是简要的分析了源码的内容以及各个mixins的逻辑，最重要的还是学会去重写它们相关的方法。
  **一般情况下，当我们在操作某一个model的时候，涉及到另外一个model中数据的修改，那么就需要对这个mixins下执行save的逻辑的方法进行重写。**

CSDN：http://blog.csdn.net/l_vip/article/details/79142105
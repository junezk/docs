# Django REST framework的各种技巧

## 1.基础讲解

### 写在最上面的话

django是一个神奇的框架，而restframework又是遵循了这个框架的另一个神奇的框架，然而由于restframework的文档稀烂无比，很多时候你必须看源码才能写出科学的代码，这挡住了很多新手的路。

要用好restframework你必须对django或者说python的几个概念有比较深刻的理解，GenericView，Mixin，子类父类集成调用，多继承时的调用顺序等等，这是用好restframework的第一步。

### 先说说rest

> REST是一种标准，restful是一种规范，根据产品需求需要定出一份方便前后端的规范，因此不是所有的标准要求都需要遵循。

### rest的一些资料

- [阮一峰的博客](http://www.ruanyifeng.com/blog/2011/09/restful)  

- [百度百科](http://baike.baidu.com/link?url=3FdyPwBm2wDmmzqQNZ0tXRRmclilHWH6rAuMU38YPEI5GwuhI9zehqUn8iDIcZKNx92CnxycX3AfrIqGMsTtb_)

### 如何用restframework实现一个（一组)api

其实就是写几个东西，就可以快速的实现 api

1.  继承某个GenericView,重写里面的某个方法，最大的是get、post、put、patch、delete这些方法，然而并不推荐（应该重写mixin里面的方法）

2.  实现一个serilizer，json化response

3.  写一个url


### 作为写框架的人，你需要考虑的事情还有那些？

每个项目总有第一个人做基础构架，这个时候就不是仅仅实现一个api就ok了，你需要考虑跟多的事情，包括

* 统一的异常处理

* api权限

* 统一的参数校验

* 缓存如何可以做的更简单统一

* 认证

* 统一的查询过滤

* 代码分层

* api的demo，具体细节之后的博客会详细讲解

继承某个Genricview，重写对应方法

```python
class CoursesView(ListCreateAPIView):

    filter_backends = (SchoolPermissionFilterBackend, filters.DjangoFilterBackend, filters.SearchFilter)
    permission_classes = (IsAuthenticated, ModulePermission)
    queryset = Course.objects.filter(is_active=True).order_by('-id')
    filter_fields = ('term',)
    search_fields = ('name', 'teacher', 'school__name')
    module_perms = ['course.course']

    def get_serializer_class(self):
        if self.request.method in SAFE_METHODS:
            return CourseFullMessageSerializer
        else:
            return CourseSerializer

    def get_queryset(self):
        return Course.objects.select_related('school', ).filter(
                is_active=True, school__is_active=True, term__is_active=True).order_by('-id')
              
    @POST('school', validators='required')
    def create(self, request, school, *args, **kwargs):
        if not SchoolPermissionFilterBackend().has_school_permission(request.user, school):
            raise Error(errors.PermissionDenied, err_message=u'没有对应学校的权限', message=u'没有对应学校的权限')
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        headers = self.get_success_headers(serializer.data)
        return Response(CourseFullMessageSerializer(serializer.instance).data, status=status.HTTP_201_CREATED, headers=headers)
```

实现一个serilizer，json化response
```python
    class CourseSerializer(serializers.ModelSerializer):
    
        class Meta:
            model = Course
            read_only_fields = ('is_active',)
    
    class CourseFullMessageSerializer(CourseSerializer):

    school = SchoolLittleMessageSerializer()
    term = serializers.CharField(source='term.name', read_only=True)
```

写一个url

```python
url(r'^courses/$', CoursesView.as_view(), name='course-list'),
```

按照这个流程你可以迅速实现大量的api，这是最基本的用法。

## 2.serializer

serializer只做一件事情，json化model对象，所以这一部分相当简单

### 讲解

拿基本的user，group为例子

首先一个关联的model

```python
class UserProfile(TimeStampedModel):

    user = models.OneToOneField(User, unique=True, db_index=True, related_name='profile')
    name = models.CharField(blank=True, max_length=255, db_index=True)
    phone = models.CharField(default='', blank=True, max_length=64)
    nickname = models.CharField(blank=True, null=True, max_length=255, db_index=True)
    avatar = models.URLField(blank=True, max_length=255, default='')
    is_cms_user = models.BooleanField(default=False, db_index=True)
    is_cms_active = models.BooleanField(default=False, db_index=True)

    class Meta:  # pylint: disable=missing-docstring
        db_table = "auth_userprofile"

    def __unicode__(self):
        return self.name
```

User对应的serializer

```python
class GroupSerializer(serializers.ModelSerializer):

    class Meta:
        model = Group
        fields = ('id', 'name')

class UserSerializer(serializers.ModelSerializer):
    groups = GroupSerializer(many=True)
    phone = serializers.CharField(source='profile.phone', read_only=True)
    name = serializers.CharField(source='profile.name', read_only=True)
    menus = serializers.SerializerMethodField()
    is_active = serializers.BooleanField(source='profile.is_cms_active')

    def get_menus(self, user):
        return get_menus(user)

    class Meta:
        model = User
        fields = ('id', 'username', 'name', 'email', 'phone', 'groups', 'menus', 'is_active')
```

一个请求的response

```json
{
    "id": 2,
    "username": "duoduo3369",
    "name": "",
    "email": "",
    "phone": "",
    "groups": [
        {
            "id": 1,
            "name": "sysadmin"
        },
        {
            "id": 17,
            "name": "大学2"
        }
    ],
    "menus": [
        {
            "menu": [
                {
                    "menu": [],
                    "codename": "information.announcement",
                    "name": "通知公告",
                    "order": 1
                },
                {
                    "menu": [],
                    "codename": "information.examinfo",
                    "name": "考试信息",
                    "order": 2
                }
            ]
       }
    ],
    "is_active": false
}
```

*   外键直接可以引用其他的serializer,例如group，可以看到response中group是嵌套的

*   外键的属性可以使用source，例如phone

*   不在原来model上的东西使用SerializerMethodField（或者在model上但是你要对这个值做一些特殊处理)


### 注意点

*   serializer可以做逻辑上的操作，然而最好不要做查询（你可以用SerializerMethodField做一些数据转换例如0变为假1变为真什么的，然而最好不要做复杂的数据库查询），这种事情可以在view上做好（注意可以用select_related减少多次查询），因为这是每一个model都要serializer一次。

*   如果说跟前端对的修改和查询使用不同的serializer，那么你就写两个，不希望修改的字段加上readonly（或者放在readonly_fields里面）


serializer的逻辑很简单，想到复杂的东西再说。

## 3.权限

django内置强大的权限系统，restframework也完全支持，为什么不用呢？

### 文档

[django permission的文档](https://docs.djangoproject.com/en/1.9/topics/auth/default/#permissions-and-authorization)  
[restframework permission的文档](http://www.django-rest-framework.org/api-guide/permissions/)

### 权限的类型

*   用户是否有访问某个api的权限

*   用户对于相同的api不同权限看到不同的数据（其实一个filter）

*   不同权限用户对于api的访问频次，其他限制等等

*   假删除，各种级联假删除


### 基本讲解

首先在django中，group以及user都可以有很多的permission，一个user会有他自身permission+所有隶属group的permission。比如：user可能显示的有3个permission，但他隶属于3个组，每个组有2个不同的权限，那么他有3+2*3个权限。

permission会在api的models.py种统一建立，在Meta中添加对应的permission然后跑migrate数据库就会有新的权限。

由于django对每一个不同的Model都会建立几个基本的权限，我会在api/models.py里面单独建一个ModulePermission的Model，没有任何其他属性，就只有一个Meta class上面对应各种权限，这就是为了syncdb用的，另外还一个原因后面再说。

```python
class ModulePermission(models.Model):
    class Meta:
        # 命名为cms设计稿里面对应 '菜单权限' 的地方, 例如用户管理
        permissions = ( 
            ("information.announcement", u"资讯管理-通知公告"),
            ("information.examinfo", u"资讯管理-考试信息"),
            ("information.memberschool", u"资讯管理-会员学校"),
            ("school.school", u"学校管理-学校管理"),
            ("course.course", u"课程管理-课程管理"),
            ("student.student", u"学生管理-学生管理"),
            ("exam.exam", u"考务管理-考试管理"),
            ("exam.room", u"考务管理-考场管理"),
            ...
        )
```


### api访问权限的具体使用

permission\_classes = (IsAuthenticated, ModulePermission)有一个ModulePermission，说明需要有对应module的权限才可以访问这个api，什么权限捏？module\_perms = \['course_course'\]，拥有了这个api的访问权限后可以看到这个。  
功能是跟学校权限有关的东西，所以需要过滤学校数据 filter_backends = \[SchoolPermissionFilterBackend,\]，这个后面再表。

```            python
class CourseDetailView(UnActiveModelMixin, DeleteForeignObjectRelModelMixin, RetrieveUpdateDestroyAPIView):

    filter_backends = [SchoolPermissionFilterBackend,]
    serializer_class = CourseSerializer
    permission_classes = (IsAuthenticated, ModulePermission)
    queryset = Course.objects.filter(is_active=True).order_by('-id')
    module_perms = ['course.course']

    def get_serializer_class(self):
        if self.request.method in SAFE_METHODS:
            return CourseFullMessageSerializer
        else:
            return CourseSerializer
```

### ModulePermission的实现

为毛把所有的权限都放在api里面呢？就是为了下面这个东西好写,因为直接从user.get\_all\_permissions拿到的permission会带模块名，放在api.models下的东东都会叫api.xxx,看下面实现就懂了

```python
# -*- coding: utf-8 -*-
from rest_framework.permissions import BasePermission, SAFE_METHODS

class ModulePermission(BasePermission):
    '''
    ModulePermission, 检查一个用户是否有对应某些module的权限

    APIView需要实现module_perms属性:
        type: list
        example: ['information.information', 'school.school']

    权限说明:
        1. is_superuser有超级权限
        2. 权限列表请在api.models.Permission的class Meta中添加(请不要用数据库直接添加)
        3. 只要用户有module_perms的一条符合结果即认为有权限, 所以module_perms是or的意思
    '''

    authenticated_users_only = True

    def has_perms(self, user, perms):
        user_perms = user.get_all_permissions()
        for perm in perms:
            if perm in user_perms:
                return True
        return False

    def get_module_perms(self, view):
        return ['api.{}'.format(perm) for perm in view.module_perms]

    def has_permission(self, request, view):
        '''
        is_superuser用户有上帝权限，测试的时候注意账号
        '''
        # Workaround to ensure DjangoModelPermissions are not applied
        # to the root view when using DefaultRouter.
        # is_superuser用户有上帝权限
        if request.user.is_superuser:
            return True

        assert view.module_perms or not isinstance(view.module_perms, list), (
            u"view需要override module属性，例如['information.information', 'school.school']"
        )

        if getattr(view, '_ignore_model_permissions', False):
            return True

        if hasattr(view, 'get_queryset'):
            queryset = view.get_queryset()
        else:
            queryset = getattr(view, 'queryset', None)

        assert queryset is not None, (
            'Cannot apply DjangoModelPermissions on a view that '
            'does not set `.queryset` or have a `.get_queryset()` method.'
        )

        return (
            request.user and
            (request.user.is_authenticated() or not self.authenticated_users_only) and
            self.has_perms(request.user, self.get_module_perms(view))
        )
        
class ModulePermissionOrReadOnly(ModulePermission):
    """
    The request is authenticated with ModulePermission, or is a read-only request.
    """

    def has_permission(self, request, view):
        return (request.method in SAFE_METHODS or super(ModulePermissionOrReadOnly, self).has_permission(request, view))
```

### 用户对于相同的api不同权限看到不同的数据

这其实是一个filter

```python
# -*- coding: utf-8 -*-
from django.db import models
from django_extensions.db.models import TimeStampedModel
from django.db.models.signals import post_save
from .signals import create_permisson

class School(TimeStampedModel):
    MIDDLE_SCHOOL = 1
    COLLEGE = 2
    school_choices = (
        (MIDDLE_SCHOOL, u"中学"),
        (COLLEGE, u"高校")
    )
    category = models.SmallIntegerField(
        choices=school_choices, db_index=True, default=MIDDLE_SCHOOL)
    name = models.CharField(max_length=255, db_index=True)
    ...
    is_active = models.BooleanField(default=True, db_index=True)
 
    class Meta:
        # 创建学校时会创建学校权限, 默认有所有学校权限
        permissions = (
            ("schoolpermission__all", u"全部学校"),
        )
 
    def __unicode__(self):
        return self.name

post_save.connect(create_permisson, sender=School)
```

上面的signal,当学校对象创建修改时会对应更新permission里面的记录，保持一致性。

```python
# -*- coding: utf-8 -*-
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType


def create_permisson(sender, instance, created=False, *args, **kwargs):
    '''创建学校时会创建学校权限'''
    from .models import School
    school = instance
    content_type = ContentType.objects.get_for_model(School)
    codename = 'schoolpermission__{}'.format(school.id)
    name = school.name
    if created:
        permission = Permission.objects.create(codename=codename,
                                               name=school.name,
                                               content_type=content_type)
    else:
        if school.is_active:
            # 如果学校建在，有可能老师把学校名字改了，更新学学校名字
            permissions = Permission.objects.filter(codename=codename, content_type=content_type)
            if not permissions.exists():
                Permission.objects.create(codename=codename,
                                          name=school.name,
                                          content_type=content_type)
            else:
                permission = permissions[0]
                if permission.name != school.name:
                    permission.name = school.name
                    permission.save()
        else:
            Permission.objects.filter(codename=codename, content_type=content_type).delete()
    return instance
```

这样只要跟学校有关的东西都必须有个叫做school的外键字段，在view中添加filter_backends = \[SchoolPermissionFilterBackend,\]，即可根据学校权限过滤

```python
# -*- coding: utf-8 -*-
from django.db.models.fields.related import ReverseSingleRelatedObjectDescriptor
 
class SchoolPermissionFilterBackend(object):
 
    def get_perms(self, user):
        '''获取用户对于学校的权限, return (True, [])
 
            超级管理员有所有用户权限,
 
        '''
        ALL_SCHOOL_PERMISSION = (True, [])
        if user.is_superuser:
            has_all_school_permission = True
            return ALL_SCHOOL_PERMISSION
 
        permissions = user.get_all_permissions()
        perms = []
        for permission in permissions:
            if permission.startswith('school.schoolpermission__'):
                perm = permission.split('school.schoolpermission__')[-1]
                if perm == 'all':
                    return ALL_SCHOOL_PERMISSION
                perms.append(perm)
        return (False, perms)
 
    def filter_queryset(self, request, queryset, view):
        user = request.user
        model_cls = queryset.model
        has_all_school_permission, perms = self.get_perms(user)
        # 如果有所有学校权限则不过滤数据
        if has_all_school_permission:
            return queryset
        # 否则根据用户拥有的学校权限过来学校有关数据
        if hasattr(model_cls, 'school') and isinstance(model_cls.school, ReverseSingleRelatedObjectDescriptor):
            queryset = queryset.filter(school__in=perms, school__is_active=True)
        return queryset
```

### 不同权限用户对于api的访问频次，其他限制

[官方文档](http://www.django-rest-framework.org/api-guide/throttling/)

然而这个我没什么可以说的，继承下重写allow_request即可。

### 细节

由于对restful规范的理解，我们知道如果定义了一个/terms/接口，那么get请求是获得list，post是新建一个term，然而如果这两个的权限不一样GenericView应该怎么写呢?答案是重写get_permissions方法（你应该怎么知道的呢？看源码）

```python
class TermsView(ListCreateAPIView):

    serializer_class = TermSerializer
    permission_classes = (IsAuthenticated, ModulePermission)
    queryset = Term.objects.filter(is_active=True).order_by('-id')
    module_perms = ['sysadmin.term']

    def get_permissions(self):
        if self.request.method in SAFE_METHODS:
            return [IsAuthenticated()]
        else:
            return [permission() for permission in self.permission_classes]
```

### 删除权限

需求是这样的：

*   假删除

*   如果没有其他东西对他有外键，则直接可以删

*   如果其他东西对他有了外键，则需要给提示（用户确定后可以删除）


首先看实现 注意继承顺序一定不能错！！！  
UnActiveModelMixin对应假删除  
DeleteForeignObjectRelModelMixin对应外键检查  
RetrieveUpdateDestroyAPIView是默认的rest的view，提供delete支持，主要是需要xxxDestroyAPIView

```python
class UnActiveModelMixin(object):
    """ 
    删除一个对象，并不真删除，级联将对应外键对象的is_active设置为false，需要外键对象都有is_active字段.
    """
    def perform_destroy(self, instance):
        rel_fileds = [f for f in instance._meta.get_fields() if isinstance(f, ForeignObjectRel)]
        links = [f.get_accessor_name() for f in rel_fileds]

        for link in links:
            manager = getattr(instance, link, None)
            if not manager:
                continue
            if isinstance(manager, models.Model):
                if hasattr(manager, 'is_active') and manager.is_active:
                    manager.is_active = False
                    manager.save()
                    raise ForeignObjectRelDeleteError(u'{} 上有关联数据'.format(link))
            else:
                if not manager.count():
                    continue
                try:
                    manager.model._meta.get_field('is_active')
                    manager.filter(is_active=True).update(is_active=False)
                except FieldDoesNotExist as ex: 
                    # 理论上，级联删除的model上面应该也有is_active字段，否则代码逻辑应该有问题
                    logger.warn(ex)
                    raise ModelDontHaveIsActiveFiled(
                            '{}.{} 没有is_active字段, 请检查程序逻辑'.format(
                                manager.model.__module__,
                                manager.model.__class__.__name__
                    ))
        instance.is_active = False
        instance.save()
```


​    
​    
```python
class DeleteForeignObjectRelModelMixin(object):
    '''删除一个对象，如果他已有外键关联则抛出异常'''
    @POST_OR_GET('force_delete', type='bool', default=False)
    def destroy(self, request, force_delete, *args, **kwargs):
        instance = self.get_object()
        if not force_delete:
            rel_fileds = [f for f in instance._meta.get_fields() if isinstance(f, ForeignObjectRel)]
            links = [f.get_accessor_name() for f in rel_fileds]
            for link in links:
                manager = getattr(instance, link, None)
                if not manager:
                    continue      
                # one to one
                if isinstance(manager, models.Model):
                    if hasattr(manager, 'is_active') and manager.is_active:
                        raise ForeignObjectRelDeleteError(u'{} 上有关联数据'.format(link))
                else:
                    try:
                        manager.model._meta.get_field('is_active')
                        if manager.filter(is_active=True).count():
                            raise ForeignObjectRelDeleteError(u'{} 上有关联数据'.format(link))
                    except FieldDoesNotExist as ex:
                        if manager.count():
                            raise ForeignObjectRelDeleteError(u'{} 上有关联数据'.format(link))
        self.perform_destroy(instance)
        return Response(status=status.HTTP_204_NO_CONTENT)                                  
```

### 怎么用呢

例如你在admin里面先新建一个group，然后把group上面勾上对应的权限，然后用户的group那儿对应的把组加上即可（当然你可以自己写对应的代码而不用admin）。  
不太建议直接在user上面加上对应的权限，虽然完全没有问题
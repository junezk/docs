# Django REST framework的使用总结

- 在序列化与反序列化时，虽然操作的数据不尽相同，但是执行的过程却是相似的，也就是说这部分代码是可以复用简化编写的；
- 在开发REST API的视图中，虽然每个视图具体操作的数据不同，但增、删、改、查的实现流程基本套路化，所以这部分代码也是可以复用简化编写；

Django REST framework 框架是一个用于构建Web API 的强大而又灵活的工具。通常简称为DRF框架 或 REST framework。DRF框架是建立在Django框架基础之上，由Tom Christie大牛二次开发的开源项目。

- [官方文档](http://www.django-rest-framework.org/) 需科学上网。
- [Github源码](https://github.com/encode/django-rest-framework/tree/master)

### 1.特点

- 提供了定义序列化器Serializer的方法，可以快速根据 Django ORM 或者其它库自动序列化/反序列化；
- 提供了丰富的类视图、Mixin扩展类，简化视图的编写；
- 丰富的定制层级：函数视图、类视图、视图集合到自动生成 API，满足各种需要；
- 多种身份认证和权限认证方式的支持；
- 内置了限流系统；
- 直观的 API web 界面；
- 可扩展性，插件丰富

### 2.环境安装与配置

```sh
pip install djangorestframework  #安装
```

DRF需要以下依赖：

- Python (2.7, 3.2, 3.3, 3.4, 3.5, 3.6)
- Django (1.11, 2.0)

**DRF是以Django扩展应用的方式提供的，所以我们可以直接利用已有的Django环境而无需从新创建。（若没有Django环境，需要先创建环境安装Django）**

在`django`配置文件**settings.py**(以具体项目为准)的**INSTALLED_APPS**中添加'rest_framework'。

```python
INSTALLED_APPS = [    ...    'rest_framework',]
```

## 3.序列化器

**序列化器的作用：**

1. **进行数据的校验**
2. **对数据对象进行转换**

### 3.1Serializer

DRF中的`Serializer`使用类来定义，须继承自`rest_framework.serializers.Serializer`

先建一个数据库模型类BookInfo.

```python
class BookInfo(models.Model):
    btitle = models.CharField(max_length=20, verbose_name='名称')
    bpub_date = models.DateField(verbose_name='发布日期', null=True)
    bread = models.IntegerField(default=0, verbose_name='阅读量')
    bcomment = models.IntegerField(default=0, verbose_name='评论量')
    image = models.ImageField(upload_to='booktest', verbose_name='图片', null=True)
```

想要为这个模型类提供一个序列化器，可以定义如下：

```python
class BookInfoSerializer(serializers.Serializer):
    """图书数据序列化器"""
    id = serializers.IntegerField( read_only=True)
    btitle = serializers.CharField( max_length=20)
    bpub_date = serializers.DateField( )
    bread = serializers.IntegerField()
    bcomment = serializers.IntegerField( )
    image = serializers.ImageField( )
```

read_only指定只参与序列化，max_length限定字符串最大长度。字段里不传参数序列化与反序列化都参与。

**注意：`serializer`不是只能为数据库模型类定义，也可以为非数据库模型类的数据定义。**`serializer`是独立于数据库之外的存在。

#### 字段与选项

**常用字段类型**：

| 字段                    | 字段构造方式                                                 |
| ----------------------- | ------------------------------------------------------------ |
| **BooleanField**        | BooleanField()                                               |
| **NullBooleanField**    | NullBooleanField()，# null,True,False                        |
| **CharField**           | CharField(max_length=None, min_length=None, allow_blank=False, trim_whitespace=True) |
| **EmailField**          | EmailField(max_length=None, min_length=None, allow_blank=False) |
| **RegexField**          | RegexField(regex, max_length=None, min_length=None, allow_blank=False) |
| **SlugField**           | SlugField(maxlength=50, min_length=None, allow_blank=False) 正则字段，验证正则模式 [a-zA-Z0-9-]+ |
| **URLField**            | URLField(max_length=200, min_length=None, allow_blank=False) |
| **UUIDField**           | UUIDField(format='hex_verbose') format: 1) `'hex_verbose'` 如`"5ce0e9a5-5ffa-654b-cee0-1238041fb31a"` 2） `'hex'` 如 `"5ce0e9a55ffa654bcee01238041fb31a"` 3）`'int'` - 如: `"123456789012312313134124512351145145114"` 4）`'urn'` 如: `"urn:uuid:5ce0e9a5-5ffa-654b-cee0-1238041fb31a"` |
| **IPAddressField**      | IPAddressField(protocol='both', unpack_ipv4=False, **options) |
| **IntegerField**        | IntegerField(max_value=None, min_value=None)                 |
| **FloatField**          | FloatField(max_value=None, min_value=None)                   |
| **DecimalField**        | DecimalField(max_digits, decimal_places, coerce_to_string=None, max_value=None, min_value=None) max_digits: 最多位数 decimal_palces: 小数点位置 |
| **DateTimeField**       | DateTimeField(format=api_settings.DATETIME_FORMAT, input_formats=None) |
| **DateField**           | DateField(format=api_settings.DATE_FORMAT, input_formats=None) |
| **TimeField**           | TimeField(format=api_settings.TIME_FORMAT, input_formats=None) |
| **DurationField**       | DurationField()                                              |
| **ChoiceField**         | ChoiceField(choices) # choices与Django的用法相同             |
| **MultipleChoiceField** | MultipleChoiceField(choices)                                 |
| **FileField**           | FileField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL) |
| **ImageField**          | ImageField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL) |
| **ListField**           | ListField(child=, min_length=None, max_length=None)          |
| **DictField**           | DictField(child=)                                            |

#### 通用参数说明：

| 参数名称           | 说明                                          |
| ------------------ | --------------------------------------------- |
| **read_only**      | 表明该字段仅用于序列化输出，默认False         |
| **write_only**     | 表明该字段仅用于反序列化输入，默认False       |
| **required**       | 表明该字段在反序列化时必须输入，默认True      |
| **default**        | 反序列化时使用的默认值                        |
| **allow_null**     | 表明该字段是否允许传入None，默认False         |
| **validators**     | 该字段使用的验证器                            |
| **error_messages** | 包含错误编号与错误信息的字典                  |
| **label**          | 用于HTML展示API页面时，显示的字段名称         |
| **help_text**      | 用于HTML展示API页面时，显示的字段帮助提示信息 |

### 3.2 序列化使用

1） 先查询出一个图书对象或所有图书

```python
from book.models import BookInfo

book = BookInfo.objects.get(id=1)
# books = BookInfo.objects.all()
```

2） 构造序列化器对象

```python
from book.serializers import BookInfoSerializer

serializer = BookInfoSerializer(book)
```

3）获取序列化数据

通过data属性可以获取序列化后的数据

```python
serializer.data
```

4）如果要被序列化的是包含多条数据的查询集`QuerySet`，可以通过添加**many=True**参数补充说明，后面这句**必加**，不然会报错。

```python
serializer = BookInfoSerializer(books, many=True)
serializer.data
```

完整代码：

```python
import json
from django.http import JsonResponse
from django.views import View
from book.models import BookInfo


class BooksView(View):

    def get(self, request):
        '''
        :param request:
        :return: 查询所有图书
        '''
        books = BookInfo.objects.all()
        ser = BookInfoSerializer(books, many=True)
        data = ser.data
        return JsonResponse(data, safe=False)
```

#### 关联对象嵌套序列化

`BookInfo`的关系模型类`HeroInfo`定义如下：

```python
# 定义英雄模型类HeroInfo
class HeroInfo(models.Model):
    GENDER_CHOICES = (
        (0, 'female'),
        (1, 'male')
    )
    hname = models.CharField(max_length=20, verbose_name='名称')
    hgender = models.SmallIntegerField(choices=GENDER_CHOICES, default=0, verbose_name='性别')
    hcomment = models.CharField(max_length=200, null=True, verbose_name='描述信息')
    hbook = models.ForeignKey(BookInfo, related_name='heroes', on_delete=models.CASCADE, verbose_name='图书')  # 外键
    is_delete = models.BooleanField(default=False, verbose_name='逻辑删除')

    class Meta:
        db_table = 'tb_heros'
        verbose_name = '英雄'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.hname
```

`BookInfoSerializer`序列化器中添加下面字段及其说明：

##### 1） PrimaryKeyRelatedField

此字段将被序列化为关联对象的主键。属性使用related_name的值，如果没有则使用关联类小写_set(heroinfo_set)

```python
heroes = serializers.PrimaryKeyRelatedField( read_only=True)
```

指明字段时需要包含read_only=True或者queryset参数：

- 包含read_only=True参数时，该字段将不能用作反序列化使用

```python
from book.serializers import HeroInfoSerializer
from book.models import HeroInfo

hero = HeroInfo.objects.get(id=6)
serializer = HeroInfoSerializer(hero)
serializer.data
```

##### 2) StringRelatedField

此字段将被序列化为关联对象的字符串表示方式（即`heroinfo`模型类`__str__`方法的返回值）

```python
heroes = serializers.StringRelatedField()
```

##### 3）使用关联对象的序列化器

```python
heroes = HeroInfoSerializer(many=True)  # 根据指定的HeroSerialzier序列化中指定的字段返回
```

注意`HeroInfoSerializer`要先在`BookInfoSerializer`定义出来才能使用。

### 3.3 反序列化使用

- 使用序列化器进行反序列化时，需要对数据进行验证后，才能获取验证成功的数据或保存成模型类对象；
- 在获取反序列化的数据前，必须调用**is_valid()**方法进行验证，验证成功返回True，否则返回False；
- 验证失败，可以通过序列化器对象的**errors**属性获取错误信息，返回字典，包含了字段和字段的错误。如果是非字段错误，可以通过修改REST framework配置中的**NON_FIELD_ERRORS_KEY**来控制错误字典中的键名；
- 验证成功，可以通过序列化器对象的**validated_data**属性获取数据。

在定义序列化器时，指明每个字段的序列化类型和选项参数，本身就是一种验证行为。通过构造序列化器对象，并将要反序列化的数据传递给data构造参数，进而进行验证。

```python
class BookView(View):
    def put(self, request, pk):
        data = json.loads(request.body.decode())
        # btitle = data.get('btitle')
        try:
            book = BookInfo.objects.get(pk=pk)
        except Exception:
            return JsonResponse({'errors': '未找到该书'})

        ser = BookInfoSerializer(book, data=data)
        ser.is_valid(raise_exception=True)
       

```

序列化器中还提供了下面的验证方式：

- 验证单一字段

```python
class BookInfoSerializer(serializers.Serializer):
   
    ...
    
        def validated_btitle(self, value):
        if value == 'python':
            raise serializers.ValidationError('书名python错误')
        return value
```

- 验证多个字段

  ```python
  class BookInfoSerializer(serializers.Serializer):
      """图书数据序列化器"""
      ...
  
      def validate(self, attrs):
          bread = attrs['bread']
          bcomment = attrs['bcomment']
          if bread < bcomment:
              raise serializers.ValidationError('阅读量小于评论量')
          return attrs
  ```

#### 保存数据

如果在验证成功后，想要基于validated_data类属性完成数据对象的创建，可以通过实现create()和update()两个方法来实现。

```python
class BookInfoSerializer(serializers.Serializer):
    """图书数据序列化器"""
    ...

    def create(self, validated_data):
        """新建"""
        return BookInfo(**validated_data)

    def update(self, instance, validated_data):
        """更新，instance为要更新的对象实例"""
        instance.btitle = validated_data.get('btitle', instance.btitle)
        instance.bpub_date = validated_data.get('bpub_date', instance.bpub_date)
        instance.bread = validated_data.get('bread', instance.bread)
        instance.bcomment = validated_data.get('bcomment', instance.bcomment)
        return instance
```

如果需要在返回数据对象的时候，也将数据保存到数据库中，则可以进行如下修改

```python
class BookInfoSerializer(serializers.Serializer):
    """图书数据序列化器"""
    ...

    def create(self, validated_data):
        """新建"""
        return BookInfo.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """更新，instance为要更新的对象实例"""
        instance.btitle = validated_data.get('btitle', instance.btitle)
        instance.bpub_date = validated_data.get('bpub_date', instance.bpub_date)
        instance.bread = validated_data.get('bread', instance.bread)
        instance.bcomment = validated_data.get('bcomment', instance.bcomment)
        instance.save()
        return instance
```

实现了上述两个方法后，在反序列化数据的时候，就可以通过save()方法返回一个数据对象实例了。

```python
class BookView(View):
    def put(self, request, pk):
   
        data = json.loads(request.body.decode())
        # btitle = data.get('btitle')
        try:
            book = BookInfo.objects.get(pk=pk)
        except Exception:
            return JsonResponse({'errors': '未找到该书'})

        ser = BookSerializer(book, data=data)
        ser.is_valid(raise_exception=True)
        ser.save() #返回数据对象实例
        return JsonResponse(ser.data)
```

**如果创建序列化器对象的时候，没有传递instance实例，则调用save()方法的时候，create()被调用，相反，如果传递了instance实例，则调用save()方法的时候，update()被调用。**

### 3.4 模型类序列化器ModelSerializer

如果我们想要使用序列化器对应的是Django的模型类，DRF为我们提供了`ModelSerializer`模型类序列化器来帮助我们快速创建一个`Serializer`类。

ModelSerializer与常规的Serializer相同，但提供了：

- 基于模型类自动生成一系列字段
- 基于模型类自动为Serializer生成validators，比如unique_together
- 包含默认的create()和update()的实现

#### 定义

要使用要继承`serializers.ModelSerializer`

比如我们创建一个BookInfoSerializer：

```python
class BookInfoSerializer(serializers.ModelSerializer):
    """图书数据序列化器"""
    class Meta:
        model = BookInfo
        fields = '__all__' # 返回所有字段
```

- model 指明参照哪个模型类
- fields 指明为模型类的哪些字段生成

我们可以在`django`环境中执行python manage.py shell命令来查看自动生成的`BookInfoSerializer`的具体实现。

```python
>>> from booktest.serializers import BookInfoSerializer
>>> serializer = BookInfoSerializer()
>>> serializer
BookInfoSerializer():
    id = IntegerField(label='ID', read_only=True)
    btitle = CharField(label='名称', max_length=20)
    bpub_date = DateField(allow_null=True, label='发布日期', required=False)
    bread = IntegerField(label='阅读量', max_value=2147483647, min_value=-2147483648, required=False)
    bcomment = IntegerField(label='评论量', max_value=2147483647, min_value=-2147483648, required=False)
    image = ImageField(allow_null=True, label='图片', max_length=100, required=False)
```

#### 指定字段

1) 使用**`fields`**来明确字段，`__all__`表名包含所有字段，也可以写明具体哪些字段，如

```python
class BookInfoSerializer(serializers.ModelSerializer):
    """图书数据序列化器"""
    class Meta:
        model = BookInfo # 指定根据哪个模型类来序列化
        fields = ('id', 'btitle', 'bpub_date')
```

2) 使用**`exclude`**可以明确排除掉哪些字段

```python
class BookInfoSerializer(serializers.ModelSerializer):
    """图书数据序列化器"""
    class Meta:
        model = BookInfo
        exclude = ('bread',)
```

3) 显示指明字段，如：

```python
class HeroInfoSerializer(serializers.ModelSerializer):
    hbook = BookInfoSerializer()

    class Meta:
        model = HeroInfo
        fields = ('id', 'hname', 'hgender', 'hcomment', 'hbook')
```

4) 指明只读字段

可以通过**read_only_fields**指明只读字段，即仅用于序列化输出的字段

```python
class BookInfoSerializer(serializers.ModelSerializer):
    """图书数据序列化器"""
    class Meta:
        model = BookInfo
        fields = ('id', 'btitle', 'bpub_date'， 'bread', 'bcomment')
        read_only_fields = ('id', 'bread', 'bcomment')
```

#### 添加额外参数

我们可以使用**`extra_kwargs`**参数为`ModelSerializer`添加或修改原有的选项参数

```python
class BookInfoSerializer(serializers.ModelSerializer):
    
    """图书数据序列化器"""
    class Meta:
        model = BookInfo
        fields = ('id', 'btitle', 'bpub_date', 'bread', 'bcomment')
        extra_kwargs = {
            'bread': {'min_value': 0, 'required': True},
            'bcomment': {'min_value': 0, 'required': True},
        }

# BookInfoSerializer():
#    id = IntegerField(label='ID', read_only=True)
#    btitle = CharField(label='名称', max_length=20)
#    bpub_date = DateField(allow_null=True, label='发布日期', required=False)
#    bread = IntegerField(label='阅读量', max_value=2147483647, min_value=0, required=True)
#    bcomment = IntegerField(label='评论量', max_value=2147483647, min_value=0, required=True)

```

## 4.视图

### 4.1 Request

`REST framework` 传入DRF封装后视图的`request`对象不再是`Django`默认的`HttpRequest`对象，而是`REST framework`提供的扩展了`HttpRequest`类的**`Request`**类的对象。

REST framework 提供了**Parser**解析器，在接收到请求后会自动根据Content-Type指明的请求数据类型（如JSON、表单等）将请求数据进行parse解析，解析为类字典对象保存到**Request**对象中。

**Request对象的数据是自动根据前端发送数据的格式进行解析之后的结果。**

无论前端发送的哪种格式的数据，我们都可以以统一的方式读取数据。

#### data

`request.data` 返回解析之后的请求体（表单、json）数据。类似于Django中标准的`request.POST`和 `request.FILES`属性，但提供如下特性：

- 包含了**解析之后**的文件和非文件数据
- 包含了对POST、PUT、PATCH请求方式解析后的数据
- 利用了REST framework的parsers解析器，不仅支持表单类型数据，也支持JSON数据

#### query_params

`request.query_params`与Django标准的`request.GET`相同，用于获取查询字符串，只是更换了更正确的名称而已。

### 4.2 Response

```python
rest_framework.response.Response
```

REST framework提供了一个响应类`Response`，使用该类构造响应对象时，响应的具体数据内容会被转换（render渲染）成符合前端需求的类型。

#### 构造方式

```python
Response(data, status=None, template_name=None, headers=None, content_type=None)
```

`data`数据不要是render处理之后的数据，只需传递python的内建类型数据即可，REST framework会使用`renderer`渲染器处理`data`。

`data`不能是复杂结构的数据，如Django的模型类对象，对于这样的数据我们可以使用`Serializer`序列化器序列化处理后（转为了Python字典类型）再传递给`data`参数。

参数说明:

- `data`: 为响应准备的序列化处理后的数据；
- `status`: 状态码，默认200；
- `template_name`: 模板名称，如果使用`HTMLRenderer` 时需指明；
- `headers`: 用于存放响应头信息的字典；
- `content_type`: 响应数据的Content-Type，通常此参数无需传递，REST framework会根据前端所需类型数据来设置该参数。

#### 状态码

为了方便设置状态码，`REST framewrok`在`rest_framework.status`模块中提供了常用状态码常量。

这样前面的获取与更新单一图书可改写为如下形式:

```python
from rest_framework.views import APIView, View
from rest_framework.response import Response
from book.models import BookInfo
from book.serializers import BookInfoSerializer

class BooksView(APIView):
    
	def get(self, request, pk):
        """
         获取单一图书
        :param request:
        :return:
        """
        # 查询数据
        try:
            book = BookInfo.objects.get(id=pk)
        except:
            return Response({"error": '错误的id'}, status=400)

        # 返回结果

        ser = BookSerializer(book)

        return Response(ser.data)

    def put(self, request, pk):
            """
             更新单一图书
            :param request:
            :return:
            """
            # 1、获取前端数据
            data = request.data
            # 2、验证数据
            book = BookInfo.objects.get(id=pk)
            ser = BookInfoSerializer(book, data=data)
            ser.is_valid(raise_exception=True)
            # 3、更新数据
            ser.save()
            # 4、返回结果
            return Response(ser.data)
```

如果返回所有图书列表，不用再像以前`JsonResponse`中需指定`safe=False`了。

### 4.3 APIView

```
rest_framework.views.APIView
```

`APIView`是REST framework提供的所有视图的基类，继承自Django的`View`父类。

`APIView`与`View`的不同之处在于：

- 传入到视图方法中的是REST framework的`Request`对象，而不是Django的`HttpRequeset`对象；
- 视图方法可以返回REST framework的`Response`对象，视图会为响应数据设置（render）符合前端要求的格式；
- 任何`APIException`异常都会被捕获到，并且处理成合适的响应信息；
- 在进行dispatch()分发前，会对请求进行身份认证、权限检查、流量控制。

##### 支持定义的属性：

- **authentication_classes** 列表或元祖，身份认证类
- **permissoin_classes** 列表或元祖，权限检查类
- **throttle_classes** 列表或元祖，流量控制类

在`APIView`中仍以常规的类视图定义方法来实现get() 、post() 或者其他请求方式的方法。

```python
from rest_framework.views import APIView
from rest_framework.response import Response

# url(r'^books/$', views.BookListView.as_view()),
class BookListView(APIView):
    def get(self, request):
        books = BookInfo.objects.all()
        serializer = BookInfoSerializer(books, many=True)
        return Response(serializer.data)
```

### 4.4 GenericAPIView

`rest_framework.generics.GenericAPIView`继承自`APIVIew`，增加了对于列表视图和详情视图可能用到的通用支持方法。通常使用时，可搭配一个或多个Mixin扩展类。

##### 支持定义的属性：

- 列表视图与详情视图通用：
  - **queryset** 列表视图的查询集
  - **serializer_class** 视图使用的序列化器
- 列表视图使用：
  - **pagination_class** 分页控制类
  - **filter_backends** 过滤控制后端
- 详情页视图使用：
  - **lookup_field** 查询单一数据库对象时使用的条件字段，默认为'`pk`'
  - **lookup_url_kwarg** 查询单一数据时URL中的参数关键字名称，默认与**look_field**相同

##### 提供的方法：

- 列表视图与详情视图通用：

  - **get_queryset(self)**

    返回视图使用的查询集，是列表视图与详情视图获取数据的基础，默认返回`queryset`属性，可以重写，例如：

    ```python
    def get_queryset(self):
        user = self.request.user
        return user.accounts.all()
    ```

  - **get_serializer_class(self)**

    返回序列化器类，默认返回`serializer_class`，可以重写，例如：

    ```python
    def get_serializer_class(self):
        if self.request.user.is_staff:
            return FullAccountSerializer
        return BasicAccountSerializer
    ```

  - ##### get_serializer(self, *args, **kwargs)

    返回序列化器对象，被其他视图或扩展类使用，如果我们在视图中想要获取序列化器对象，可以直接调用此方法。

    **注意，在提供序列化器对象的时候，REST framework会向对象的context属性补充三个数据：request、format、view，这三个数据对象可以在定义序列化器时使用。**

- 详情视图使用：

  - **get_object(self)** 返回详情视图所需的模型类数据对象，默认使用`lookup_field`参数来过滤queryset。 在试图中可以调用该方法获取详情信息的模型类对象。

    **若详情访问的模型类对象不存在，会返回404。**

    **该方法会默认使用`APIView`提供的check_object_permissions方法检查当前对象是否有权限被访问。**

```python
class BooksView(GenericAPIView):
    queryset = BookInfo.objects.all() #指定查询集
    serializer_class = BookInfoSerializer # 指定序列化器

    def get(self, request):
        '''查询所有图书'''
        books = self.get_queryset() 获取查询集
        # book = self.get_object() 获取单一数据对象
        serializer = self.get_serializer(books, many = True) #获取序列化器
        return Response(serializer.data)
    
      def post(self, request):
        """
         保存图书
        """
        # 1、获取前端数据
        data = request.data
        # 2、验证数据
        ser = self.get_serializer(data=data)
        ser.is_valid(raise_exception=True)
        # 3、保存数据
        ser.save()
        # 4、返回结果
        return Response(ser.data)
```

两个类关系图如下：

![img](Django REST framework的使用总结.assets/16b60b76a3f8ac10)

## 5.五个扩展类

### 5.1 ListModelMixin

列表视图扩展类，提供`list(request, *args, **kwargs)`方法快速实现列表视图，返回200状态码。

```python
from rest_framework.mixins import ListModelMixin

class BooksView(ListModelMixin, GenericAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    def get(self, request):
        return self.list(request)
```

### 5.2 CreateModelMixin

创建视图扩展类，提供`create(request, *args, **kwargs)`方法快速实现创建资源的视图，成功返回201状态码。如果序列化器对前端发送的数据验证失败，返回400错误。

```python
from rest_framework.mixins import CreateModelMixin

class BooksView(CreateModelMixin, GenericAPIView):    
    serializer_class = BookInfoSerializer

    def post(self, request):
        return self.create(request)
```

### 5.3 RetrieveModelMixin

详情视图扩展类，提供`retrieve(request, *args, **kwargs)`方法，可以快速实现返回一个存在的数据对象。如果存在，返回200， 否则返回404。

```python
class BooksView(RetrieveModelMixin, GenericAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    def get(self, request, pk):
        return self.retrieve(request)
```

### 5.4 UpdateModelMixin

更新视图扩展类，提供`update(request, *args, **kwargs)`方法，可以快速实现更新一个存在的数据对象。同时也提供`partial_update(request, *args, **kwargs)`方法，可以实现局部更新。

成功返回200，序列化器校验数据失败时，返回400错误。

### 5.5 DestroyModelMixin

删除视图扩展类，提供`destroy(request, *args, **kwargs)`方法，可以快速实现删除一个存在的数据对象。成功返回204，不存在返回404。

![img](Django REST framework的使用总结.assets/16b60de1e48b3897)

`GenericAPIView`配合扩展类使用举例如下，它为我们省略了很多代码。

```python
from rest_framework.views import APIView, View
from rest_framework.generics import GenericAPIView
from rest_framework.mixins import CreateModelMixin, UpdateModelMixin, DestroyModelMixin, RetrieveModelMixin, ListModelMixin
from rest_framework.response import Response
from book.models import BookInfo
from book.serializers import BookInfoSerializer


class BooksView(CreateModelMixin, ListModelMixin, GenericAPIView):
    serializer_class = BookInfoSerializer  # 指定序列化器
    queryset = BookInfo.objects.all()  # 指定数据对象查询集

    def get(self, request):
        '''获取所有图书'''
        return self.list(request)

    def post(self, request):
        '''保存图书'''
        return self.create(request)


class BookView(GenericAPIView, RetrieveModelMixin, UpdateModelMixin):
    serializer_class = BookSerializer  # 指定序列化器
    queryset = BookInfo.objects.all()  # 指定数据对象查询集

    def get(self, request, pk):
        '''查询单一图书'''
        return self.retrieve(request, pk)

    def put(self, request, pk):
        '''修改图书'''
        return self.update(request.pk)
    def delete(self, request, pk):
        '''删除图书'''
        return self.destroy(request,pk)
```

### 5.6 可用子类视图

#### CreateAPIView

提供 post 方法

继承自： GenericAPIView、CreateModelMixin

#### ListAPIView

提供 get 方法

继承自：GenericAPIView、ListModelMixin

#### RetireveAPIView

提供 get 方法

继承自: GenericAPIView、RetrieveModelMixin

#### DestoryAPIView

提供 delete 方法

继承自：GenericAPIView、DestoryModelMixin

#### UpdateAPIView

提供 put 和 patch 方法

继承自：GenericAPIView、UpdateModelMixin

#### RetrieveUpdateAPIView

提供 get、put、patch方法

继承自： GenericAPIView、RetrieveModelMixin、UpdateModelMixin

#### RetrieveUpdateDestoryAPIView

提供 get、put、patch、delete方法

继承自：GenericAPIView、RetrieveModelMixin、UpdateModelMixin、DestoryModelMixin

使用举例：

```python
from rest_framework.generics import ListCreateAPIView, RetrieveUpdateDestroyAPIView
from book.models import BookInfo
from book.serializers import BookInfoSerializer


class BooksView(ListCreateAPIView):
    serializer_class = BookSerializer  # 指定序列化器
    queryset = BookInfo.objects.all()  # 指定数据对象查询集


class BookView(RetrieveUpdateDestroyAPIView):
    serializer_class = BookInfoSerializer  # 指定序列化器
    queryset = BookInfo.objects.all()  # 指定数据对象查询集
```

使用上面的代码就能完成所有的增删改查逻辑，大大简化了视图层里的代码，提高了开发效率，我们只需花时间写好序列化器里的验证逻辑就好了。

## 6.视图集

### 6.1 ViewSet

使用视图集`ViewSet`，可以将一系列逻辑相关的动作放到一个类中：

- list() 提供一组数据
- retrieve() 提供单个数据
- create() 创建数据
- update() 保存数据
- destory() 删除数据

ViewSet视图集类不再实现get()、post()等方法，而是实现动作 **action** 如 list() 、create() 等。

视图集只在使用as_view()方法的时候，才会将**action**动作与具体请求方式对应上。如：

```python
class BookInfoViewSet(viewsets.ViewSet):

    def list(self, request):
        ...

    def retrieve(self, request, pk=None):
        ...
```

在设置路由时，我们可以如下操作

```python
urlpatterns = [
    url(r'^books/$', BookInfoViewSet.as_view({'get':'list'}),
    url(r'^books/(?P<pk>\d+)/$', BookInfoViewSet.as_view({'get': 'retrieve'})
]
```

#### action属性

在**视图集**中，我们可以通过action对象属性来获取当前请求视图集时的action动作是哪个。

```python
def get_serializer_class(self):
    if self.action == 'create':
        return OrderCommitSerializer
    else:
        return OrderDataSerializer
```

### 6.2 常用视图集父类

#### ViewSet

继承自`APIView`，作用也与APIView基本类似，提供了身份认证、权限校验、流量管理等。

在ViewSet中，没有提供任何动作action方法，需要我们自己实现action方法。

#### GenericViewSet

继承自`GenericAPIView`，作用也与GenericAPIVIew类似，提供了get_object、get_queryset等方法便于列表视图与详情信息视图的开发。

#### ModelViewSet

继承自`GenericAPIVIew`，同时包括了ListModelMixin、RetrieveModelMixin、CreateModelMixin、UpdateModelMixin、DestoryModelMixin。所以所有增删改查功能都具备。

```python
#  路由层
     url(r'^books/$', view_set.BooksModelViewSet.as_view({'get': 'list', 'post': 'create'})),
     url(r'^books/(?P<pk>\d+)/$', view_set.BooksModelViewSet.as_view({'get': 'retrieve','put':'update','delete':'destroy'})),
    
 # 视图层
 class BooksModelViewSet(ModelViewSet):


    serializer_class = BookInfoSerializer
    queryset = BookInfo.objects.all()
```

`ModelViewSet`虽好，但有时也要按具体的逻辑考虑，例如注册，只需要保存数据到数据库，没必要把所有功能都暴露出去，也造成资源的浪费。

#### ReadOnlyModelViewSet

继承自`GenericAPIVIew`，同时包括了ListModelMixin、RetrieveModelMixin。

![img](Django REST framework的使用总结.assets/16b60df65529979a)

### 6.3 定义附加action

在视图集中，除了上述默认的方法动作外，还可以添加自定义动作。

添加自定义动作需要使用`rest_framework.decorators.action`装饰器。

以action装饰器装饰的方法名会作为action动作名，与list、retrieve等同。

action装饰器可以接收两个参数：

- **methods**: 该action支持的请求方式，列表传递
- detail: 表示是action中要处理的是否是视图资源的对象（即是否通过url路径获取主键）
  - True 表示使用通过URL获取的主键对应的数据对象
  -  False 表示不使用URL获取主键

### 6.4 路由Routers

对于视图集ViewSet，我们除了可以自己手动指明请求方式与动作action之间的对应关系外，还可以使用Routers来帮助我们快速实现路由信息。

REST framework提供了两个router

- **`SimpleRouter`**
- **`DefaultRouter`**

#### 使用方法

1） 创建router对象，并注册视图集，例如

```python
from rest_framework import routers

router = routers.SimpleRouter()
router.register(r'books', BookInfoViewSet, base_name='book')
```

register(prefix, viewset, base_name)

- prefix 该视图集的路由前缀
- viewset 视图集
- base_name 路由名称的前缀

如上述代码会形成的路由如下：

```python
^books/$    name: book-list
^books/{pk}/$   name: book-detail
```

2）添加路由数据

可以有两种方式：

```python
urlpatterns = [    ...]
urlpatterns += router.urls
```

或

```
urlpatterns = [    ...,    url(r'^', include(router.urls))]
```

这种方式只会生成视图集扩展类里面定义的方法，对于自定义的方法不能生成。

#### 视图集中包含附加action

导入`rest_framework.decorators.action`装饰器可解决上面问题。

```python
...
from rest_framework.decorators import action

class BookInfoViewSet(mixins.ListModelMixin, mixins.RetrieveModelMixin, GenericViewSet):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    @action(methods=['get'], detail=False) #detail = False不生成正则匹配
    def latest(self, request, pk):
        ...

    @action(methods=['put'], detail=True)#detail = True生成正则匹配
    def read(self, request, pk):
        ...
```

此视图集会形成的路由：

```python
^books/latest/$    name: book-latest
^books/{pk}/read/$  name: book-read
```

**要使用自动生成路由的方法，视图必须继承自视图集。**

除了`SimpleRouter`,还有`DefaultRouter`，区别在于后者能生成首页，而前者不能。

使用视图集只能处理前端通过request发过来的数据是表单或json数据的类型。

## 7 其他功能

### 7.1 认证Authentication

可以在配置文件中配置全局默认的认证方案

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',   # 基本表单认证
        'rest_framework.authentication.SessionAuthentication',  # session认证
    )
}
```

也可以在视图中通过设置authentication_classess属性来设置

```python
from rest_framework.authentication import SessionAuthentication, BasicAuthentication
from rest_framework.views import APIView

class ExampleView(APIView):
    authentication_classes = (SessionAuthentication, BasicAuthentication)
    ...
```

认证失败会有两种可能的返回值：

- 401 Unauthorized 未认证
- 403 Permission Denied 权限被禁止

### 7.2 权限Permissions

权限控制可以限制用户对于视图的访问和对于具体数据对象的访问。

- 在执行视图的dispatch()方法前，会先进行视图访问权限的判断
- 在通过get_object()获取具体对象时，会进行对象访问权限的判断

#### 使用

可以在配置文件中设置默认的权限管理类，如

```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated', # 授权访问
    )
}

```

如果未指明，则采用如下默认配置

```python
'DEFAULT_PERMISSION_CLASSES': (
   'rest_framework.permissions.AllowAny', 
)
```

也可以在具体的视图中通过permission_classes属性来设置，如

```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.viewsets import ModelViewSet

BooksModelViewSet(ModelViewSet):
    permission_classes = (IsAuthenticated,) #登录授权用户
    ...
```

![img](Django REST framework的使用总结.assets/16b60d3c617a5a44)

如果认证失败，返回如下信息：

![img](Django REST framework的使用总结.assets/16b60d572991dff1)

#### 提供的权限

- AllowAny 允许所有用户
- IsAuthenticated 仅通过认证的用户
- IsAdminUser 仅管理员用户
- IsAuthenticatedOrReadOnly 认证的用户可以完全操作，否则只能get读取

```python
from rest_framework.authentication import SessionAuthentication
from rest_framework.permissions import IsAuthenticated
from rest_framework.generics import RetrieveAPIView

class BookListView(RetrieveAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer
    authentication_classes = [SessionAuthentication]
    permission_classes = [IsAuthenticated]
```

### 7.3 限流Throttling

可以对接口访问的频次进行限制，以减轻服务器压力，**主要用于反爬虫**。

#### 使用

可以在配置文件中，使用`DEFAULT_THROTTLE_CLASSES` 和 `DEFAULT_THROTTLE_RATES`进行全局配置，

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ),
    'DEFAULT_THROTTLE_RATES': {
        'anon': '4/day', # 匿名用户每天访问4次
        'user': '100/day'  # 登录用户访问100次
    }
}
```

`DEFAULT_THROTTLE_RATES` 可以使用 `second`, `minute`, `hour` 或`day`来指明周期。

也可以在具体视图中通过throttle_classess属性来配置，如

```python
from rest_framework.throttling import UserRateThrottle
from rest_framework.views import APIView

class BookListView(APIView):    throttle_classes = (UserRateThrottle,)
    ...
```

![img](Django REST framework的使用总结.assets/16b60cf83bc9d0f4)

#### 可选限流类

1） `AnonRateThrottle`

限制所有匿名未认证用户，使用IP区分用户。

使用`DEFAULT_THROTTLE_RATES['anon']` 来设置频次

2）`UserRateThrottle`

限制认证用户，使用User id 来区分。

使用`DEFAULT_THROTTLE_RATES['user']` 来设置频次

3）`ScopedRateThrottle`

限制用户对于每个视图的访问频次，使用ip或user id。

```python
class ContactListView(APIView):
    throttle_scope = 'contacts'
    ...

class ContactDetailView(APIView):
    throttle_scope = 'contacts'
    ...

class UploadView(APIView):
    throttle_scope = 'uploads'
    ...
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.ScopedRateThrottle',
    ),
    'DEFAULT_THROTTLE_RATES': {
        'contacts': '1000/day',
        'uploads': '20/day'
    }
}
```

```python
from rest_framework.authentication import SessionAuthentication
from rest_framework.permissions import IsAuthenticated
from rest_framework.generics import RetrieveAPIView
from rest_framework.throttling import UserRateThrottle

class BooksView(RetrieveAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer
    authentication_classes = [SessionAuthentication]
    permission_classes = [IsAuthenticated]
    throttle_classes = (UserRateThrottle,)
```

### 7.4 过滤Filtering

对于列表数据可能需要根据字段进行过滤，我们可以通过添加django-fitlter扩展来增强支持，实现了`ListModelMixin`扩展类及子类的视图类才能使用。

```
pip install django-filter
```

在配置文件中增加过滤后端的设置：

```python
INSTALLED_APPS = [
    ...
    'django_filters',  # 需要注册应用，
]

REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ('django_filters.rest_framework.DjangoFilterBackend',)
}
```

在视图中添加`filter_fields`属性，指定可以过滤的字段

```python
class BookListView(ListAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer
    filter_fields = ('btitle', 'bread')
#视图请求类似如下：
# 127.0.0.1:8000/books/?bread=20
```

![img](Django REST framework的使用总结.assets/16b60d6827ff98c8)

### 7.5 排序OrderingFilter

对于列表数据，REST framework提供了**`OrderingFilter`**过滤器来帮助我们快速指明数据按照指定字段进行排序。同样要实现`ListModelMixin`扩展类及子类的视图才能使用。

#### 使用

在类视图中设置`filter_backends`，使用`rest_framework.filters.OrderingFilter`过滤器，REST framework会在请求的查询字符串参数中检查是否包含了ordering参数，如果包含了ordering参数，则按照ordering参数指明的排序字段对数据集进行排序。

前端可以传递的ordering参数的可选字段值需要在ordering_fields中指明。

```python
class BookListView(ListAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer
    filter_backends = [OrderingFilter]
    ordering_fields = ('id', 'bread', 'bpub_date')

# 127.0.0.1:8000/books/?ordering=-bread # 按阅读量降序查询
```

### 7.6 分页Pagination

REST framework提供了分页的支持。我们可以在配置文件中设置全局的分页方式，如：

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS':  'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 5  # 每页数目
}
```

也可通过自定义Pagination类，来为视图添加不同分页行为。在视图中通过`pagination_class`属性来指明。

```python
from rest_framework.pagination import PageNumberPagination
class LargeResultsSetPagination(PageNumberPagination):
    page_size = 5
    page_size_query_param = 'page_size'
    max_page_size = 10
class BookDetailView(RetrieveAPIView):
    queryset = BookInfo.objects.all().orderby('id')
    serializer_class = BookInfoSerializer
    pagination_class = LargeResultsSetPagination
```

**注意：如果在视图内关闭分页功能，只需在视图内设置**

```
pagination_class = None
```

#### 可选分页器

1） **PageNumberPagination**

前端访问网址形式：

```
GET  http://api.example.org/books/?page=4
```

可以在子类中定义的属性：

- page_size 每页数目
- page_query_param 前端发送的页数关键字名，默认为"page"
- page_size_query_param 前端发送的每页数目关键字名，默认为None
- max_page_size 前端最多能设置的每页数量

```python
from rest_framework.pagination import PageNumberPagination

class StandardPageNumberPagination(PageNumberPagination):
    page_size_query_param = 'page_size'
    max_page_size = 10

class BookListView(ListAPIView):
    queryset = BookInfo.objects.all().order_by('id')
    serializer_class = BookInfoSerializer
    pagination_class = StandardPageNumberPagination

# 127.0.0.1/books/?page=1&page_size=5  #请求每页会返回5条数据
```

![img](Django REST framework的使用总结.assets/16b60d7bd9388076)

2）**LimitOffsetPagination**

前端访问网址形式：

```
GET http://api.example.org/books/?limit=5&offset=10
```

可以在子类中定义的属性：

- default_limit 默认限制，默认值与`PAGE_SIZE`设置一直
- limit_query_param limit参数名，默认'limit'
- offset_query_param offset参数名，默认'offset'
- max_limit 最大limit限制，默认None

```python
from rest_framework.pagination import LimitOffsetPagination

class BookListView(ListAPIView):
    queryset = BookInfo.objects.all().order_by('id')
    serializer_class = BookInfoSerializer
    pagination_class = LimitOffsetPagination

# 127.0.0.1:8000/books/?offset=3&limit=2
```

### 7.7 异常处理 Exceptions

REST framework提供了异常处理，我们可以自定义异常处理函数。

```python
from rest_framework.views import exception_handler

def custom_exception_handler(exc, context):
    # 先调用REST framework默认的异常处理方法获得标准错误响应对象
    response = exception_handler(exc, context)

    # 在此处补充自定义的异常处理
    if response is not None:
        response.data['status_code'] = response.status_code

    return response
```

在配置文件中声明自定义的异常处理

```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'my_project.my_app.utils.custom_exception_handler'#定义处理异常的路径，以项目实际为准
}
```

如果未声明，会采用默认的方式，如下

```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'rest_framework.views.exception_handler' 
}
```

补充上处理关于数据库的异常

```python
from rest_framework.views import exception_handler as drf_exception_handler
from rest_framework import status
from django.db import DatabaseError

def exception_handler(exc, context):
    response = drf_exception_handler(exc, context)

    if response is None:
        view = context['view']
        if isinstance(exc, DatabaseError):
            print('[%s]: %s' % (view, exc))
            response = Response({'detail': '服务器内部错误'}, status=status.HTTP_507_INSUFFICIENT_STORAGE)

    return response
```

#### REST framework定义的异常

- APIException 所有异常的父类
- ParseError 解析错误
- AuthenticationFailed 认证失败
- NotAuthenticated 尚未认证
- PermissionDenied 权限决绝
- NotFound 未找到
- MethodNotAllowed 请求方式不支持
- NotAcceptable 要获取的数据格式不支持
- Throttled 超过限流次数
- ValidationError 校验失败
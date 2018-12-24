# flask/django 动态查询表结构相同表名不同数据的Model实现

## 1.问题

为了控制数据的增长，经常需要分表，数据库中存在多张结构相同，表名相关的表，如：  
table_201706  
table_201707  
table_201708

怎么通过SQLAlchemy 或者django查询相关的数据表，而不用每次都创建Model呢

## 2.解决方法

分别在flask和django下实现，代码如下

### 2.1 flask+sqlalchemy

```python
# -*-coding:utf-8

class NewDynamicModel(object):
    """
    动态产生模型和表的对应关系模型
    :param base_cls: 基类模型，虚类，如TemplateModel
    :param tb_name: 数据库中对应的表名， 如tb_test_2017
    :return: Model class
    eg:
    '''
    class TemplateModel(db.Model):
        __abstract__ = True
        id = db.Column(db.Integer(), autoincrement=True, primary_key=True)
        name = db.Column(db.VARCHAR(32), nullable=False)

    Test_2017 = NewDynamicModel(TemplateModel, 'tb_test_2017')
    print Test_2017.query.all()
    '''
    """
    _instance = dict()

    def __new__(cls, base_cls, tb_name):
        new_cls_name = "%s_To_%s" % (
            base_cls.__name__, '_'.join(map(lambda x:x.capitalize(),tb_name.split('_'))))

        if new_cls_name not in cls._instance:
            model_cls = type(new_cls_name, (base_cls,),
                             {'__tablename__': tb_name})
            cls._instance[new_cls_name] = model_cls

        return cls._instance[new_cls_name]
```


Bug：由于新的数据模型是通过type动态创建的，实际module 下的py文件并不存在该类的代码，因而在通过pickle等方式序列化的时候，会报找不到类的错误。  
Fixbug: 通过inspect库，拷贝基类的代码作为副本，并替换tablename 属性，写入临时类定义文件，并引入module。  
新的方式实现如下：
```python
    class NewDynamicModel(object):
        """
        动态产生模型和表的对应关系模型
        :param base_cls: 基类模型，虚类，如TemplateModel
        :param tb_name: 数据库中对应的表名， 如tb_test_2017
        :return: Model class
        eg:
        '''
        class TemplateModel(db.Model):
            __abstract__ = True
            id = db.Column(db.Integer(), autoincrement=True, primary_key=True)
            name = db.Column(db.VARCHAR(32), nullable=False)


        Test_2017 = NewDynamicModel(TemplateModel, 'tb_test_2017')
        print Test_2017.query.all()
        '''
        """

    @staticmethod
    def get_import_codes(Model):
        """
        获取基类的import依赖
        :param Model:
        :return:
        """
        module = inspect.getmodule(Model)
        all_codelines = inspect.getsourcelines(module)
        import_codes = []
        import_codes.append('# -*-coding:utf-8\n')
        for i in all_codelines[0]:
            match = re.search(r'[from]*[\w|\s]*import [\w|\s]*', i)
            if match:
                import_codes.append(i)
        import_codes.extend(['\n', '\n'])

        return import_codes

    @staticmethod
    def get_codes(Model, new_model_name, tb_name):
        """
        获取基类的实现代码
        :param Model:
        :param new_model_name:
        :param tb_name:
        :return:
        """
        codes = inspect.getsourcelines(Model)[0]
        result = []
        has_alias_tb_name = False
        result.append(codes[0].replace(Model.__name__, new_model_name))
        for line in codes[1:]:
            match = re.search(r'\s+__tablename__\s+=\s+\'(?P<name>\w+)\'', line)
            abstract = re.search(r'(?P<indent>\s+)__abstract__\s+=\s+', line)
            if abstract:
                del line
                continue

            if match:
                name = match.groupdict()['name']
                line = line.replace(name, tb_name)
                has_alias_tb_name = True

            result.append(line)

        if not has_alias_tb_name:
            result.append("%s__tablename__ = '%s'\n" % ('    ', tb_name))

        return result

    @staticmethod
    def create_new_module(module_name, codes):
        """
        创建新表映射类的module文件
        :param module_name:
        :param codes:
        :return:
        """
        f_path = os.path.join(CURRENT_PATH, '_tmp/%s.py' % module_name)
        fp = open(f_path, 'w')
        for i in codes:
            fp.write(i)
        fp.close()

        return import_module('sync_session.models._tmp.%s' % module_name)

    _instance = dict()

    def __new__(cls, base_cls, tb_name):
        new_cls_name = "%s_To_%s" % (
            base_cls.__name__, ''.join(map(lambda x: x.capitalize(), tb_name.split('_'))))

        if tb_name not in engine.table_names():
            raise TableNotExist

        if new_cls_name not in cls._instance:
            import_codes = cls.get_import_codes(base_cls)
            class_codes = cls.get_codes(base_cls, new_cls_name, tb_name)
            import_codes.extend(class_codes)
            new_module_name = new_cls_name.lower()
            new_module = cls.create_new_module(new_module_name, import_codes)
            model_cls = getattr(new_module, new_cls_name)

            cls._instance[new_cls_name] = model_cls

        return cls._instance[new_cls_name]
```

### 2.2 django

```python
# -*-coding:utf-8
from django.db import models

class NewDynamicModel(object):
    """
    动态产生模型和表的对应关系模型
    :param base_cls: 基类模型，虚类，如TemplateModel
    :param tb_name: 数据库中对应的表名， 如tb_test_2017
    :return: Model class
    eg:
    '''
    class TemplateModel(models.Model):
        id = models.AutoField(primary_key=True)
        name = models.CharField(max_length=50)

        class Meta:
            abstract = True

    Test_2017 = NewDynamicModel(TemplateModel, 'tb_test_2017')
    print Test_2017.objects.all()
    '''
    """
    _instance = dict()

    def __new__(cls, base_cls, tb_name):
        new_cls_name = "%s_To_%s" % (
            base_cls.__name__, '_'.join(map(lambda x:x.capitalize(),tb_name.split('_'))))

        if new_cls_name not in cls._instance:
            new_meta_cls = base_cls.Meta
            new_meta_cls.db_table = tb_name
            model_cls = type(new_cls_name, (base_cls,),
                             {'__tablename__': tb_name, 'Meta': new_meta_cls, '__module__': cls.__module__})
            cls._instance[new_cls_name] = model_cls

        return cls._instance[new_cls_name]
```
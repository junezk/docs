如何使用faker创建假数据
==============

有些时候我们需要一些假数据来测试，在python中有一款非常好用的包，faker，可以满足这类的需求。  
项目地址 [https://github.com/joke2k/faker](https://github.com/joke2k/faker)

### 安装：

`pip install faker`

或者  
直接上 github 下载安装即可

### 使用：

要创建假数据，需要先有一个faker实例

    from faker import Faker
    fake = Faker()

然后调用方法即可

    fake.name()
    # 'Kristen Ward'
    
    fake.address()
    # '26745 Todd Park'
    # 'West Jamie, NV 91152-9807'

在faker中，name和address这种方法被称为提供器，提供器的类型很多，除了上面提到的name和address，还有：

* company ： 公司
* zipcode ： 邮编
* bothify ：创建随机字符，他接受参数text，然后把text中的#和?替换成随机的数字和字母,例子：

        fake.bothify(text='### => ???')
        # 849 => DXX
        
            实际上他是同时调用了lexify和numerify，两者同样需要一个text参数，分别把text中的#和?替换为随机数字和字母

    


faker的提供器很多，想查找更多提供器，请看：[https://faker.readthedocs.io/...](https://faker.readthedocs.io/en/latest/providers/)

### 本地化：

想要创建中文数据,可以使用下面的方法:

`fake = Faker('zh_CN')`

Faker接受的本地化参数，创建的对象会生成对应语言的数据，如果没有找到对应的语言，会使用默认的"en_US"  
可以在 [https://faker.readthedocs.io/...](https://faker.readthedocs.io/en/master/locales.html) 中查看所有支持的语言

### 自定义提供器：

有时候，你会想要创建自己的提供器，这时候只需要继承提供器的基类，并实现一些方法，随后调用add_provider将类加入到faker实例中即可

    from faker import Faker
    fake = Faker()
    
    # 引入基类
    from faker.providers import BaseProvider
    
    # 创建并继承，实现方法
    class MyProvider(BaseProvider):
        def foo(self):
            return 'bar'
    
    # 添加到实例中
    fake.add_provider(MyProvider)
    
    # 调用
    fake.foo()
    # 'bar'

### 随机种子：

当你想要生成相同的数据时，你需要调用faker的seed方法为其设置种子，这样之后使用提供器的时候，便会产生相同的内容

    fake.seed(136)
    fake.name()
    # 'Kelsey Hamilton'

注意，在设置随机种子后，如果调用同一个提供器两次，第二次调用会使faker实例的种子失效

```python
fake.seed(1)
fake.name()
# 'Ryan Gallagher'
fake.address()
# '7631 Johnson Village Suite 690\nAdamsbury, NC 50008'
fake.name()
# 'Stanley Parks'
fake.seed(1)
fake.name()
# 'Ryan Gallagher'
fake.name()
# 'Jon Cole'
fake.address()
# '70669 Heather Grove\nNew Sierra, WI 00080-6360'
```
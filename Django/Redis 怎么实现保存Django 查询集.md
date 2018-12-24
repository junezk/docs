# Redis 怎么实现保存Django 查询集

我们在用Python Django开发秒杀系统的时候，要将Django 查询到商品详细信息保存到Redis缓存中去，但Redis不能直接保存对象，但有什么方法呢？

我们发现可用Python的pickle模块。

pickle模块可以序列化对象并保存到磁盘中，并在需要的时候读取出来，任何对象都可以执行序列化操作。

Pickle模块中最常用的函数为：

  1、pickle.dumps(obj[, protocol])

​       函数的功能：将obj对象序列化为string形式，而不是存入文件中。

​       参数讲解：

- - obj：想要序列化的obj对象。
  - protocal：如果该项省略，则默认为0。如果为负值或HIGHEST_PROTOCOL，则使用最高的协议版本。

  2、pickle.loads(string)

​       函数的功能：从string中读出序列化前的obj对象。

​       参数讲解：

- - string：文件名称。

【**代码示例**】

```python
.....
#取出商品id
product_id = request.GET.get('product_id')
#构造商品key
product_key = 'product_{}'.format(product_id)
#根据商品key从缓存中取数据
product_detail = getcache(product_key)
#对取出数据进行判断
if product_detail:
    #如取出了数据，对数据进行序列化读出为对象
    product_detail =  pickle.loads(product_detail)
#如没有数据    
elif   product_detail is None:
     #从数据库中查询数据
    product_detail = SaleProducts.objects.filter(id=product_id)
    #对数据进行序列化，并保存到缓存中
    setcache(product_key,60*10,pickle.dumps(product_detail))
     
......
```

缓存相关函数

```python
# 设置key,value
def setcache(key,time,value):
    master.setex(key,time,value)

# 读取key\value
def getcache(key):
    return master.get(key)
```


## python scrapy框架继承FilesPipeline item_completed不回调的问题 原

现象: 自定义文件下载类, 继承FilesPipeline, 重写item_completed, 但文件下载完不回调的问题

原因:自定义类重写了process_item方法, 但应该返回Deferred实例, 参考父类FilesPipeline的父类MediaPipeline的process_item函数:

```
def process_item(self, item, spider):
    info = self.spiderinfo
    requests = arg_to_iter(self.get_media_requests(item, info))
    dlist = [self._process_request(r, info) for r in requests]
    dfd = DeferredList(dlist, consumeErrors=1)
    return dfd.addCallback(self.item_completed, item, info)
```

FilesPipeline比较特殊, process_item不像其他pipeline返回item或者None, 应该在重写的process_item里返回父类的process_item返回值:

```
return super(XXXFileStorePipeline, self).process_item(item, spider)
```
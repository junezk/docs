# **Flask 文件夹上传：从前端到后端**

文件上传是 Web 开发肯定会碰到的问题，而文件夹上传则更加难缠。网上关于文件夹上传的资料多集中在前端，缺少对于后端的关注，然后讲某个后端框架文件上传的文章又不会涉及文件夹。今天研究了一下这个问题，在此记录。

先说两个问题：

1. 是否所有后端框架都支持文件夹上传？
2. 是否所有浏览器都支持文件夹上传？

第一个问题：YES，第二个问题：NO

只要后端框架对于表单的支持是完整的，那么必然支持文件夹上传。至于浏览器，截至目前，只有 Chrome 支持。 Chrome 大法好！

不要期望文件上传这个功能的浏览器兼容性，这是做不到的。

好，假定我们的所有用户都用上了 Chrome，要怎么做才能成功上传一个文件夹呢？这里不用`drop`这种高大上的东西，就用最传统的`<input>`。用表单 submit 和 ajax 都可以做，先看 submit 方式。

```html
<form method="POST" enctype=multipart/form-data>
  <input type='file' name="file" webkitdirectory >
  <button>upload</button>
</form>
```

我们只要添加上 `webkitdirectory` 这个属性，在选择的时候就可以选择一个文件夹了，如果不加，文件夹被选中的时候就是灰色的。不过貌似加上这个属性就没法选中文件了... `enctype=multipart/form-data` 也是必要的，解释参见[这里](http://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean)

![img](https://laike9m.com/media/content/BlogPost/images/dirup.png)

如果用 ajax 方式，我们可以省去`<form>`，只留下`<input>`就 OK。

```html
<input type='file' webkitdirectory >  
<button id="upload-btn" type="button">upload</button>  
```

但是这样是不够的，关键在于 Js 的使用。

```javascript
var files = [];
$(document).ready(function(){
  $("input").change(function(){
    files = this.files;
  });
});
$("#upload-btn").click(function(){
  var fd = new FormData();
  for (var i = 0; i < files.length; i++) {
    fd.append("file", files[i]);
  }
  $.ajax({
    url: "/upload/",
    method: "POST",
    data: fd,
    contentType: false,
    processData: false,
    cache: false,
    success: function(data){
      console.log(data);
    }
  });
});
```

用 ajax 方式，我们必须手动构造一个 [`FormData` Object](https://developer.mozilla.org/en-US/docs/Web/Guide/Using_FormData_Objects), 然后放在 data 里面提交到后端。 `FormData` 好像就只有一个 `append` 方法，第一个参数是 key，第二个参数是 value，用来构造表单数据。`ajax`请求中，通过 input 元素的 [files](http://www.w3schools.com/jsref/prop_fileupload_files.asp) 属性获取上传的文件。`files`属性不论加不加 `webkitdirectory` 都是存在的，用法也基本一样。不过当我们上传文件夹时，`files` 中会包含文件相对路径的信息，之后会看到。 
用 ajax 上传的好处有两点，首先是异步，这样不会导致页面卡住，其次是能比较方便地实现上传进度条。关于上传进度条的实现可以参考[这里](https://github.com/kirsle/flask-multi-upload)。需要注意的是`contentType`和`processData`必须设置成`false`，参考了[这里](http://stackoverflow.com/questions/9622901/how-to-upload-a-file-using-jquery-ajax-and-formdata)。

前端说完了，再说后端。这里以 flask 为例。

```python
@app.route('/upload/', methods=['POST'])
def upload():
    pprint(request.files.getlist("file"))
    pprint(request.files.getlist("file")[2].filename)
    return "upload success"
```

现在可以解释为什么说所有后端框架都支持文件夹上传了，因为在后端看来文件夹上传和选中多个文件上传并没有什么不同，而后者框架都会支持。flask 的 `getlist` 方法一般用来处理提交的表单中 value 是一个 Array 的情况（前端用`name="key[]"`这种技巧实现），这里用来处理多个上传的文件。

我们选择了一个这样的目录上传

```
car/
|
+--+-car.jpeg
|  +-download.jpeg
|
+--car1/
   |
   +-SS.jpeg
```

`pprint(request.files.getlist("file"))`打出下面的结果：

```python
[<FileStorage: u'car/car.jpeg' ('image/jpeg')>,
 <FileStorage: u'car/download.jpeg' ('image/jpeg')>,
 <FileStorage: u'car/car1/SS.jpeg' ('image/jpeg')>]
```

可以看到，相对路径被保留了下来。可以用`filename`属性获取相对路径，比如`request.files.getlist("file")[2].filename`的结果就是`u'car/car1/SS.jpeg'`。接下来，该怎么保存文件就怎么保存，这就是各个框架自己的事情了。

**EDIT** 
查了一下，确实文件夹上传模式和文件上传模式是不兼容的，参见 [这里](https://code.google.com/p/chromium/issues/detail?id=59818)，引用关键部分：

> We only propagate a single file chooser mode which could be one of: { OPEN*FILE, OPEN*MULTI*FILE, FOLDER, SAVEAS*FILE }. Only one mode can be selected and they cannot be or'ed or combined. Therefore there's no chance to enable both mode.

**EDIT 2** 
文件上传的程序给师兄和自己用了几个月，单文件上传一直很稳定。今天试图传一个包含很多文件(2000+)的文件夹上去，遇到了问题——进度条能够走到最后，但是随后就看到浏览器控制台里报错 `net::ERR_EMPTY_RESPONSE`。看了下后端的输出，Flask 并没有接到请求。首先怀疑的是文件数量过多或者大小太大，查了下，浏览器有限制但似乎是 4G，我还远远没达到，所以不是这个原因。然后看有人提到 PHP 的 timeout，即一个请求如果持续太久后端就直接断掉它。不过 Flask 似乎也没有这种设置。偶然间看到[这里](http://stackoverflow.com/questions/6816215/gunicorn-nginx-timeout-problem)提到 gunicorn 默认 timeout 是 30s，恍然大悟，原来是 gunicorn 导致的。

于是在 gunicorn 启动的时候加上 `--timeout 120`，这次是 500，总算有点进展，因为请求至少发到后端去了。看了 flask 的 log，错误原因是 too many open files，然后 `ulimit` 一下果然只有 1024。Flask 上传文件时，500KB 以下的直接以 StringIO Object 的形式存在内存中，大于 500KB 的用 tempfile 存在磁盘上。所以我传的文件一多，很容易开的文件描述符就超了。改 ulimit 也遇到一点小问题，最后按照[这里](http://stackoverflow.com/questions/17483723/command-not-found-when-using-sudo-ulimit)说的用 `sudo sh -c "ulimit -n 65535 && exec su $LOGNAME"` 解决。之后再次上传就没有问题了。

参考资料: 
1. http://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean 
2. https://developer.mozilla.org/en-US/docs/Web/Guide/UsingFormDataObjects 
3. http://www.w3schools.com/jsref/propfileuploadfiles.asp 
4. https://github.com/kirsle/flask-multi-upload 
5. http://stackoverflow.com/questions/9622901/how-to-upload-a-file-using-jquery-ajax-and-formdata

资料4的那个demo给了我巨大帮助，没它的代码估计我会多花几倍时间，虽然它实现的是文件上传而非文件夹，但其实没什么不同。
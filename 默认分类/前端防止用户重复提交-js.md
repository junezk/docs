# 前端防止用户重复提交-js

# 背景

前端在向后端进行数据提交的时候，通常会需要在第一次提交返回前，阻止用户在快速点击发送二次请求，即防止重复提交，最简单的方法是使用标志参数或者 class 元素控制，但缺点是，每个控制重复提交的地方都需要加上这个逻辑，重复性太强，且控制逻辑不统一。

目前前端使用的是http协议，所以提交方式为两种

- 异步提交，使用jQuery.ajax()
- form 表单同步提交

## 异步防重复提交的方案如下

通过 jQuery 提供的 ajaxPrefilter 方法，将在请求提交之前进行过滤，仅保留第一次请求，后续的请求 abort 阻止掉，具体实现代码如下

```js
/**
 * _pendingRequests = {
 *  'http:xxx.xxxx.do':['domain=P2P','xxxx=aaa'],
 *  'http:xxx.yyyy.do':['domain=P3P','xxxx=bbb']
 * }
 * 该对象的 key 是请求的 url ，value 是由请求参数转化成的字符串数组
 */
var _pendingRequests = {};
$.ajaxPrefilter(function(options, originalOptions, jqXHR) {
    var p_item = {  //保存请求请求的url
            key:options.url,
            index:0
        },
        dataArray = options.data ? options.data.split('&') : [];
        compareData = function(beforD,afterD) {
            //当url相同时，以此比较保存的参数对象，若参数对象相同，则返回false，若第一个就相同，则跳出循环
            // 反之说明当前参数对象列表中没有与将要提交的参数相同，则可看为不同的请求，返回true，允许发起请求
            var result  = false;
            for(var i=0;i<beforD.length;i++){
                if(beforD[i]){
                    result = false;
                    var beforData = beforD[i];
                    for(var j=0;j<beforData.length;j++){
                        if(afterD[j] !== beforData[j]){
                            result = true;
                            break;
                        }
                    }
                    if (!result){
                        break;
                    }
                }else {
                    result = true;
                    continue;
                }
            }
            return result;
        };

    //若请求队列中不存在或者同一个请求不同参数，且不为html后缀，则加入队列中
    if (( !_pendingRequests[p_item.key] || compareData(_pendingRequests[p_item.key],dataArray) ) && p_item.key.indexOf('.html') === -1) {
        //给 index 赋值是因为请求是异步返回的，index用于标记第一个请求
        if(_pendingRequests[p_item.key]){
            p_item.index = _pendingRequests[p_item.key].push(dataArray)-1;
        } else{
            _pendingRequests[p_item.key] = [dataArray];
            p_item.index = 0;
        }
    } else if (p_item.key.indexOf('.html') === -1) {
        jqXHR.abort();	// 放弃后触发的重复提交，仅保留第一次提交
        //pendingRequests[key].abort();	// 放弃先触发的提交
    }
    var complete = options.complete;
    //请求完成
    options.complete = function(jqXHR, textStatus) {
        // 通过 key 和 index 获取成功返回的请求，将其值为 null ，下一次该请求便是在请求队列中便是新的一个请求
        _pendingRequests[p_item.key][p_item.index] = null;
        if ($.isFunction(complete)) {
            complete.apply(this, arguments);
        }
    };
});
```

[jquery.ajaxprefilter官方文档](https://link.juejin.im/?target=http%3A%2F%2Fapi.jquery.com%2Fjquery.ajaxprefilter%2F)

## 表单提交防重复提交的方案如下

表单的处理就稍微要麻烦点，但大致思路和异步的相同，等待第一次请求返回的同时，阻止后续触发的请求发送 首先基于jquery扩展了一个自定义的方法，如下

```js
$.fn.preventDoubleSubmission = function() {
    $(this).on('submit', function(e) {
        var $form = $(this);
        // $form.data('submitted') 通过该变量判断请求的状态
        if ($form.data('submitted') === true) {
            //阻止请求
            e.preventDefault();
        } else {
            $form.data('submitted', true);
            if ($form.attr('target') === '_blank') {
                setTimeout(function() {
                    $form.data('submitted', false);
                }, 800);
            }
        }
    });
    return this;
};
```

当表单初次提交时，通过 jQuery.data() 设置一个标志位，当表单重复提交时，判断设置的标志位，若是提交状态，将阻止表单的提交事件。当form.target = _blank 提交后打开新界面的情况，将在800毫秒后恢复原界面表单可提交状态。

为了方便对全站的表单提交统一处理，可对需要放重复提交的表单添加一个class `preventDouble`,在页面渲染后，统一加上事件监听

```js
//扫描带有 preventDouble 标识的form表单
$(function() {
    var f = $('.contain form.preventDouble');
    for (var i=0;i<f.length;i++){
        $(f[i]).preventDoubleSubmission();
    }
});
```

小贴士

> 提交按钮需使用 `type=’submit’` ，因为监听的是表单的submit事件 不建议多次监听submit事件，会导致放重复提交失效 在表单提交前通常会有些表单检验的操作，所以当校验失败的时候，可以通过 `event.preventDefault()` 阻止表单提交
# JSTREE 实现AJAX重载入时刷新所有节点树

```javascript
$().ready(function() {
    var tree = $('#tree');
    tree.jstree({
        'core': { data: null }
    });
    $("#xreload").on("click", null, function(e) {
        //e.preventDefault();
        var url = $(this).attr("data-url"); //+ "?" + Math.random();
        ajax.get(url, null, "json", function(json) {
            tree.jstree(true).settings.core.data = json;
            tree.jstree(true).refresh();
            //console.debug("reloaded.");
        });
    });
});
```


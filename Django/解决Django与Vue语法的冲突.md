# 解决Django与Vue语法的冲突

在Vue中使用{{ }}，在Django的模板中使用的也是{{ }}，若在模板中即使用Vue也使用Django，就会引起冲突，该如何解决这种冲突呢？

## 方法一：使用verbatim标签解决冲突

自Django1.5以来，加入了 {% verbatim myblock %} {% endverbatim myblock %}标签，被此标签包裹的代码将不会被Django的模板引擎渲染。这样以来，我们可以把带有{{ }} 的Vue代码放在 {% verbatim myblock %}标签里，如下所示：

```html
<div id="app">
    {% verbatim myblock %}
        {{ message }}
    {% endverbatim myblock %}
</div>
```

## 方法二：修改Vue的{{ }} 为{[{ }]}

Vue.config.delimiters = [“{[{“, “}]}”];
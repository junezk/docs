# 全面解析Vue3 Reactive家族和Ref家族API

## 前言

你是不是习惯了Vue2的赋值即响应式？Vue2还有个Vue.observable但你从没用过？结果Vue3像跳跳糖一样跳出来这么多的响应式API，你有没有懵逼的感觉？

不慌，挨个学。由于官方文档写的晦涩难懂，所以我写下这篇。原创：简书microkof。

首先说明，全文提到的“基本数据”是指“数据类型为基本数据类型的数据”，“原始数据”是指被转变为响应式数据之前的纯对象或基本数据。

## reactive

Vue 3的根基。返回对象的响应式副本，响应式转换是“深层”的——它影响所有嵌套property。返回Proxy对象，**不**等于原始对象。建议只操作Proxy对象，不要操作原始对象。

官方建议，对来自于服务器的数据或者注定要响应式的数据执行reactive之前，最好不要用临时变量储存原始数据，因为没有意义，而且两个变量容易让初学者引起误操作。

```xml
<template>
  <button @click="r.a++">count is: {{ r.a }}</button>
</template>

<script>
import { reactive } from "vue";
export default {
  setup() {
    let r = reactive({ a: 1 });
    return {
      r,
    };
  },
};
</script>
```

## ref

ref说白了就是`reactive({value: 原始数据})`。下方代码如果打印r对象，会得到RefImpl（ref）对象，它有一个value属性指向基础类型值30。



```xml
<template>
  <button @click="r++">count is: {{ r }}</button>
</template>

<script>
import { ref } from "vue";
export default {
  setup() {
    let r = ref(30);
    return {
      r,
    };
  },
};
</script>
```

## 为什么似乎Proxy已经解决所有问题，还要有ref API呢？

因为ES的Proxy API是为引用数据类型服务的，它无法为基本数据类型提供代理。如果强行代理，Vue会有提示：`value cannot be made reactive: 30`。

那么为什么Vue2的defineproperty并没有区分基本数据类型和引用数据类型呢？

因为defineproperty就是Object的静态方法，它只是为对象服务的，甚至无法对数组服务，因此Vue 2弄了一个data根对象来存放基本数据类型，这样无论什么类型，都是根对象的property，所以也就能代理基本数据类型。而Proxy能对所有引用类型代理，Vue 3也不再用data根对象，而是一个个的变量，所以带来了新问题，如何代理基本数据类型呢？并没有原生办法，只能构建一个{value: Proxy Object}结构的对象，这样Proxy也就能代理了。

问题来了，同样是响应式结构，ref跟reactive的区别是什么？

## ref与reactive的区别

| 对比             | ref                        | reactive   |
| ---------------- | -------------------------- | ---------- |
| 返回数据类型     | RefImpl对象（也叫ref对象） | Proxy对象  |
| 传入基本类型返回 | `{value: 基本类型}`        | 禁止这么做 |
| 传入引用类型返回 | `{value: Proxy对象}`       | Proxy对象  |

两者分别适用场合：

1. `ref`可以为基本类型添加响应式，也可以为引用类型添加响应式，`reactive`只能为引用类型添加响应式。
2. 对于引用类型，什么时候用`ref`，什么时候用`reactive`？简单说，如果你只打算修改引用类型的一个属性，那么推荐用`reactive`，如果你打算变量重赋值，那么一定要用`ref`。具体见下文。

## 使用组合式API的话，请一定了解“重赋值自身”和“重赋值自身属性”的区别

这一点非常重要。先看配置项式API中重赋值Proxy的范例，跟Vue 2没有区别，页面会渲染出`[ { "name": "赵六", "age": 21 } ]`字符：



```xml
<template>
  <div>
    {{ jsonData }}
  </div>
</template>

<script>
export default {
  data() {
    return {
      jsonData: [
        {
          name: "牛二",
          age: 13,
        },
      ],
    };
  },
  created() {
    this.jsonData = [
      {
        name: "王五",
        age: 19,
      },
    ];
  },
  mounted() {
    this.jsonData = [
      {
        name: "赵六",
        age: 21,
      },
    ];
  },
};
</script>
```

而使用组合式API，你会发现，reactive后的Proxy在onMounted中重赋值无法触发渲染，最终页面显示`[ { "name": "王五", "age": 19 } ]`而不是赵六：



```xml
<template>
  <div>
    {{ jsonData }}
  </div>
</template>

<script>
import { onMounted, reactive } from "vue";
export default {
  setup() {
    let jsonData = reactive([
      {
        name: "牛二",
        age: 13,
      },
    ]);
    jsonData = reactive([
      {
        name: "王五",
        age: 19,
      },
    ]);
    onMounted(() => {
      jsonData = reactive([
        {
          name: "赵六",
          age: 100,
        },
      ]);
    });
    return {
      jsonData,
    };
  },
};
</script>
```

原因在于jsonData尽管是响应式的，但是响应式的是它的属性，而不是它自身，重赋值它自身跟重赋值它的属性是两码事。所以，想在组合式API中让数据具备响应式，必须用`ref`，因为`ref`又对`Proxy`包装了一层，修改`ref`其实是修改它的`value`，它的`value`一定是响应式的，因此视图就正常更新了。

再多说一点，如果数据是服务器返回的LIST数据，而且只显示、不变更，那么最好是使用`shallowRef`来包装数据，可以节能。如果会有变更，那么应该用`ref`。

下例中采用了Ref语法糖。页面会显示Michael的信息：



```xml
<template>
  <div>
    {{ jsonData }}
    <ul>
      <li v-for="item in jsonData" :key="item.name">
        {{ item.name }} - {{ item.age }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
ref: jsonData = [
  {
    name: "Jim",
    age: 13,
  },
];
jsonData = [
  {
    name: "Tom",
    age: 19,
  },
];
onMounted(() => {
  jsonData = [
    {
      name: "Michael",
      age: 100,
    },
  ];
});
</script>
```

现在好像ref把引用数据类型也管起来了，到底啥时候才适合用reactive呢？很简单啊，如果你确信你只可能去改引用类型数据的属性，那么一定要用reactive，如果还有可能要整体重赋值，那还得用ref。所以说：需要在组合式API里给变量重赋值的话，无论什么数据类型都必须用ref，不可以用reactive。

到此，我们清楚了ref与reactive都必须存在的理由，接着说，reactive有一套兄弟API，ref也有一套，它们都是干什么用的？先看reactive的：

## reactive与shallowReactive的区别

打印的话，乍一看没有区别，但是，shallow的中文意义是“浅层的”，shallowReactive不代理深层property，只会指向原始对象的深层property。

注意，给shallowReactive传入Proxy是没有意义的，即便这么做，直接返回该Proxy。

shallowReactive的用途是：如果一个对象的深层不可能变化，那么就没必要深层响应，这时候用shallowReactive可以节省系统开销。

下例中，按下第2个button不会有反应，只有又去按下第1个button之后，视图刷新，第二个button才有反应。



```xml
<template>
<div>
  <button @click="r.b.c++">count is: {{ r.b.c }}</button>
  <button @click="s.b.c++">count is: {{ s.b.c }}</button>
</div>
</template>

<script>
import { reactive, shallowReactive } from "vue";
export default {
  setup() {
    let r = reactive({a: 1, b: {c: 2}});
    console.log(r);
    let s = shallowReactive({a: 1, b: {c: 2}});
    console.log(s);
    return {
      r,s
    };
  },
};
</script>
```

## reactive与readonly的区别

reactive一般只接受ES普通的引用数据类型，尽管它也可以接受Proxy对象，但是没有意义、没有必要，但readonly可以接受Proxy对象，而且有实际意义，它可以获取纯对象或者Proxy或者RefImpl，返回原始代理的只读代理。说白了它做2步操作，先reactive，然后另生成一个只读Proxy。

readonly的只读是深层的只读：访问的任何嵌套property也是只读的。

readonly存在的意义有2个，一个是保护数据不被修改，另一个是提升性能。

下例中，第2个button点击不会有反应。



```xml
<template>
<div>
  <button @click="r.b.c++">count is: {{ r.b.c }}</button>
  <button @click="s.b.c++">count is: {{ s.b.c }}</button>
</div>
</template>

<script>
import { reactive, readonly } from "vue";
export default {
  setup() {
    let r = reactive({a: 1, b: {c: 2}});
    console.log(r);
    let s = readonly({a: 1, b: {c: 2}});
    console.log(s);
    return {
      r,s
    };
  },
};
</script>
```

## readonly与shallowReadonly的区别

就像reactive与shallowReactive的一样，shallowReadonly只会给对象的第一层property设置只读，不去管深层property，因此深层property**并没有被代理**，只会指向原始对象。

下例中：

按下button1会有报错提示：`Set operation on key "c" failed: target is readonly.`，因为r是深层只读的。

按下button2没有任何反应，因为shallowReadonly的深层是指向原始值的，修改原始对象不会反映到视图上。

按下button3也会有报错提示：`Set operation on key "a" failed: target is readonly.`，因为shallowReadonly是浅层只读的，a恰好是浅层property。



```xml
<template>
<div>
  <button @click="r.b.c++">count is: {{ r.b.c }}</button>
  <button @click="s.b.c++">count is: {{ s.b.c }}</button>
  <button @click="s.a++">count is: {{ s.a }}</button>
</div>
</template>

<script>
import { readonly, shallowReadonly } from "vue";
export default {
  setup() {
    let r = readonly({a: 1, b: {c: 2}});
    console.log(r);
    let s = shallowReadonly({a: 1, b: {c: 2}});
    console.log(s);
    return {
      r,s
    };
  },
};
</script>
```

## shallowReactive与shallowReadonly的区别

首先说，两者对深层property的态度是一致的，即“不去代理”，深层property都是指向原始对象的深层property，都允许直接修改原始对象的深层property，区别在于对待浅层property方面。

shallowReactive允许修改浅层property，shallowReadonly不允许，Vue3会阻止并给出报错。

## isReactive、isReadonly、isProxy的区别

- isReactive：Proxy是否是由reactive创建，是则返回true
- isReadonly：Proxy由readonly创建则返回true
- isProxy：上面两个满足任意一条，就返回true

上例中，如果加入这3条打印会得到什么？

```typescript
    console.log(isReadonly(s)); // true
    console.log(isReadonly(s.a)); // false，因为s.a得到的是a的值，而不是a自身，a的值当然不是响应式的
    console.log(isReadonly(s.b)); // false，道理同上
```

## toRaw是什么

官方已经解释的很清楚，返回proxy的原始对象。这是一个转义口，2个作用：可用于临时读取而不会引起proxy访问/跟踪开销，也可用于写入而不会触发视图更新。

官方又说，不建议保留对原始对象的持久引用。请谨慎使用。这句话什么意思？就是说：

1. 尽量不要把原始对象赋值给变量，尽量减少中间变量；
2. 将原始对象转换为Proxy之后，如果你临时打算操作一下原始对象，那么也不要因为这个目的就早早的把原始对象赋值给变量，而是应该用`toRaw(proxy)`，以获取原始对象，比如得到一个变量R，然后你可以操作R，操作完成之后就不要再碰R，而应继续操作Proxy。

下例中：

按下button1，会发现button2也跟着变，这表明Proxy的基本原理：操作Proxy会反映到原始对象身上。

按下button2，没有任何反应，表明操作原始对象不会反映到视图上。这时候重新按下button1，会发现数字跳跃了几个数，这表明直接修改原始对象之后，Proxy对原始对象继续代理，并不需要重新reactive。



```xml
<template>
<div>
  <button @click="r.b.c++">count is: {{ r.b.c }}</button>
  <button @click="s.b.c++">count is: {{ s.b.c }}</button>
</div>
</template>

<script>
import { reactive, toRaw } from "vue";
export default {
  setup() {
    let r = reactive({a: 1, b: {c: 2}});
    console.log(r);
    let s = toRaw(r);
    console.log(s);
    return {
      r,s
    };
  },
};
</script>
```

markRaw与readonly的区别

markRaw是操作原始对象的，它的意义是将原始对象或者原始对象的某个浅层或深层property标记为“永远不允许被代理”。Vue3会给对象的第一层或某深层加一个标记`__v_skip: true`，这样，即便原始对象被reactive之后，得到的该层和更深层就不会被代理。

如果想给原始对象的某个property加markRaw，需要执行3步，先定义变量指向该property，然后markRaw这个变量得到新对象，然后让源对象的property指向新对象。

如果将加了标记的原始对象当做其他原始对象的属性，其他原始对象被reactive之后，加了标记的对象依然不会被reactive。也就是说，reactive见了`__v_skip: true`就绕着走。

注意：虽然Vue3只会在表层加标记，但是会影响深层的property。

markRaw与readonly的区别，在于侧重点不同：

- markRaw允许被修改，但不允许被代理。这里尽管说允许修改，但是修改的意义不大，毕竟Vue的核心思想是响应式，在添加响应式之前修改意义不大。
- readonly不允许被修改，但已经被代理。

它们两者相同点在于，从不同角度节省系统开销。

markRaw的用途：

首先说，直接给某个对象全盘markRaw是没有意义的，因为你就是开发者，你不想让某对象被reactive，那么你不去写reactive就好了啊。所以markRaw的用途应该是：允许对象被reactive，但是阻止对象的部分内容被reactive。

## markRaw与shallowReactive的区别

|              | markRaw    | shallowReactive                          |
| ------------ | ---------- | ---------------------------------------- |
| 作用         | 阻止响应式 | 让浅层property响应式，不操作深层property |
| 浅层property | 阻止响应式 | 执行响应式                               |
| 深层property | 阻止响应式 | 不执行响应式，也不阻止                   |

- 如果希望阻止其他程序员将对象响应式，则可以使用markRaw来保护。
- 如果刚好打算不让从第2层到最深层的所有property响应式，那么用shallowReactive可能更好。
- 如果想要更定制化的阻止某些property被响应式，那么应当使用markRaw。例如下例中，x.b被标记，然后重复值给自己，此时再将x响应式，x.b依然没有被响应式。所以点击button不会有反应。



```xml
<template>
<div>
  <button @click="s.b.c++">count is: {{ s.b.c }}</button>
</div>
</template>

<script>
import { markRaw, reactive } from "vue";
export default {
  setup() {
    let x = {a:1, b: {c: 2}};
    x.b = markRaw(x.b);
    let s = reactive(x);
    console.log(isReactive(s)); // true
    console.log(isReactive(s.b)); // false
    return {
      s
    };
  },
};
</script>
```

最后我们看ref和它的兄弟API。

## ref与shallowRef的区别

|              | ref                           | shallowRef                              |
| ------------ | ----------------------------- | --------------------------------------- |
| 本质         | reactive({value: 原始数据})   | shallowReactive({value: 原始数据})      |
| 区别点       | {value: 原始数据}被深层响应式 | 只有value被响应式，原始数据没有响应式   |
| 传入基本类型 | 两个API无差别                 | 两个API无差别，性能考虑尽量用shallowRef |
| 传入引用类型 | value指向Proxy                | value指向原始数据                       |

shallowRef的作用是只对value添加响应式，因此，必须是value被重新赋值才会触发响应式。shallowRef的出现主要是为了节省系统开销。

下例中，点击button1会有反应，点击button2不会有反应。关键是点击button3，我们知道在<template>里，如果给s重新赋值，其实相当于给s.value重新赋值，由于value是响应式的，这时候button2和button3都会有变化。



```xml
<template>
  <div>
    <button @click="r.b.c++">count is: {{ r.b.c }}</button>
    <button @click="s.b.c++">count is: {{ s.b.c }}</button>
    <button @click="s = { a: 10, b: { c: 20 } }">count is: {{ s.b.c }}</button>
  </div>
</template>

<script>
import { ref, shallowRef } from 'vue';
export default {
  setup() {
    let r = ref({ a: 1, b: { c: 2 } });
    let s = shallowRef({ a: 1, b: { c: 2 } });
    return {
      r,
      s,
    };
  },
};
</script>
```

## toRef是咋回事

先看看这个题目，看看Proxy对象里面的基本数据是否具备响应式：

```xml
<template>
<div>
  <button @click="r.a++">count is: {{ r.a }}</button>
  <button>count is: {{ s }}</button>
</div>
</template>

<script>
import { reactive, toRef } from "vue";
export default {
  setup() {
    let r = reactive({a:1});
    console.log(r);
    let s = r.a;
    console.log(s);
    return {
      r,s
    };
  },
};
</script>
```

当我点击button1的时候，你说button2会变吗？并不会。变量s就是个基本数据，没有任何响应式。很不爽是不是？现在我改改，把`let s = r.a;`改成`let s = toRef(r, 'a');`，然后再试试？

可以看到button2的数字跟着变了！这就是toRef的作用：当一个变量指向一个对象的某个property，且这个property**是基本数据类型**时，必须用toRef才能变量与对象的响应式连接。如果这个property是引用数据类型，就不需要动用toRef。

toRef的用途之一是用于传参，可传递一个响应式的基本数据类型。

toRef还有一个特点是可以提前绑定，看个例子，r的原始数据并没有property叫c，但是我就任性，我就提前让s赋值为toRef(r, 'c')，这时候两个button上是没有数据的，毕竟property c是不存在的，在我点击button1之后，两个button都显示了3，说明提前绑定是有用的。



```xml
<template>
<div>
  <button @click="r.c = 3">count is: {{ r.c }}</button>
  <button>count is: {{ s }}</button>
</div>
</template>

<script>
import { reactive, toRef } from "vue";
export default {
  setup() {
    let r = reactive({a:{b:2}});
    console.log(r);
    let s = toRef(r, 'c');
    console.log(s);
    return {
      r,s
    };
  },
};
</script>
```

## ref与toRef的区别

|      | ref                                                       | toRef                                                |
| ---- | --------------------------------------------------------- | ---------------------------------------------------- |
| 用法 | ref(原始值)                                               | toRef(Proxy, 'xxprop')                               |
| 返回 | ref对象                                                   | 同左                                                 |
| 误区 | 不要给ref传入纯对象的属性，毫无意义且造成困惑，应传原始值 | 不要给toRef传入原始值，毫无意义且造成困惑，应传Proxy |

讲解一下误区。比如下例中：

- 点击button1，打印的ref对象是如期待的`{ count: 4 }`，视图也更新为`4`，但是原始值并没有变，依然是`{ count: 3 }`。这说明：给ref传纯对象的属性会造成困惑。
- 刷新页面，只点击button2，打印的ref对象变了，原始值也变了，但是视图没有更新，还是3。说明：给toRef传入原始值是错误的操作，应当传入Proxy，但也证明toRef对传入值是指向关系。
- 刷新页面，只点击button3，一切如期待，说明：上面的说法是正确的。



```xml
<template>
  <div>
    <p>{{ state1 }}</p>
    <button @click="add1">增加</button>

    <p>{{ state2 }}</p>
    <button @click="add2">增加</button>

    <p>{{ state3.a }} - {{ state4 }}</p>
    <button @click="add3">增加</button>
  </div>
</template>

<script>
import { reactive, ref, toRef } from "vue";
export default {
  setup() {
    const obj = { count: 3 };
    const state1 = ref(obj.count);
    const state2 = toRef(obj, "count");
    const state3 = reactive({ a: 5 });
    const state4 = toRef(state3, "a");

    function add1() {
      state1.value++;
      console.log("原始值obj：", obj);
      console.log("state1：", state1);
    }

    function add2() {
      state2.value++;
      console.log("原始值obj：", obj);
      console.log("state2：", state2);
    }

    function add3() {
      state4.value++;
      console.log("state3：", state3);
      console.log("state4：", state4);
    }

    return { state1, state2, state3, state4, add1, add2, add3 };
  },
};
</script>
```

## toRef与toRefs的区别

toRefs可以看做批量版本的toRef。

|          | toRef                           | toRefs                                      |
| -------- | ------------------------------- | ------------------------------------------- |
| 用法     | toRef(Proxy, 'xxprop')          | toRefs(Proxy)                               |
| 返回     | ObjectRefImpl对象               | 同左                                        |
| 作用     | 创建变量到Proxy属性的响应式连接 | 创建变量每个属性到Proxy每个属性的响应式连接 |
| 连接关系 | 一对一                          | 多对多                                      |

下例中，当点击button1时，所有button都会有反应。

```xml
<template>
<div>
  <button @click="r.c = 3">count is: {{ r.c }}</button>
  <button>count is: {{ s }}</button>
  <button>count is: {{ t.c.value }}</button>
</div>
</template>

<script>
import { reactive, toRef, toRefs } from "vue";
export default {
  setup() {
    let r = reactive({a:{b:2}, c: 4});
    console.log(r);
    let s = toRef(r, 'c');
    console.log(s);
    let t = toRefs(r);
    console.log(t.c)
    return {
      r,s,t
    };
  },
};
</script>
```

toRefs的一大用途是变相解构Proxy。首先了解一个常识，Proxy如果解构，基本数据会丢失响应式。现在我既想要解构Proxy，又不想丢失响应式，怎么办？可以使用toRefs。

下例中，变量c是基本数据，它不具备响应式，因此button1被点击之后，button2不会跟着变。如果将`let {c} = r;`改成`let {c} = toRefs(r);`，则变量c具备了响应式，button2会跟着变。



```xml
<template>
<div>
  <button @click="r.c = 3">count is: {{ r.c }}</button>
  <button>count is: {{ c }}</button>
</div>
</template>

<script>
import { reactive, toRefs } from "vue";
export default {
  setup() {
    let r = reactive({a:{b:2}, c: 4});
    let {c} = r;
    return {
      r,c
    };
  },
};
</script>
```

toRefs的简洁用法：

`return {...toRefs(Proxy), others}`可用于返回解构的Proxy，而不需要创建一个临时变量。

如果组件只需要返回一个解构的Proxy，可以更简略：`return toRefs(Proxy)`。

## customRef是什么

customRef跟ref、toRef、toRefs有很大区别，它生成的ref对象会自定义get和set。

customRef的主要用途至少有2个：

- 时机上说，可以控制视图更新的时机，可以延迟更新。其他ref兄弟API都做不到。
- 内容上说，可以修改传入的原始数据，让原始数据与返回值不相同。其他兄弟API也做不到。

```xml
<template>
<div>
{{text}} - <input v-model="text" />
</div>
</template>

<script>
function useDebouncedRef(value, delay = 200) {
  let timeout
  return customRef((track, trigger) => {
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        clearTimeout(timeout)
        timeout = setTimeout(() => {
          value = newValue + 1
          console.log(value)
          trigger()
        }, delay)
      }
    }
  })
}

export default {
  setup() {
    return {
      text: useDebouncedRef('hello')
    }
  }
}
</script>
```

测试：在<input>里快速敲入一串字符，左边的`{{text}}`位置会延迟出现结果，而且会节流，而且console.log(value)也会延迟打印。

解释：track和trigger，其中track用于追踪，写在return之前。trigger是触发，用在赋值给value语句之后。

将ref改写成customRef应该怎么写？去掉延时，且`value = newValue`即可。

## unref是什么

unref类似于toRaw。unref的本质就是解包，把`{value: Proxy || 基本数据}`解成`Proxy || 基本数据`。unref对toRefs创造的对象的各个属性也起作用，因为各个属性也是ref对象。

由于Vue3的开发原则是尽量不要直接修改内部值，对ref来讲就是尽量不要修改Proxy，如果某些场景下非要直接修改Proxy，需要用unref临时将ref还原为Proxy。与toRaw一样，修改完Proxy之后并不需要重新执行ref。

下例中，点击button1和button2，两个按钮都会有反应。

```xml
<template>
<div>
  <button @click="r.b.c++">count is: {{ r.b.c }}</button>
  <button @click="s.b.c++">count is: {{ s.b.c }}</button>
</div>
</template>

<script>
import { ref, unref } from "vue";
export default {
  setup() {
    let r = ref({a: 1, b: {c: 2}});
    let s = unref(r);
    return {
      r,s
    };
  },
};
</script>
```

## toRaw与unref的区别

用这三行代码就很容易说明了：

```csharp
    const obj = { count: 3 };
    const state = ref(obj);
    console.log(toRaw(state.value) === toRaw(unref(state))); // true
```

说明：

1. 当变量state为ref对象时，state.value === unref(state)为真，两边都是Proxy对象，也说明，想得到ref的Proxy，有2种方式：.value或者unref，完全等价。
2. 当变量state为ref对象时，toRaw(state.value) === toRaw(unref(state))两边都是原始值，也说明想获得ref对象的原始值且原始值为引用类型时，有2种方式，toRaw(state.value)或者toRaw(unref(state))，如果原始值为基本类型，也有2种方式，state.value或者unref(state)。所以，toRaw并不是对ref没用，ref想得到原始对象就得用toRaw。
3. toRaw对ref对象无效，必须作用于ref.value。

## triggerRef是什么

我们知道shallowRef返回的ref对象的value指向的是内部值，如果，我既想使用shallowRef生成ref对象（为了节省开销），又想偶尔修改value指向的内部值的某个property，又希望那个ref对象得到响应，我该怎么办？这时候可以用triggerRef。

下例中，如果注释掉`triggerRef(r)`，那么点击button不会看到任何反应，因为`{a:{b:2}, c: 4}`都是非响应式的。

```xml
<template>
<div>
  <button @click="onClick">count is: {{ r.c }}</button>
</div>
</template>

<script>
import { shallowRef, triggerRef } from "vue";
export default {
  setup() {
    let r = shallowRef({a:{b:2}, c: 4});
    function onClick() {
      r.value.c = 6;
      triggerRef(r);
    }
    return {
      r, onClick
    };
  },
};
</script>
```

## isRef是什么

这个不说了。

## Vue 2的ref在Vue 3怎么用？

这算是一个附录。Vue 2的ref在Vue 3依然存在，依然那样标记：

```xml
<div ref="elA">div元素</div>
```

但是在`setup() {}`里并不是Vue 2那么用，而是定义一个顶层变量，值很简单，一律写成`ref(null)`即可，重要的是变量名，变量名必须与`<div ref="elA">`的`elA`一致，然后onMounted里才能获取、操作elA元素，elA.value就指向这个DOM元素，最后不要忘记return这个elA变量：

```jsx
import { ref, onMounted } from 'vue'
export default {
  setup() {
      const elA = ref(null);

      // 在挂载后才能通过 elA 获取到目标元素
      onMounted(() => {
        elA.value.innerHTML = '内容被修改'
      })

      return {elA}
  }
}
```
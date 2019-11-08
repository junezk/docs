# (建议精读)原生JS灵魂之问(中)，检验自己是否真的熟悉JavaScript？

笔者最近在对原生JS的知识做系统梳理，因为我觉得JS作为前端工程师的根本技术，学再多遍都不为过。打算来做一个系列，一共分三次发，以一系列的问题为驱动，当然也会有追问和扩展，内容系统且完整，对初中级选手会有很好的提升，高级选手也会得到复习和巩固。这是本系列的第二篇。

扫了一眼目录后，也许你可能会说：这些八百年都用不到的东西，我为什么要会?是，我承认真实业务场景中遇到诸如手写splice、深拷贝的场景并不多，但我要说的是，问这些问题的初衷并不是让你拿到平时去用的，而是检验你对`JS语言的理解`有没有到达那样的水准，有一些`边界情况`是否能够考虑到，有没有基本的`计算机素养`(比如最基本的排序方法到底理不理解)，未来有没有潜力去设计出更加优秀的产品或者框架。如果你仅仅是想通过一篇文章来解决业务中的临时问题，那不好意思，请出门左拐，这篇文章确实不适合你。但如果你觉得自己的原生编程能力还有待提高，想让自己的思维能力上一个台阶，希望我这篇"呕心沥血"整理了1万六千多字的文章能够让你有所成长。另外补充一句，本文并不针对面试，但以下任何一篇的内容放在面试中，都是非常惊艳的操作：）

## 第七篇: 函数的arguments为什么不是数组？如何转化成数组？

因为arguments本身并不能调用数组方法，它是一个另外一种对象类型，只不过属性从0开始排，依次为0，1，2...最后还有callee和length属性。我们也把这样的对象称为类数组。

常见的类数组还有：

- 1. 用getElementsByTagName/ClassName()获得的HTMLCollection
- 1. 用querySelector获得的nodeList

那这导致很多数组的方法就不能用了，必要时需要我们将它们转换成数组，有哪些方法呢？

### 1. Array.prototype.slice.call()

```
function sum(a, b) {
  let args = Array.prototype.slice.call(arguments);
  console.log(args.reduce((sum, cur) => sum + cur));//args可以调用数组原生的方法啦
}
sum(1, 2);//3
复制代码
```

### 2. Array.from()

```
function sum(a, b) {
  let args = Array.from(arguments);
  console.log(args.reduce((sum, cur) => sum + cur));//args可以调用数组原生的方法啦
}
sum(1, 2);//3
复制代码
```

这种方法也可以用来转换Set和Map哦！

### 3. ES6展开运算符

```
function sum(a, b) {
  let args = [...arguments];
  console.log(args.reduce((sum, cur) => sum + cur));//args可以调用数组原生的方法啦
}
sum(1, 2);//3
复制代码
```

### 4. 利用concat+apply

```
function sum(a, b) {
  let args = Array.prototype.concat.apply([], arguments);//apply方法会把第二个参数展开
  console.log(args.reduce((sum, cur) => sum + cur));//args可以调用数组原生的方法啦
}
sum(1, 2);//3
复制代码
```

当然，最原始的方法就是再创建一个数组，用for循环把类数组的每个属性值放在里面，过于简单，就不浪费篇幅了。

## 第七篇: forEach中return有效果吗？如何中断forEach循环？

在forEach中用return不会返回，函数会继续执行。

```
let nums = [1, 2, 3];
nums.forEach((item, index) => {
  return;//无效
})
复制代码
```

中断方法：

1. 使用try监视代码块，在需要中断的地方抛出异常。
2. 官方推荐方法（替换方法）：用every和some替代forEach函数。every在碰到return false的时候，中止循环。some在碰到return true的时候，中止循环

## 第八篇: JS判断数组中是否包含某个值

### 方法一：array.indexOf

> 此方法判断数组中是否存在某个值，如果存在，则返回数组元素的下标，否则返回-1。

```
var arr=[1,2,3,4];
var index=arr.indexOf(3);
console.log(index);
复制代码
```

### 方法二：array.includes(searcElement[,fromIndex])

> 此方法判断数组中是否存在某个值，如果存在返回true，否则返回false

```
var arr=[1,2,3,4];
if(arr.includes(3))
    console.log("存在");
else
    console.log("不存在");
复制代码
```

### 方法三：array.find(callback[,thisArg])

> 返回数组中满足条件的**第一个元素的值**，如果没有，返回undefined

```
var arr=[1,2,3,4];
var result = arr.find(item =>{
    return item > 3
});
console.log(result);
复制代码
```

### 方法四：array.findeIndex(callback[,thisArg])

> 返回数组中满足条件的第一个元素的下标，如果没有找到，返回`-1`]

```
var arr=[1,2,3,4];
var result = arr.findIndex(item =>{
    return item > 3
});
console.log(result);
复制代码
```

当然，for循环当然是没有问题的，这里讨论的是数组方法，就不再展开了。

## 第九篇: JS中flat---数组扁平化

对于前端项目开发过程中，偶尔会出现层叠数据结构的数组，我们需要将多层级数组转化为一级数组（即提取嵌套数组元素最终合并为一个数组），使其内容合并且展开。那么该如何去实现呢？

需求:多维数组=>一维数组

```
let ary = [1, [2, [3, [4, 5]]], 6];// -> [1, 2, 3, 4, 5, 6]
let str = JSON.stringify(ary);
复制代码
```

### 1. 调用ES6中的flat方法

```
ary = arr.flat(Infinity);
复制代码
```

### 2. replace + split

```
ary = str.replace(/(\[|\])/g, '').split(',')
复制代码
```

### 3. replace + JSON.parse

```
str = str.replace(/(\[|\]))/g, '');
str = '[' + str + ']';
ary = JSON.parse(str);
复制代码
```

### 4. 普通递归

```
let result = [];
let fn = function(ary) {
  for(let i = 0; i < ary.length; i++) {
    let item = ary[i];
    if (Array.isArray(ary[i])){
      fn(item);
    } else {
      result.push(item);
    }
  }
}
复制代码
```

### 5. 利用reduce函数迭代

```
function flatten(ary) {
    return ary.reduce((pre, cur) => {
        return pre.concat(Array.isArray(cur) ? flatten(cur) : cur);
    }, []);
}
let ary = [1, 2, [3, 4], [5, [6, 7]]]
console.log(flatten(ary))
复制代码
```

### 6：扩展运算符

```
//只要有一个元素有数组，那么循环继续
while (ary.some(Array.isArray)) {
  ary = [].concat(...ary);
}
复制代码
```

这是一个比较实用而且很容易被问到的问题，欢迎大家交流补充。

## 第十篇: JS数组的高阶函数——基础篇

### 1.什么是高阶函数

概念非常简单，如下:

> `一个函数`就可以接收另一个函数作为参数或者返回值为一个函数，`这种函数`就称之为高阶函数。

那对应到数组中有哪些方法呢？

### 2.数组中的高阶函数

#### 1.map

- 参数:接受两个参数，一个是回调函数，一个是回调函数的this值(可选)。

其中，回调函数被默认传入三个值，依次为当前元素、当前索引、整个数组。

- 创建一个新数组，其结果是该数组中的每个元素都调用一个提供的函数后返回的结果
- 对原来的数组没有影响

```
let nums = [1, 2, 3];
let obj = {val: 5};
let newNums = nums.map(function(item,index,array) {
  return item + index + array[index] + this.val; 
  //对第一个元素，1 + 0 + 1 + 5 = 7
  //对第二个元素，2 + 1 + 2 + 5 = 10
  //对第三个元素，3 + 2 + 3 + 5 = 13
}, obj);
console.log(newNums);//[7, 10, 13]
复制代码
```

当然，后面的参数都是可选的 ，不用的话可以省略。

#### 2. reduce

- 参数: 接收两个参数，一个为回调函数，另一个为初始值。回调函数中三个默认参数，依次为积累值、当前值、整个数组。

```
let nums = [1, 2, 3];
// 多个数的加和
let newNums = nums.reduce(function(preSum,curVal,array) {
  return preSum + curVal; 
}, 0);
console.log(newNums);//6
复制代码
```

不传默认值会怎样？

不传默认值会自动以第一个元素为初始值，然后从第二个元素开始依次累计。

#### 3. filter

参数: 一个函数参数。这个函数接受一个默认参数，就是当前元素。这个作为参数的函数返回值为一个布尔类型，决定元素是否保留。

filter方法返回值为一个新的数组，这个数组里面包含参数里面所有被保留的项。

```
let nums = [1, 2, 3];
// 保留奇数项
let oddNums = nums.filter(item => item % 2);
console.log(oddNums);
复制代码
```

#### 4. sort

参数: 一个用于比较的函数，它有两个默认参数，分别是代表比较的两个元素。

举个例子:

```
let nums = [2, 3, 1];
//两个比较的元素分别为a, b
nums.sort(function(a, b) {
  if(a > b) return 1;
  else if(a < b) return -1;
  else if(a == b) return 0;
})
复制代码
```

当比较函数返回值大于0，则 a 在 b 的后面，即a的下标应该比b大。

反之，则 a 在 b 的后面，即 a 的下标比 b 小。

整个过程就完成了一次升序的排列。

当然还有一个需要注意的情况，就是比较函数不传的时候，是如何进行排序的？

> 答案是将数字转换为字符串，然后根据字母unicode值进行升序排序，也就是根据字符串的比较规则进行升序排序。

## 第十一篇: 能不能实现数组map方法 ?

依照 [ecma262 草案](https://tc39.es/ecma262/#sec-array.prototype.map)，实现的map的规范如下:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="726" height="449"></svg>)



下面根据草案的规定一步步来模拟实现map函数:

```
Array.prototype.map = function(callbackFn, thisArg) {
  // 处理数组类型异常
  if (this === null || this === undefined) {
    throw new TypeError("Cannot read property 'map' of null or undefined");
  }
  // 处理回调类型异常
  if (Object.prototype.toString.call(callbackfn) != "[object Function]") {
    throw new TypeError(callbackfn + ' is not a function')
  }
  // 草案中提到要先转换为对象
  let O = Object(this);
  let T = thisArg;

  
  let len = O.length >>> 0;
  let A = new Array(len);
  for(let k = 0; k < len; k++) {
    // 还记得原型链那一节提到的 in 吗？in 表示在原型链查找
    // 如果用 hasOwnProperty 是有问题的，它只能找私有属性
    if (k in O) {
      let kValue = O[k];
      // 依次传入this, 当前项，当前索引，整个数组
      let mappedValue = callbackfn.call(T, KValue, k, O);
      A[k] = mappedValue;
    }
  }
  return A;
}
复制代码
```

这里解释一下, length >>> 0, 字面意思是指"右移 0 位"，但实际上是把前面的空位用0填充，这里的作用是保证len为数字且为整数。

举几个特例：

```
null >>> 0  //0

undefined >>> 0  //0

void(0) >>> 0  //0

function a (){};  a >>> 0  //0

[] >>> 0  //0

var a = {}; a >>> 0  //0

123123 >>> 0  //123123

45.2 >>> 0  //45

0 >>> 0  //0

-0 >>> 0  //0

-1 >>> 0  //4294967295

-1212 >>> 0  //4294966084
复制代码
```

总体实现起来并没那么难，需要注意的就是使用 in 来进行原型链查找。同时，如果没有找到就不处理，能有效处理稀疏数组的情况。

最后给大家奉上V8源码，参照源码检查一下，其实还是实现得很完整了。

```
function ArrayMap(f, receiver) {
  CHECK_OBJECT_COERCIBLE(this, "Array.prototype.map");

  // Pull out the length so that modifications to the length in the
  // loop will not affect the looping and side effects are visible.
  var array = TO_OBJECT(this);
  var length = TO_LENGTH(array.length);
  if (!IS_CALLABLE(f)) throw %make_type_error(kCalledNonCallable, f);
  var result = ArraySpeciesCreate(array, length);
  for (var i = 0; i < length; i++) {
    if (i in array) {
      var element = array[i];
      %CreateDataProperty(result, i, %_Call(f, receiver, element, i, array));
    }
  }
  return result;
}
复制代码
```

参考:

[V8源码](https://github.com/v8/v8/blob/ad82a40509c5b5b4680d4299c8f08d6c6d31af3c/src/js/array.js#L1132)

[Array 原型方法源码实现大揭秘](https://juejin.im/post/5d76f08ef265da03970be192#comment)

[ecma262草案](https://tc39.es/ecma262/#sec-array.prototype.map)

## 第十二篇: 能不能实现数组reduce方法 ?

依照 [ecma262 草案](https://tc39.es/ecma262/#sec-array.prototype.reduce)，实现的reduce的规范如下:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)



其中有几个核心要点:

1、初始值不传怎么处理

2、回调函数的参数有哪些，返回值如何处理。

```
Array.prototype.reduce  = function(callbackfn, initialValue) {
  // 异常处理，和 map 一样
  // 处理数组类型异常
  if (this === null || this === undefined) {
    throw new TypeError("Cannot read property 'reduce' of null or undefined");
  }
  // 处理回调类型异常
  if (Object.prototype.toString.call(callbackfn) != "[object Function]") {
    throw new TypeError(callbackfn + ' is not a function')
  }
  let O = Object(this);
  let len = O.length >>> 0;
  let k = 0;
  let accumulator = initialValue;
  if (accumulator === undefined) {
    for(; k < len ; k++) {
      // 查找原型链
      if (k in O) {
        accumulator = O[k];
        k++;
        break;
      }
    }
  }
  // 表示数组全为空
  if(k === len && accumulator === undefined) 
    throw new Error('Each element of the array is empty');
  for(;k < len; k++) {
    if (k in O) {
      // 注意，核心！
      accumulator = callbackfn.call(undefined, accumulator, O[k], O);
    }
  }
  return accumulator;
}
复制代码
```

其实是从最后一项开始遍历，通过原型链查找跳过空项。

最后给大家奉上V8源码，以供大家检查:

```
function ArrayReduce(callback, current) {
  CHECK_OBJECT_COERCIBLE(this, "Array.prototype.reduce");

  // Pull out the length so that modifications to the length in the
  // loop will not affect the looping and side effects are visible.
  var array = TO_OBJECT(this);
  var length = TO_LENGTH(array.length);
  return InnerArrayReduce(callback, current, array, length,
                          arguments.length);
}

function InnerArrayReduce(callback, current, array, length, argumentsLength) {
  if (!IS_CALLABLE(callback)) {
    throw %make_type_error(kCalledNonCallable, callback);
  }

  var i = 0;
  find_initial: if (argumentsLength < 2) {
    for (; i < length; i++) {
      if (i in array) {
        current = array[i++];
        break find_initial;
      }
    }
    throw %make_type_error(kReduceNoInitial);
  }

  for (; i < length; i++) {
    if (i in array) {
      var element = array[i];
      current = callback(current, element, i, array);
    }
  }
  return current;
}
复制代码
```

参考:

[V8源码](https://github.com/v8/v8/blob/ad82a40509c5b5b4680d4299c8f08d6c6d31af3c/src/js/array.js#L1132)

[ecma262草案](https://tc39.es/ecma262/#sec-array.prototype.map)

## 第十三篇: 能不能实现数组 push、pop 方法 ?

参照 ecma262 草案的规定，关于 push 和 pop 的规范如下图所示:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)





![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="658" height="489"></svg>)



首先来实现一下 push 方法:

```
Array.prototype.push = function(...items) {
  let O = Object(this);
  let len = this.length >>> 0;
  let argCount = items.length >>> 0;
  // 2 ** 53 - 1 为JS能表示的最大正整数
  if (len + argCount > 2 ** 53 - 1) {
    throw new TypeError("The number of array is over the max value restricted!")
  }
  for(let i = 0; i < argCount; i++) {
    O[len + i] = items[i];
  }
  let newLength = len + argCount;
  O.length = newLength;
  return newLength;
}
复制代码
```

亲测已通过MDN上所有测试用例。[MDN链接](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/push)

然后来实现 pop 方法:

```
Array.prototype.pop = function() {
  let O = Object(this);
  let len = this.length >>> 0;
  if (len === 0) {
    O.length = 0;
    return undefined;
  }
  len --;
  let value = O[len];
  delete O[len];
  O.length = len;
  return value;
}
复制代码
```

亲测已通过MDN上所有测试用例。[MDN链接](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/pop)

参考链接:

[V8数组源码](https://github.com/v8/v8/blob/ad82a40509c5b5b4680d4299c8f08d6c6d31af3c/src/js/array.js)

[ecma262规范草案](https://tc39.es/ecma262)

[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)

## 第十四篇: 能不能实现数组filter方法 ?



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="695" height="551"></svg>)



代码如下:

```
Array.prototype.filter = function(callbackfn, thisArg) {
  // 处理数组类型异常
  if (this === null || this === undefined) {
    throw new TypeError("Cannot read property 'filter' of null or undefined");
  }
  // 处理回调类型异常
  if (Object.prototype.toString.call(callbackfn) != "[object Function]") {
    throw new TypeError(callbackfn + ' is not a function')
  }
  let O = Object(this);
  let len = O.length >>> 0;
  let resLen = 0;
  let res = [];
  for(let i = 0; i < len; i++) {
    if (i in O) {
      let element = O[i];
      if (callbackfn.call(thisArg, O[i], i, O)) {
        res[resLen++] = element;
      }
    }
  }
  return res;
}
复制代码
```

MDN上所有测试用例亲测通过。

参考:

[V8数组部分源码第1025行](https://github.com/v8/v8/blob/ad82a40509c5b5b4680d4299c8f08d6c6d31af3c/src/js/array.js)

[MDN中filter文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

## 第十五篇: 能不能实现数组splice方法 ?

splice 可以说是最受欢迎的数组方法之一，api 灵活，使用方便。现在来梳理一下用法:

- 1. splice(position, count) 表示从 position 索引的位置开始，删除count个元素
- 1. splice(position, 0, ele1, ele2, ...) 表示从 position 索引的元素后面插入一系列的元素
- 1. splice(postion, count, ele1, ele2, ...) 表示从 position 索引的位置开始，删除 count 个元素，然后再插入一系列的元素
- 1. 返回值为`被删除元素`组成的`数组`。

接下来我们实现这个方法。

参照ecma262草案的规定，详情请[点击](https://tc39.es/ecma262/#sec-array.prototype.splice)。

首先我们梳理一下实现的思路。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1164" height="467"></svg>)



### 初步实现

```
Array.prototype.splice = function(startIndex, deleteCount, ...addElements)  {
  let argumentsLen = arguments.length;
  let array = Object(this);
  let len = array.length;
  let deleteArr = new Array(deleteCount);
   
  // 拷贝删除的元素
  sliceDeleteElements(array, startIndex, deleteCount, deleteArr);
  // 移动删除元素后面的元素
  movePostElements(array, startIndex, len, deleteCount, addElements);
  // 插入新元素
  for (let i = 0; i < addElements.length; i++) {
    array[startIndex + i] = addElements[i];
  }
  array.length = len - deleteCount + addElements.length;
  return deleteArr;
}
复制代码
```

先拷贝删除的元素，如下所示:

```
const sliceDeleteElements = (array, startIndex, deleteCount, deleteArr) => {
  for (let i = 0; i < deleteCount; i++) {
    let index = startIndex + i;
    if (index in array) {
      let current = array[index];
      deleteArr[i] = current;
    }
  }
};
复制代码
```

然后对删除元素后面的元素进行挪动, 挪动分为三种情况:

1. 添加的元素和删除的元素个数相等
2. 添加的元素个数小于删除的元素
3. 添加的元素个数大于删除的元素

当两者相等时，

```
const movePostElements = (array, startIndex, len, deleteCount, addElements) => {
  if (deleteCount === addElements.length) return;
}
复制代码
```

当添加的元素个数小于删除的元素时, 如图所示:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="993" height="449"></svg>)



```
const movePostElements = (array, startIndex, len, deleteCount, addElements) => {
  //...
  // 如果添加的元素和删除的元素个数不相等，则移动后面的元素
  if(deleteCount > addElements.length) {
    // 删除的元素比新增的元素多，那么后面的元素整体向前挪动
    // 一共需要挪动 len - startIndex - deleteCount 个元素
    for (let i = startIndex + deleteCount; i < len; i++) {
      let fromIndex = i;
      // 将要挪动到的目标位置
      let toIndex = i - (deleteCount - addElements.length);
      if (fromIndex in array) {
        array[toIndex] = array[fromIndex];
      } else {
        delete array[toIndex];
      }
    }
    // 注意注意！这里我们把后面的元素向前挪，相当于数组长度减小了，需要删除冗余元素
    // 目前长度为 len + addElements - deleteCount
    for (let i = len - 1; i >= len + addElements.length - deleteCount; i --) {
      delete array[i];
    }
  } 
};
复制代码
```

当添加的元素个数大于删除的元素时, 如图所示:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="898" height="428"></svg>)



```
const movePostElements = (array, startIndex, len, deleteCount, addElements) => {
  //...
  if(deleteCount < addElements.length) {
    // 删除的元素比新增的元素少，那么后面的元素整体向后挪动
    // 思考一下: 这里为什么要从后往前遍历？从前往后会产生什么问题？
    for (let i = len - 1; i >= startIndex + deleteCount; i--) {
      let fromIndex = i;
      // 将要挪动到的目标位置
      let toIndex = i + (addElements.length - deleteCount);
      if (fromIndex in array) {
        array[toIndex] = array[fromIndex];
      } else {
        delete array[toIndex];
      }
    }
  }
};
复制代码
```

### 优化一: 参数的边界情况

当用户传来非法的 startIndex 和 deleteCount 或者负索引的时候，需要我们做出特殊的处理。

```
const computeStartIndex = (startIndex, len) => {
  // 处理索引负数的情况
  if (startIndex < 0) {
    return startIndex + len > 0 ? startIndex + len: 0;
  } 
  return startIndex >= len ? len: startIndex;
}

const computeDeleteCount = (startIndex, len, deleteCount, argumentsLen) => {
  // 删除数目没有传，默认删除startIndex及后面所有的
  if (argumentsLen === 1) 
    return len - startIndex;
  // 删除数目过小
  if (deleteCount < 0) 
    return 0;
  // 删除数目过大
  if (deleteCount > len - deleteCount) 
    return len - startIndex;
  return deleteCount;
}

Array.prototype.splice = function (startIndex, deleteCount, ...addElements) {
  //,...
  let deleteArr = new Array(deleteCount);
  
  // 下面参数的清洗工作
  startIndex = computeStartIndex(startIndex, len);
  deleteCount = computeDeleteCount(startIndex, len, deleteCount, argumentsLen);
   
  // 拷贝删除的元素
  sliceDeleteElements(array, startIndex, deleteCount, deleteArr);
  //...
}
复制代码
```

### 优化二: 数组为密封对象或冻结对象

什么是密封对象?

> 密封对象是不可扩展的对象，而且已有成员的[[Configurable]]属性被设置为false，这意味着不能添加、删除方法和属性。但是属性值是可以修改的。

什么是冻结对象？

> 冻结对象是最严格的防篡改级别，除了包含密封对象的限制外，还不能修改属性值。

接下来，我们来把这两种情况一一排除。

```
// 判断 sealed 对象和 frozen 对象, 即 密封对象 和 冻结对象
if (Object.isSealed(array) && deleteCount !== addElements.length) {
  throw new TypeError('the object is a sealed object!')
} else if(Object.isFrozen(array) && (deleteCount > 0 || addElements.length > 0)) {
  throw new TypeError('the object is a frozen object!')
}
复制代码
```

好了，现在就写了一个比较完整的splice，如下:

```
const sliceDeleteElements = (array, startIndex, deleteCount, deleteArr) => {
  for (let i = 0; i < deleteCount; i++) {
    let index = startIndex + i;
    if (index in array) {
      let current = array[index];
      deleteArr[i] = current;
    }
  }
};

const movePostElements = (array, startIndex, len, deleteCount, addElements) => {
  // 如果添加的元素和删除的元素个数相等，相当于元素的替换，数组长度不变，被删除元素后面的元素不需要挪动
  if (deleteCount === addElements.length) return;
  // 如果添加的元素和删除的元素个数不相等，则移动后面的元素
  else if(deleteCount > addElements.length) {
    // 删除的元素比新增的元素多，那么后面的元素整体向前挪动
    // 一共需要挪动 len - startIndex - deleteCount 个元素
    for (let i = startIndex + deleteCount; i < len; i++) {
      let fromIndex = i;
      // 将要挪动到的目标位置
      let toIndex = i - (deleteCount - addElements.length);
      if (fromIndex in array) {
        array[toIndex] = array[fromIndex];
      } else {
        delete array[toIndex];
      }
    }
    // 注意注意！这里我们把后面的元素向前挪，相当于数组长度减小了，需要删除冗余元素
    // 目前长度为 len + addElements - deleteCount
    for (let i = len - 1; i >= len + addElements.length - deleteCount; i --) {
      delete array[i];
    }
  } else if(deleteCount < addElements.length) {
    // 删除的元素比新增的元素少，那么后面的元素整体向后挪动
    // 思考一下: 这里为什么要从后往前遍历？从前往后会产生什么问题？
    for (let i = len - 1; i >= startIndex + deleteCount; i--) {
      let fromIndex = i;
      // 将要挪动到的目标位置
      let toIndex = i + (addElements.length - deleteCount);
      if (fromIndex in array) {
        array[toIndex] = array[fromIndex];
      } else {
        delete array[toIndex];
      }
    }
  }
};

const computeStartIndex = (startIndex, len) => {
  // 处理索引负数的情况
  if (startIndex < 0) {
    return startIndex + len > 0 ? startIndex + len: 0;
  } 
  return startIndex >= len ? len: startIndex;
}

const computeDeleteCount = (startIndex, len, deleteCount, argumentsLen) => {
  // 删除数目没有传，默认删除startIndex及后面所有的
  if (argumentsLen === 1) 
    return len - startIndex;
  // 删除数目过小
  if (deleteCount < 0) 
    return 0;
  // 删除数目过大
  if (deleteCount > len - deleteCount) 
    return len - startIndex;
  return deleteCount;
}

Array.prototype.splice = function(startIndex, deleteCount, ...addElements)  {
  let argumentsLen = arguments.length;
  let array = Object(this);
  let len = array.length >>> 0;
  let deleteArr = new Array(deleteCount);

  startIndex = computeStartIndex(startIndex, len);
  deleteCount = computeDeleteCount(startIndex, len, deleteCount, argumentsLen);

  // 判断 sealed 对象和 frozen 对象, 即 密封对象 和 冻结对象
  if (Object.isSealed(array) && deleteCount !== addElements.length) {
    throw new TypeError('the object is a sealed object!')
  } else if(Object.isFrozen(array) && (deleteCount > 0 || addElements.length > 0)) {
    throw new TypeError('the object is a frozen object!')
  }
   
  // 拷贝删除的元素
  sliceDeleteElements(array, startIndex, deleteCount, deleteArr);
  // 移动删除元素后面的元素
  movePostElements(array, startIndex, len, deleteCount, addElements);

  // 插入新元素
  for (let i = 0; i < addElements.length; i++) {
    array[startIndex + i] = addElements[i];
  }

  array.length = len - deleteCount + addElements.length;

  return deleteArr;
}
复制代码
```

以上代码对照[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)中的所有测试用例亲测通过。

相关测试代码请前往: [传送门](https://github.com/sanyuan0704/frontend_daily_question/blob/master/test_splice.js)

最后给大家奉上V8源码，供大家检查： [V8数组 splice 源码第 660 行](https://github.com/v8/v8/blob/ad82a40509c5b5b4680d4299c8f08d6c6d31af3c/src/js/array.js#L660)

## 第十六篇: 能不能实现数组sort方法？

估计大家对 JS 数组的sort 方法已经不陌生了，之前也对它的用法做了详细的总结。那，它的内部是如何来实现的呢？如果说我们能够进入它的内部去看一看， 理解背后的设计，会使我们的思维和素养得到不错的提升。

sort 方法在 V8 内部相对与其他方法而言是一个比较高深的算法，对于很多边界情况做了反复的优化，但是这里我们不会直接拿源码来干讲。我们会来根据源码的思路，实现一个 跟引擎性能**一样**的排序算法，并且一步步拆解其中的奥秘。

### V8 引擎的思路分析

首先大概梳理一下源码中排序的思路:

设要排序的元素个数是n：

- 当 n <= 10 时，采用`插入排序`

- 当 n > 10 时，采用

  ```
  三路快速排序
  ```

  - 10 < n <= 1000, 采用中位数作为哨兵元素
  - n > 1000, 每隔 200~215 个元素挑出一个元素，放到一个新数组，然后对它排序，找到中间位置的数，以此作为中位数

在动手之前，我觉得我们有必要**为什么**这么做搞清楚。

第一、为什么元素个数少的时候要采用插入排序？

虽然`插入排序`理论上说是O(n^2)的算法，`快速排序`是一个O(nlogn)级别的算法。但是别忘了，这只是理论上的估算，在实际情况中两者的算法复杂度前面都会有一个系数的， 当 n 足够小的时候，快速排序`nlogn`的优势会越来越小，倘若插入排序O(n^2)前面的系数足够小，那么就会超过快排。而事实上正是如此，`插入排序`经过优化以后对于小数据集的排序会有非常优越的性能，很多时候甚至会超过快排。

因此，对于很小的数据量，应用`插入排序`是一个非常不错的选择。

第二、为什么要花这么大的力气选择哨兵元素？

因为`快速排序`的性能瓶颈在于递归的深度，最坏的情况是每次的哨兵都是最小元素或者最大元素，那么进行partition(一边是小于哨兵的元素，另一边是大于哨兵的元素)时，就会有一边是空的，那么这么排下去，递归的层数就达到了n, 而每一层的复杂度是O(n)，因此快排这时候会退化成O(n^2)级别。

这种情况是要尽力避免的！如果来避免？

就是让哨兵元素进可能地处于数组的中间位置，让最大或者最小的情况尽可能少。这时候，你就能理解 V8 里面所做的种种优化了。

接下来，我们来一步步实现的这样的官方排序算法。

### 插入排序及优化

最初的插入排序可能是这样写的:

```
const insertSort = (arr, start = 0, end) => {
  end = end || arr.length;
  for(let i = start; i < end; i++) {
    let j;
    for(j = i; j > start && arr[j - 1] > arr[j]; j --) {
      let temp = arr[j];
      arr[j] = arr[j - 1];
      arr[j - 1] = temp;
    }
  }
  return;
}
复制代码
```

看似可以正确的完成排序，但实际上交换元素会有相当大的性能消耗，我们完全可以用变量覆盖的方式来完成，如图所示:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="811" height="505"></svg>)



优化后代码如下:

```
const insertSort = (arr, start = 0, end) => {
  end = end || arr.length;
  for(let i = start; i < end; i++) {
    let e = arr[i];
    let j;
    for(j = i; j > start && arr[j - 1] > e; j --)
      arr[j] = arr[j-1];
    arr[j] = e;
  }
  return;
}
复制代码
```

接下来正式进入到 sort 方法。

### 寻找哨兵元素

sort的骨架大致如下:

```
Array.prototype.sort = (comparefn) => {
  let array = Object(this);
  let length = array.length >>> 0;
  return InnerArraySort(array, length, comparefn);
}

const InnerArraySort = (array, length, comparefn) => {
  // 比较函数未传入
  if (Object.prototype.toString.call(callbackfn) !== "[object Function]") {
    comparefn = function (x, y) {
      if (x === y) return 0;
      x = x.toString();
      y = y.toString();
      if (x == y) return 0;
      else return x < y ? -1 : 1;
    };
  }
  const insertSort = () => {
    //...
  }
  const getThirdIndex = (a, from, to) => {
    // 元素个数大于1000时寻找哨兵元素
  }
  const quickSort = (a, from, to) => {
    //哨兵位置
    let thirdIndex = 0;
    while(true) {
      if(to - from <= 10) {
        insertSort(a, from, to);
        return;
      }
      if(to - from > 1000) {
        thirdIndex = getThirdIndex(a, from , to);
      }else {
        // 小于1000 直接取中点
        thirdIndex = from + ((to - from) >> 2);
      }
    }
    //下面开始快排
  }
}
复制代码
```

我们先来把求取哨兵位置的代码实现一下:

```
const getThirdIndex = (a, from, to) => {
  let tmpArr = [];
  // 递增量，200~215 之间，因为任何正数和15做与操作，不会超过15，当然是大于0的
  let increment = 200 + ((to - from) & 15);
  let j = 0;
  from += 1;
  to -= 1;
  for (let i = from; i < to; i += increment) {
    tmpArr[j] = [i, a[i]];
    j++;
  }
  // 把临时数组排序，取中间的值，确保哨兵的值接近平均位置
  tmpArr.sort(function(a, b) {
    return comparefn(a[1], b[1]);
  });
  let thirdIndex = tmpArr[tmpArr.length >> 1][0];
  return thirdIndex;
}
复制代码
```

### 完成快排

接下来我们来完成快排的具体代码：

```
const _sort = (a, b, c) => {
  let arr = [a, b, c];
  insertSort(arr, 0, 3);
  return arr;
}

const quickSort = (a, from, to) => {
  //...
  // 上面我们拿到了thirdIndex
  // 现在我们拥有三个元素，from, thirdIndex, to
  // 为了再次确保 thirdIndex 不是最值，把这三个值排序
  [a[from], a[thirdIndex], a[to - 1]] = _sort(a[from], a[thirdIndex], a[to - 1]);
  // 现在正式把 thirdIndex 作为哨兵
  let pivot = a[thirdIndex];
  // 正式进入快排
  let lowEnd = from + 1;
  let highStart = to - 1;
  // 现在正式把 thirdIndex 作为哨兵, 并且lowEnd和thirdIndex交换
  let pivot = a[thirdIndex];
  a[thirdIndex] = a[lowEnd];
  a[lowEnd] = pivot;
  
  // [lowEnd, i)的元素是和pivot相等的
  // [i, highStart) 的元素是需要处理的
  for(let i = lowEnd + 1; i < highStart; i++) {
    let element = a[i];
    let order = comparefn(element, pivot);
    if (order < 0) {
      a[i] = a[lowEnd];
      a[lowEnd] = element;
      lowEnd++;
    } else if(order > 0) {
      do{
        highStart--;
        if(highStart === i) break;
        order = comparefn(a[highStart], pivot);
      }while(order > 0)
      // 现在 a[highStart] <= pivot
      // a[i] > pivot
      // 两者交换
      a[i] = a[highStart];
      a[highStart] = element;
      if(order < 0) {
        // a[i] 和 a[lowEnd] 交换
        element = a[i];
        a[i] = a[lowEnd];
        a[lowEnd] = element;
        lowEnd++;
      }
    }
  }
  // 永远切分大区间
  if (lowEnd - from > to - highStart) {
    // 继续切分lowEnd ~ from 这个区间
    to = lowEnd;
    // 单独处理小区间
    quickSort(a, highStart, to);
  } else if(lowEnd - from <= to - highStart) {
    from = highStart;
    quickSort(a, from, lowEnd);
  }
}
复制代码
```

### 测试结果

测试结果如下:

一万条数据:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="299" height="123"></svg>)



十万条数据:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)



一百万条数据:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="291" height="119"></svg>)



一千万条数据:



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="276" height="120"></svg>)



结果仅供大家参考，因为不同的node版本对于部分细节的实现可能不一样，我现在的版本是v10.15。

从结果可以看到，目前版本的 node 对于有序程度较高的数据是处理的不够好的，而我们刚刚实现的排序通过反复确定哨兵的位置就能 有效的规避快排在这一场景下的劣势。

最后给大家完整版的sort代码:

```
const sort = (arr, comparefn) => {
  let array = Object(arr);
  let length = array.length >>> 0;
  return InnerArraySort(array, length, comparefn);
}

const InnerArraySort = (array, length, comparefn) => {
  // 比较函数未传入
  if (Object.prototype.toString.call(comparefn) !== "[object Function]") {
    comparefn = function (x, y) {
      if (x === y) return 0;
      x = x.toString();
      y = y.toString();
      if (x == y) return 0;
      else return x < y ? -1 : 1;
    };
  }
  const insertSort = (arr, start = 0, end) => {
    end = end || arr.length;
    for (let i = start; i < end; i++) {
      let e = arr[i];
      let j;
      for (j = i; j > start && comparefn(arr[j - 1], e) > 0; j--)
        arr[j] = arr[j - 1];
      arr[j] = e;
    }
    return;
  }
  const getThirdIndex = (a, from, to) => {
    let tmpArr = [];
    // 递增量，200~215 之间，因为任何正数和15做与操作，不会超过15，当然是大于0的
    let increment = 200 + ((to - from) & 15);
    let j = 0;
    from += 1;
    to -= 1;
    for (let i = from; i < to; i += increment) {
      tmpArr[j] = [i, a[i]];
      j++;
    }
    // 把临时数组排序，取中间的值，确保哨兵的值接近平均位置
    tmpArr.sort(function (a, b) {
      return comparefn(a[1], b[1]);
    });
    let thirdIndex = tmpArr[tmpArr.length >> 1][0];
    return thirdIndex;
  };

  const _sort = (a, b, c) => {
    let arr = [];
    arr.push(a, b, c);
    insertSort(arr, 0, 3);
    return arr;
  }

  const quickSort = (a, from, to) => {
    //哨兵位置
    let thirdIndex = 0;
    while (true) {
      if (to - from <= 10) {
        insertSort(a, from, to);
        return;
      }
      if (to - from > 1000) {
        thirdIndex = getThirdIndex(a, from, to);
      } else {
        // 小于1000 直接取中点
        thirdIndex = from + ((to - from) >> 2);
      }
      let tmpArr = _sort(a[from], a[thirdIndex], a[to - 1]);
      a[from] = tmpArr[0]; a[thirdIndex] = tmpArr[1]; a[to - 1] = tmpArr[2];
      // 现在正式把 thirdIndex 作为哨兵
      let pivot = a[thirdIndex];
      [a[from], a[thirdIndex]] = [a[thirdIndex], a[from]];
      // 正式进入快排
      let lowEnd = from + 1;
      let highStart = to - 1;
      a[thirdIndex] = a[lowEnd];
      a[lowEnd] = pivot;
      // [lowEnd, i)的元素是和pivot相等的
      // [i, highStart) 的元素是需要处理的
      for (let i = lowEnd + 1; i < highStart; i++) {
        let element = a[i];
        let order = comparefn(element, pivot);
        if (order < 0) {
          a[i] = a[lowEnd];
          a[lowEnd] = element;
          lowEnd++;
        } else if (order > 0) {
          do{
            highStart--;
            if (highStart === i) break;
            order = comparefn(a[highStart], pivot);
          }while (order > 0) ;
          // 现在 a[highStart] <= pivot
          // a[i] > pivot
          // 两者交换
          a[i] = a[highStart];
          a[highStart] = element;
          if (order < 0) {
            // a[i] 和 a[lowEnd] 交换
            element = a[i];
            a[i] = a[lowEnd];
            a[lowEnd] = element;
            lowEnd++;
          }
        }
      }
      // 永远切分大区间
      if (lowEnd - from > to - highStart) {
        // 单独处理小区间
        quickSort(a, highStart, to);
        // 继续切分lowEnd ~ from 这个区间
        to = lowEnd;
      } else if (lowEnd - from <= to - highStart) {
        quickSort(a, from, lowEnd);
        from = highStart;
      }
    }
  }
  quickSort(array, 0, length);
}
复制代码
```

参考链接:

[V8 sort源码(点开第997行)](https://github.com/v8/v8/blob/ad82a40509c5b5b4680d4299c8f08d6c6d31af3c/src/js/array.js#997)

[冴羽排序源码专题](https://juejin.im/post/59e80dc6f265da432a7aaf15)

## 第十七篇: 能不能模拟实现一个new的效果？

`new`被调用后做了三件事情:

1. 让实例可以访问到私有属性
2. 让实例可以访问构造函数原型(constructor.prototype)所在原型链上的属性
3. 如果构造函数返回的结果不是引用数据类型

```
function newOperator(ctor, ...args) {
    if(typeof ctor !== 'function'){
      throw 'newOperator function the first param must be a function';
    }
    let obj = Object.create(ctor.prototype);
    let res = ctor.apply(obj, args);
    
    let isObject = typeof res === 'object' && res !== null;
    let isFunction = typoof res === 'function';
    return isObect || isFunction ? res : obj;
};
复制代码
```

## 第十八篇: 能不能模拟实现一个 bind 的效果？

实现bind之前，我们首先要知道它做了哪些事情。

1. 对于普通函数，绑定this指向
2. 对于构造函数，要保证原函数的原型对象上的属性不能丢失

```
Function.prototype.bind = function (context, ...args) {
    // 异常处理
    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }
    // 保存this的值，它代表调用 bind 的函数
    var self = this;
    var fNOP = function () {};

    var fbound = function () {
        self.apply(this instanceof self ? 
            this : 
            context, args.concat(Array.prototype.slice.call(arguments)));
    }

    fNOP.prototype = this.prototype;
    fbound.prototype = new fNOP();

    return fbound;
}
复制代码
```

也可以这么用 Object.create 来处理原型:

```
Function.prototype.bind = function (context, ...args) {
    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;

    var fbound = function () {
        self.apply(this instanceof self ? 
            this : 
            context, args.concat(Array.prototype.slice.call(arguments)));
    }

    fbound.prototype = Object.create(self.prototype);

    return fbound;
}
复制代码
```

## 第十八篇: 能不能实现一个 call/apply 函数？

引自`冴羽`大佬的代码，可以说比较完整了。

```
Function.prototype.call = function (context) {
    var context = context || window;
    context.fn = this;

    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }

    var result = eval('context.fn(' + args +')');

    delete context.fn
    return result;
}
复制代码
```

不过我认为换成 ES6 的语法会更精炼一些:

```
Function.prototype.call = function (context, ...args) {
  var context = context || window;
  context.fn = this;

  var result = eval('context.fn(...args)');

  delete context.fn
  return result;
}
复制代码
```

类似的，有apply的对应实现:

```
Function.prototype.apply = function (context, args) {
  let context = context || window;
  context.fn = this;
  let result = eval('context.fn(...args)');

  delete context.fn
  return result;
}
复制代码
```

## 第十九篇: 谈谈你对JS中this的理解。

其实JS中的this是一个非常简单的东西，只需要理解它的执行规则就OK。

在这里不想像其他博客一样展示太多的代码例子弄得天花乱坠， 反而不易理解。

call/apply/bind可以显式绑定, 这里就不说了。

主要这些场隐式绑定的场景讨论:

1. 全局上下文
2. 直接调用函数
3. 对象.方法的形式调用
4. DOM事件绑定(特殊)
5. new构造函数绑定
6. 箭头函数

### 1. 全局上下文

全局上下文默认this指向window, 严格模式下指向undefined。

### 2. 直接调用函数

比如:

```
let obj = {
  a: function() {
    console.log(this);
  }
}
let func = obj.a;
func();
复制代码
```

这种情况是直接调用。this相当于全局上下文的情况。

### 3. 对象.方法的形式调用

还是刚刚的例子，我如果这样写:

```
obj.a();
复制代码
```

这就是`对象.方法`的情况，this指向这个对象

### 4. DOM事件绑定

onclick和addEventerListener中 this 默认指向绑定事件的元素。

IE比较奇异，使用attachEvent，里面的this默认指向window。

### 5. new+构造函数

此时构造函数中的this指向实例对象。

### 6. 箭头函数？

箭头函数没有this, 因此也不能绑定。里面的this会指向当前最近的非箭头函数的this，找不到就是window(严格模式是undefined)。比如:

```
let obj = {
  a: function() {
    let do = () => {
      console.log(this);
    }
    do();
  }
}
obj.a(); // 找到最近的非箭头函数a，a现在绑定着obj, 因此箭头函数中的this是obj
复制代码
```

> 优先级: new > call、apply、bind > 对象.方法 > 直接调用。

## 第二十篇: JS中浅拷贝的手段有哪些？

### 重要: 什么是拷贝？

首先来直观的感受一下什么是拷贝。

```
let arr = [1, 2, 3];
let newArr = arr;
newArr[0] = 100;

console.log(arr);//[100, 2, 3]
复制代码
```

这是直接赋值的情况，不涉及任何拷贝。当改变newArr的时候，由于是同一个引用，arr指向的值也跟着改变。

现在进行浅拷贝:

```
let arr = [1, 2, 3];
let newArr = arr.slice();
newArr[0] = 100;

console.log(arr);//[1, 2, 3]
复制代码
```

当修改newArr的时候，arr的值并不改变。什么原因?因为这里newArr是arr浅拷贝后的结果，newArr和arr现在引用的已经不是同一块空间啦！

这就是浅拷贝！

但是这又会带来一个潜在的问题:

```
let arr = [1, 2, {val: 4}];
let newArr = arr.slice();
newArr[2].val = 1000;

console.log(arr);//[ 1, 2, { val: 1000 } ]
复制代码
```

咦!不是已经不是同一块空间的引用了吗？为什么改变了newArr改变了第二个元素的val值，arr也跟着变了。

这就是浅拷贝的限制所在了。它只能拷贝一层对象。如果有对象的嵌套，那么浅拷贝将无能为力。但幸运的是，深拷贝就是为了解决这个问题而生的，它能 解决无限极的对象嵌套问题，实现彻底的拷贝。当然，这是我们下一篇的重点。 现在先让大家有一个基本的概念。

接下来，我们来研究一下JS中实现浅拷贝到底有多少种方式？

### 1. 手动实现

```
const shallowClone = (target) => {
  if (typeof target === 'object' && target !== null) {
    const cloneTarget = Array.isArray(target) ? []: {};
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
          cloneTarget[prop] = target[prop];
      }
    }
    return cloneTarget;
  } else {
    return target;
  }
}
复制代码
```

### 2. Object.assign

但是需要注意的是，Object.assgin() 拷贝的是对象的属性的引用，而不是对象本身。

```
let obj = { name: 'sy', age: 18 };
const obj2 = Object.assign({}, obj, {name: 'sss'});
console.log(obj2);//{ name: 'sss', age: 18 }
复制代码
```

### 3. concat浅拷贝数组

```
let arr = [1, 2, 3];
let newArr = arr.concat();
newArr[1] = 100;
console.log(arr);//[ 1, 2, 3 ]
复制代码
```

### 4. slice浅拷贝

开头的例子不就说的这个嘛！

### 5. ...展开运算符

```
let arr = [1, 2, 3];
let newArr = [...arr];//跟arr.slice()是一样的效果
复制代码
```

## 第二十一篇: 能不能写一个完整的深拷贝？

上一篇已经解释了什么是深拷贝，现在我们来一起实现一个完整且专业的深拷贝。

### 1. 简易版及问题

```
JSON.parse(JSON.stringify());
复制代码
```

估计这个api能覆盖大多数的应用场景，没错，谈到深拷贝，我第一个想到的也是它。但是实际上，对于某些严格的场景来说，这个方法是有巨大的坑的。问题如下：

> 1. 无法解决`循环引用`的问题。举个例子：

```
const a = {val:2};
a.target = a;
复制代码
```

拷贝a会出现系统栈溢出，因为出现了`无限递归`的情况。

> 1. 无法拷贝一写`特殊的对象`，诸如 RegExp, Date, Set, Map等。

> 1. 无法拷贝`函数`(划重点)。

因此这个api先pass掉，我们重新写一个深拷贝，简易版如下:

```
const deepClone = (target) => {
  if (typeof target === 'object' && target !== null) {
    const cloneTarget = Array.isArray(target) ? []: {};
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
          cloneTarget[prop] = deepClone(target[prop]);
      }
    }
    return cloneTarget;
  } else {
    return target;
  }
}
复制代码
```

现在，我们以刚刚发现的三个问题为导向，一步步来完善、优化我们的深拷贝代码。

### 2. 解决循环引用

现在问题如下:

```
let obj = {val : 100};
obj.target = obj;

deepClone(obj);//报错: RangeError: Maximum call stack size exceeded
复制代码
```

这就是循环引用。我们怎么来解决这个问题呢？

创建一个Map。记录下已经拷贝过的对象，如果说已经拷贝过，那直接返回它行了。

```
const isObject = (target) => (typeof target === 'object' || typeof target === 'function') && target !== null;

const deepClone = (target, map = new Map()) => {
  if(map.get(target)) 
    return target;


  if (isObject(target)) {
    map.put(target, true);
    const cloneTarget = Array.isArray(target) ? []: {};
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
          cloneTarget[prop] = deepClone(target[prop]);
      }
    }
    return cloneTarget;
  } else {
    return target;
  }
}
复制代码
```

现在来试一试：

```
const a = {val:2};
a.target = a;
let newA = deepClone(a);
console.log(newA)//{ val: 2, target: { val: 2, target: [Circular] } }
复制代码
```

好像是没有问题了, 拷贝也完成了。但还是有一个潜在的坑, 就是map 上的 key 和 map 构成了`强引用关系`，这是相当危险的。我给你解释一下与之相对的弱引用的概念你就明白了：

> 在计算机程序设计中，弱引用与强引用相对， 是指不能确保其引用的对象不会被垃圾回收器回收的引用。 一个对象若只被弱引用所引用，则被认为是不可访问（或弱可访问）的，并因此可能在任何时刻被回收。 --百度百科

说的有一点绕，我用大白话解释一下，被弱引用的对象可以在`任何时候被回收`，而对于强引用来说，只要这个强引用还在，那么对象`无法被回收`。拿上面的例子说，map 和 a一直是强引用的关系， 在程序结束之前，a 所占的内存空间一直`不会被释放`。

怎么解决这个问题？

很简单，让 map 的 key 和 map 构成`弱引用`即可。ES6给我们提供了这样的数据结构，它的名字叫`WeakMap`，它是一种特殊的Map, 其中的键是`弱引用`的。其键必须是对象，而值可以是任意的。

稍微改造一下即可:

```
const deepClone = (target, map = new WeakMap()) => {
  //...
}
复制代码
```

### 3. 拷贝特殊对象

#### 可继续遍历

对于特殊的对象，我们使用以下方式来鉴别:

```
Object.prototype.toString.call(obj);
复制代码
```

梳理一下对于可遍历对象会有什么结果：

```
["object Map"]
["object Set"]
["object Array"]
["object Object"]
["object Arguments"]
复制代码
```

好，以这些不同的字符串为依据，我们就可以成功地鉴别这些对象。

```
const getType = Object.prototype.toString.call(obj);

const canTraverse = {
  '[object Map]': true,
  '[object Set]': true,
  '[object Array]': true,
  '[object Object]': true,
  '[object Arguments]': true,
};

const deepClone = (target, map = new Map()) => {
  if(!isObject(target)) 
    return target;
  let type = getType(target);
  let cloneTarget;
  if(!canTraverse[type]) {
    // 处理不能遍历的对象
    return;
  }else {
    // 这波操作相当关键，可以保证对象的原型不丢失！
    let ctor = target.prototype;
    cloneTarget = new ctor();
  }

  if(map.get(target)) 
    return target;
  map.put(target, true);

  if(type === mapTag) {
    //处理Map
    target.forEach((item, key) => {
      cloneTarget.set(deepClone(key), deepClone(item));
    })
  }
  
  if(type === setTag) {
    //处理Set
    target.forEach(item => {
      target.add(deepClone(item));
    })
  }

  // 处理数组和对象
  for (let prop in target) {
    if (target.hasOwnProperty(prop)) {
        cloneTarget[prop] = deepClone(target[prop]);
    }
  }
  return cloneTarget;
}
复制代码
```

#### 不可遍历的对象

```
const boolTag = '[object Boolean]';
const numberTag = '[object Number]';
const stringTag = '[object String]';
const dateTag = '[object Date]';
const errorTag = '[object Error]';
const regexpTag = '[object RegExp]';
const funcTag = '[object Function]';
复制代码
```

对于不可遍历的对象，不同的对象有不同的处理。

```
const handleRegExp = (target) => {
  const { source, flags } = target;
  return new target.constructor(source, flags);
}

const handleFunc = (target) => {
  // 待会的重点部分
}

const handleNotTraverse = (target, tag) => {
  const Ctor = targe.constructor;
  switch(tag) {
    case boolTag:
    case numberTag:
    case stringTag:
    case errorTag: 
    case dateTag:
      return new Ctor(target);
    case regexpTag:
      return handleRegExp(target);
    case funcTag:
      return handleFunc(target);
    default:
      return new Ctor(target);
  }
}
复制代码
```

### 4. 拷贝函数

虽然函数也是对象，但是它过于特殊，我们单独把它拿出来拆解。

提到函数，在JS种有两种函数，一种是普通函数，另一种是箭头函数。每个普通函数都是 Function的实例，而箭头函数不是任何类的实例，每次调用都是不一样的引用。那我们只需要 处理普通函数的情况，箭头函数直接返回它本身就好了。

那么如何来区分两者呢？

答案是: 利用原型。箭头函数是不存在原型的。

代码如下:

```
const handleFunc = (func) => {
  // 箭头函数直接返回自身
  if(!func.prototype) return func;
  const bodyReg = /(?<={)(.|\n)+(?=})/m;
  const paramReg = /(?<=\().+(?=\)\s+{)/;
  const funcString = func.toString();
  // 分别匹配 函数参数 和 函数体
  const param = paramReg.exec(funcString);
  const body = bodyReg.exec(funcString);
  if(!body) return null;
  if (param) {
    const paramArr = param[0].split(',');
    return new Function(...paramArr, body[0]);
  } else {
    return new Function(body[0]);
  }
}
复制代码
```

到现在，我们的深拷贝就实现地比较完善了。不过在测试的过程中，我也发现了一个小小的bug。

### 5. 小小的bug

如下所示:

```
const target = new Boolean(false);
const Ctor = target.constructor;
new Ctor(target); // 结果为 Boolean {true} 而不是 false。
复制代码
```

对于这样一个bug，我们可以对 Boolean 拷贝做最简单的修改， 调用valueOf: new target.constructor(target.valueOf())。

但实际上，这种写法是不推荐的。因为在ES6后不推荐使用【new 基本类型()】这 样的语法，所以es6中的新类型 Symbol 是不能直接 new 的，只能通过 new Object(SymbelType)。

因此我们接下来统一一下:

```
const handleNotTraverse = (target, tag) => {
  const Ctor = targe.constructor;
  switch(tag) {
    case boolTag:
      return new Object(Boolean.prototype.valueOf.call(target));
    case numberTag:
      return new Object(Number.prototype.valueOf.call(target));
    case stringTag:
      return new Object(String.prototype.valueOf.call(target));
    case errorTag: 
    case dateTag:
      return new Ctor(target);
    case regexpTag:
      return handleRegExp(target);
    case funcTag:
      return handleFunc(target);
    default:
      return new Ctor(target);
  }
}
复制代码
```

### 6. 完整代码展示

OK!是时候给大家放出完整版的深拷贝啦:

```js
const getType = obj => Object.prototype.toString.call(obj);

const isObject = (target) => (typeof target === 'object' || typeof target === 'function') && target !== null;

const canTraverse = {
  '[object Map]': true,
  '[object Set]': true,
  '[object Array]': true,
  '[object Object]': true,
  '[object Arguments]': true,
};
const mapTag = '[object Map]';
const setTag = '[object Set]';
const boolTag = '[object Boolean]';
const numberTag = '[object Number]';
const stringTag = '[object String]';
const symbolTag = '[object Symbol]';
const dateTag = '[object Date]';
const errorTag = '[object Error]';
const regexpTag = '[object RegExp]';
const funcTag = '[object Function]';

const handleRegExp = (target) => {
  const { source, flags } = target;
  return new target.constructor(source, flags);
}

const handleFunc = (func) => {
  // 箭头函数直接返回自身
  if(!func.prototype) return func;
  const bodyReg = /(?<={)(.|\n)+(?=})/m;
  const paramReg = /(?<=\().+(?=\)\s+{)/;
  const funcString = func.toString();
  // 分别匹配 函数参数 和 函数体
  const param = paramReg.exec(funcString);
  const body = bodyReg.exec(funcString);
  if(!body) return null;
  if (param) {
    const paramArr = param[0].split(',');
    return new Function(...paramArr, body[0]);
  } else {
    return new Function(body[0]);
  }
}

const handleNotTraverse = (target, tag) => {
  const Ctor = target.constructor;
  switch(tag) {
    case boolTag:
      return new Object(Boolean.prototype.valueOf.call(target));
    case numberTag:
      return new Object(Number.prototype.valueOf.call(target));
    case stringTag:
      return new Object(String.prototype.valueOf.call(target));
    case symbolTag:
      return new Object(Symbol.prototype.valueOf.call(target));
    case errorTag: 
    case dateTag:
      return new Ctor(target);
    case regexpTag:
      return handleRegExp(target);
    case funcTag:
      return handleFunc(target);
    default:
      return new Ctor(target);
  }
}

const deepClone = (target, map = new WeakMap()) => {
  if(!isObject(target)) 
    return target;
  let type = getType(target);
  let cloneTarget;
  if(!canTraverse[type]) {
    // 处理不能遍历的对象
    return handleNotTraverse(target, type);
  }else {
    // 这波操作相当关键，可以保证对象的原型不丢失！
    let ctor = target.constructor;
    cloneTarget = new ctor();
  }

  if(map.get(target)) 
    return target;
  map.set(target, true);

  if(type === mapTag) {
    //处理Map
    target.forEach((item, key) => {
      cloneTarget.set(deepClone(key, map), deepClone(item, map));
    })
  }
  
  if(type === setTag) {
    //处理Set
    target.forEach(item => {
      cloneTarget.add(deepClone(item, map));
    })
  }

  // 处理数组和对象
  for (let prop in target) {
    if (target.hasOwnProperty(prop)) {
        cloneTarget[prop] = deepClone(target[prop], map);
    }
  }
  return cloneTarget;
}
```
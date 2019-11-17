# ES6 特性快速扫盲

> 本文由 [yanglbme ](https://github.com/yanglbme)原创，首发于“[掘金](https://juejin.im/)”，禁止未授权转载。

`ES6`，也称 `ECMAScript2015`，对目前来说，它并不是 JavaScript 最新的标准，但是它已经广泛用于编程实践中。如果你还没用过 ES6，现在还不算太晚，跟我一起扫一遍吧。

## 1. forEach

### 场景

数组遍历。

```
var languages = ["JavaScript", "Python", "Rust", "Go"];
复制代码
```

### 传统实现

```
for (var i = 0; i < languages.length; ++i) {
    console.log(languages[i]);
}
复制代码
```

### ES6 实现

```
languages.forEach(function (e) {
    console.log(e);
})
复制代码
```

## 2. map

### 场景

数组每个元素翻倍，存储到另一个数组中。

```
var numbers = [1, 2, 4, 8, 16];
复制代码
```

### 传统实现

```
var doubled = [];

for (var i = 0; i < numbers.length; ++i) {
    doubled.push(numbers[i] * 2);
}

console.log(doubled);
复制代码
```

### ES6 实现

```
var doubled = numbers.map(function (e) {
    return e * 2;
});

console.log(doubled);
复制代码
```

## 3. filter

### 场景

过滤掉数组中元素 `weight` 小于 10 的元素（即：大于等于 10 才存放）。

```
var apples = [
    { id: 1, weight: 18 },
    { id: 2, weight: 9 },
    { id: 3, weight: 11 },
];
复制代码
```

### 传统实现

```
var filtered = [];

for (var i = 0; i < apples.length; ++i) {
    if (apples[i].weight >= 10) {
        filtered.push(apples[i]);
    }
}

console.log(filtered);
复制代码
```

### ES6 实现

```
var filtered = apples.filter(function (e) {
    return e.weight >= 10;
});

console.log(filtered);
复制代码
```

## 4. find

### 场景

查找 name 为 `bingo` 的元素，找到第一个符合条件的就返回。

```
var users = [
    { id: 1, name: "kevin" },
    { id: 2, name: "cicic" },
    { id: 3, name: "alexy" },
    { id: 4, name: "bingo" },
    { id: 5, name: "henry" },
];
复制代码
```

### 传统实现

```
var found;
for (var i = 0; i < users.length; ++i) {
    if (users[i].name === "bingo") {
        found = users[i];
        break;
    }
}

console.log(found);
复制代码
```

### ES6 实现

```
var found = users.find(function (e) {
    return e.name === "bingo";
});

console.log(found);
复制代码
```

## 5. every

### 场景

判断数组每本书的 year 是否都大于 2018（即：一假即假）。

```
var books = [
    { id: 1, name: "book1", year: 2019},
    { id: 2, name: "book2", year: 2017},
    { id: 3, name: "book3", year: 2019},
];
复制代码
```

### 传统实现

```
var every = true;

for (var i = 0; i < books.length; ++i) {
    if (books[i].year <= 2018) {
        every = false;
        break;
    }
}

console.log(every);
复制代码
```

### ES6 实现

```
var every = books.every(function (e) {
    return e.year > 2018;
});

console.log(every);
复制代码
```

## 6. some

### 场景

判断数组中是否存在 content 长度小于 3 的元素（即：一真即真）。

```
var inputs = [
    { id: 1, content: "掘金阴明"},
    { id: 2, content: "阿里小号"},
    { id: 3, content: "微信"},
    { id: 4, content: "头条大号"},
];
复制代码
```

### 传统实现

```
var some = false;

for (var i = 0; i < inputs.length; ++i) {
    if (inputs[i].content.length < 3) {
        some = true;
        break;
    }
}

console.log(some);
复制代码
```

### ES6 实现

```
var some = inputs.some(function (e) {
    return e.content.length < 3;
});

console.log(some);
复制代码
```

## 7. reduce

### 场景1

计算元素总和。

```
var nums = [2, 4, 8, 16];
复制代码
```

### 传统实现1

```
var sum = 0;

for (var i = 0; i < nums.length; ++i) {
    sum += nums[i];
}

console.log(sum);
复制代码
```

### ES6 实现1

```
var sum = nums.reduce(function (init, number) {
    return init + number;
});

console.log(sum);
复制代码
```

### 场景2

属性抽离，将数组中的每个元素的 color 属性抽离出来放到数组中。

```
var colors = [
    { id: 1, color: "red" },
    { id: 2, color: "white" },
    { id: 3, color: "yellow" },
];
复制代码
```

### 传统实现2

```
var newColors = [];

for (var i = 0; i < colors.length; ++i) {
    newColors.push(colors[i].color);
}

console.log(newColors);
复制代码
```

### ES6 实现2

```
var newColors = colors.reduce(function (init, color) {
    init.push(color.color);
    return init;
}, []);

console.log(newColors);
复制代码
```

> 注：reduce 可指定初始值，比如上面的 `[]`。

## 8. let/const

### var、let 区别

- var

```
function testVar() {
    var a = 10;
    if (true) {
        var a = 20;
        console.log(a);  // 20
    }
    // 在外部也生效
    console.log(a);   // 20
}

testVar();
复制代码
```

- let

```
function testLet() {
    let a = 10;
    if (true) {
        let a =  20;
        // 仅在当前作用域生效
        console.log(a);  // 20
    }
    console.log(a);  // 10
}

testVar();
复制代码
```

> 注：在前面的示例中，var 可改为 let。

### const

- 常量值不可修改

```
const x = 100;

// 这里会报错
x = 1000;  // Uncaught TypeError: Assignment to constant variable.
复制代码
```

- 数组元素可 push 添加

```
const arr = []
arr.push(1)
arr.push("2")

// 正常打印
console.log(arr)  // [1, "2"]
复制代码
```

## 9. 模板字符串

模板字符串使用反引号 ` (Tab 键上方的符号)来代替普通字符串中的用双引号和单引号。模板字符串可以包含特定语法 `${expression}` 的占位符。占位符中的表达式和周围的文本会一起传递给一个默认函数，该函数负责将所有的部分连接起来。

### 场景1

多行字符串。

### 传统实现1

```
// 这里是使用单引号 '
console.log('string text line 1\n' +
'string text line 2');
// "string text line 1
// string text line 2"
复制代码
```

### ES6 实现1

```
// 注意是使用反引号 `
console.log(`string text line 1
string text line 2`);
// "string text line 1
// string text line 2"
复制代码
```

### 场景2

插入表达式。

### 传统实现2

```
var a = 5;
var b = 10;
console.log('Fifteen is ' + (a + b) + ' and\nnot ' + (2 * a + b) + '.');
// "Fifteen is 15 and
// not 20."
复制代码
```

### ES6 实现2

```
var a = 5;
var b = 10;
console.log(`Fifteen is ${a + b} and
not ${2 * a + b}.`);
// "Fifteen is 15 and
// not 20."
复制代码
```

### 场景3

嵌套模板。

### 传统实现3

```
var classes = 'header'
classes += (isLargeScreen() ?
   '' : item.isCollapsed ?
     ' icon-expander' : ' icon-collapser');
复制代码
```

### ES6 实现3

```
const classes = `header ${ isLargeScreen() ? '' :
    (item.isCollapsed ? 'icon-expander' : 'icon-collapser') }`;
复制代码
```

## 10. 箭头函数

箭头函数的作用：

- 简化代码
- 改变 this 指向

### 场景1

两数相加。

### 传统实现1

```
const add = function(a, b) {
    return a + b;
}
复制代码
```

### ES6 实现

```
const add = (a, b) => a + b;
复制代码
```

### 场景2

数组元素翻倍。

```
const original = [2, 4, 8, 16];
const double = original.map(e => e * 2);
console.log(double);
复制代码
```

### 场景3

改变 this 指向。

### 传统实现3

```
const team = {
    members: ["bingo", "alex"],
    teamName: "ES6",
    teamSummary: function() {
        let self = this;
        return this.members.map(function (e) {
            return `${e}隶属于${self.teamName}小组`;
        })
    }
    // 或者使用bind绑定的方式
}
console.log(team.teamSummary())
复制代码
```

### ES6 实现3

```
const team = {
    members: ["bingo", "alex"],
    teamName: "ES6",
    teamSummary: function() {
        return this.members.map(e => `${e}隶属于${this.teamName}小组`);
    }
}
console.log(team.teamSummary())
复制代码
```

## 11. 增强对象字面量

增强对象字面量的一个作用就是：简化代码。在下面的示例中，注释的部分是简化前所使用。

```
function createBookShop(inventory) {
    return {
        // inventory: inventory,
        inventory,
        // inventoryValue: function() {
        inventoryValue() {
            return this.inventory.reduce((total, book) => total + book.price, 0);
        },
        // priceForTitle: function(title) {
        priceForTitle(title) {
            return this.inventory.find(book => book.title === title).price;
        }
    }
}

const inventory = [
    { title: "Vue.js", price: 20 },
    { title: "React.js", price: 19 },
]

const bookShop = createBookShop(inventory);
console.log(bookShop.inventoryValue());
console.log(bookShop.priceForTitle("Vue.js"));
复制代码
```

## 12. 函数参数默认值

```
function request(url, method="GET") {
    // ...
    console.log(method);
}

request("google.com", "POST");
复制代码
```

以上使用了 ES6 函数参数默认值，看一下上面的代码，若使用 Babel 进行转化，会得到什么：

```
"use strict";

function request(url) {
  var method = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : "GET";
  // ...
  console.log(method);
}

request("google.com", "POST");
复制代码
```

可以看到，使用 ES6 函数参数默认值，代码可以简化不少。

## 13. 展开运算符

Spread Operator，也叫展开运算符，有了它，我们可以更快、更便捷地操作数组。

```
// 不固定参数个数
function total(...numbers) {
    return numbers.reduce((sum, number) => sum + number, 0);
}

console.log(total(2, 4, 6, 8))  // 4个参数，20
console.log(total(1, 2, 3, 4, 5));  // 5个参数，15
复制代码
```

再看个示例：

```
let colors1 = ["red", "orange"];
let colors2 = ["blue", "white", "green"];

let totalColors = ["black", ...colors1, ...colors2];
console.log(totalColors);  // ["black", "red", "orange", "blue", "white", "green"]
复制代码
```

## 14. 解构

### 场景1

解构对象。

```
let user = {
    name: "Amy",
    age: 20,
    college: "szu"
}
复制代码
```

### 传统实现1

```
function print(user) {
    console.log(`${user.name}的年龄是${user.age}`);
}

print(user);  // Amy的年龄是20
复制代码
```

### ES6 实现1

```
function print({name, age, college}) {
    console.log(`${name}的年龄是${age}`);
}
print(user);  // Amy的年龄是20
复制代码
```

### 场景2

解构数组。

```
const names = ["Bingo", "Iris", "Alex"];

const [name1, name2, name3] = names;
console.log(name1, name2, name3);  // Bingo Iris Alex

// 返回数组个数
const { length } = names;
console.log(length);  // 3

// 结合展开运算符
const [name, ...rest] = names;
console.log(name);  // Bingo
console.log(rest);  // ["Iris", "Alex"]
复制代码
```

### 场景3

将数组转化为对象。

```
const points = [
    [4, 5],
    [1, 2],
    [3, 6],
];

let res = points.map(([x, y]) => {
    return { x, y };
})
console.log(res);  //  [{…}, {…}, {…}]
复制代码
```

## 15. 面向对象 class

### 场景

创建对象，并实现继承。

### 传统实现

```
// 首字母大写
function Car(options) {
    this.title = options.title;
}

Car.prototype.drive = function() {
    return "Vroom";
}

const car = new Car({ title: "BMW" });
console.log(car);
console.log(car.drive())

// 继承
function Toyota(options) {
    // 通过Car调用call()方法
    Car.call(this, options);
    this.color = options.color;
}

// 不容易理解
Toyota.prototype =  Object.create(Car.prototype);
Toyota.prototype.constructor = Toyota;

const toyota = new Toyota({ color: "red", title: "Focus" });
console.log(toyota.title);  // Focus
console.log(toyota.drive());  // Vroom
复制代码
```

### ES6 实现

```
class Car {
    constructor({ title }) {
        this.title = title;
    }

    drive() {
        return "Vroom";
    }
}

const car = new Car({ title: "BMW" });
console.log(car);
console.log(car.drive())

// 继承
class Toyota extends Car {
    constructor({ title, color }) {
        super(title);
        this.color = color;
    }
}

const toyota = new Toyota({ color: "red", title: "Focus" });
console.log(toyota)  // Focus
console.log(toyota.drive())  // Vroom
复制代码
```

## 16. generator 生成器

### 场景

斐波那契数列。

### 传统实现

```
function fib(max) {
    var a = 0, b = 1, arr = [0, 1];
    while (arr.length < max) {
        [a, b] = [b, a + b];
        arr.push(b);
    }
    return arr;
}

console.log(fib(5))
复制代码
```

### ES6 实现

```
// 注意，function后加多了个*
function* fib(max) {
    let a = 0, b = 1, n = 0;
    while (n < max) {
        // 使用yield关键字
        yield a;
        [a, b] = [b, a + b];
        ++n;
    }
    return;
}

// 使用for..of遍历
for (let x of fib(10)) {
    console.log(x);
}
复制代码
```

## 17. 新的数据结构 Map

Map 中的键可以是任何类型的，比如 `function()`、`{}`、`string` 等。

```
const map = new Map();

const key1 = 'some string',
      key2 = {},
      key3 = function() {};

// 为key设置value
map.set(key1, 'Value of key1');
map.set(key2, 'Value of key2');
map.set(key3, 'Value of key3');

console.log(map.get(key1))
console.log(map.get(key2))
console.log(map.get(key3))

// 使用for..of遍历
for (let [key, value] of map) {
    console.log(`${key}=>${value}`);
}

// 只获取key
for (let key of map.keys()) {
    console.log(key);
}

// 只获取value
for (let value of map.values()) {
    console.log(value);
}

// 使用forEach遍历
map.forEach((key, value) => {
    console.log(`${key}=>${value}`);
})
复制代码
```

## 18. 新的数据结构 Set

Set 可以存储任何类型的不重复数据。

```
const set = new Set();
set.add({name: "bingo"});
set.add(100);
set.add(true);
set.add(100);
console.log(set);  //  {{…}, 100, true}

console.log(set.size)  // 3
console.log(set.has(100));  // true
console.log(set.has({name: "bingo"})) //  匹配的是地址，false

set.delete(100);

// 使用for..of遍历
for (let item of set) {
    console.log(item);
}

// 使用forEach遍历
set.forEach(e => {
    console.log(e);
})

// 将set转换为array
const setArray = Array.from(set);
console.log(setArray);
复制代码
```

## 19. Promise

Promise 的三种状态：

- unresolved：等待任务完成；
- resolved：任务完成并且没有任何问题；
- rejected：任务完成，但是出现问题。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="481" height="311"></svg>)



### 场景

生成 0-2 之间的随机数，如果小于 1，则等待一段时间后返回成功，否则返回失败。

```
let promise = new Promise((resolve, reject) => {
    var timeout = Math.random() * 2;
    setTimeout(() => {
        if (timeout < 1) {
            resolve("success:ok");
        } else {
            reject("error:timeout");
        }
    }, timeout * 1000);
});

promise
    .then(res => console.log(`成功：${res}`))
    .catch(res => console.log(`失败：${res}`))
复制代码
```

> 成功时，执行 then；失败时，执行 catch。

## 20. fetch

### 场景

发送网络请求获取数据。

以下是一个基本的 fetch 请求，`fetch()` 返回一个包含响应结果的 promise（一个 Response 对象）：

```
fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(resp => resp.json())
    .then(res => console.log(res))
复制代码
```

`fetch()` 支持第二个可选参数，一个可以控制不同配置的 init 对象：

```
postData('https://jsonplaceholder.typicode.com/posts/1', {answer: 42})
  .then(data => console.log(data)) 
  .catch(error => console.error(error))

function postData(url, data) {
  return fetch(url, {
    cache: 'no-cache', 
    credentials: 'same-origin',
    headers: {
      'user-agent': 'Mozilla/4.0 MDN Example',
    },
    method: 'GET', // *GET, POST, PUT, DELETE, etc.
    mode: 'cors', // no-cors, cors, *same-origin
    redirect: 'follow', // manual, *follow, error
    referrer: 'no-referrer', // *client, no-referrer
  })
  .then(response => response.json()) // parses response to JSON
}
```

## 21. async/await(ES7)

`async/await` 是 JavaScript 中最终极的异步解决方案。

```
async function getUsers() {
    const resp = await fetch('https://jsonplaceholder.typicode.com/users');
    const data = await resp.json();
    return data;
}

// async 返回一个 Promise 对象，因此可以调用 then 获取结果。
getUsers().then(users => console.log(users));
```

以上，完。
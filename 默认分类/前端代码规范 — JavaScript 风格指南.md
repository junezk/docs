# 前端代码规范 — JavaScript 风格指南

## 前言

本文基于 github 项目 [airbnb/javascript](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fairbnb%2Fjavascript%23types) 翻译，也加入了一些个人理解。规范有利于我们更好的提高代码可读性，避免一些不必要的 bug。但是，并没有统一的标准和硬性要求，这里只是给大家提供一些参考，适合团队和自己的才是最好的。

**个人博客地址 🍹🍰 fe-code**

## 类型

- 1.1 基本类型

> 基本类型赋值时，应该直接使用类型的值

- `string`
- `number`
- `boolean`
- `null`
- `undefined`
- `symbol`

```
const foo = 1;
let bar = foo;

bar = 9;

console.log(foo, bar); // => 1,9
复制代码
```

- 复杂类型

> 复杂类型赋值其实是地址的引用

- `object`
- `array`
- `function`

```
const foo = [1, 2];
const bar = foo;

bar[0] = 9;

console.log(foo[0], bar[0]); // => 9, 9
// const 只能阻止引用类型地址的重新赋值
// 并不能保证引用类型的属性等不变
复制代码
```

## 状态的使用（原文为 Reference）

- 2.1 所有的赋值都用`const`，避免使用`var`. eslint: [`prefer-const`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fprefer-const.html), [`no-const-assign`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-const-assign.html)

> 尽量确保你的代码中的状态是可控范围内的，重复引用会出现难以理解的 bug 和代码。

```
// bad
var a = 1;
var b = 2;

// good
const a = 1;
const b = 2;
复制代码
```

- 2.2 如果你一定要对参数重新赋值，那就用`let`，而不是`var`. eslint: [`no-var`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-var.html)

> `let`是块级作用域，`var`是函数级作用域，同样是为了减少代码的不可控，减少 “意外”

```
// bad
var count = 1;
if (true) {
  count += 1;
}

// good, use the let.
let count = 1;
if (true) {
  count += 1;
}
复制代码
```

- 2.3 `let`、`const`都是块级作用域

```
// const 和 let 都只存在于它定义的那个块级作用域
{
  let a = 1;
  const b = 1;
}
console.log(a); // ReferenceError
console.log(b); // ReferenceError
复制代码
```

## 对象

- 3.1 使用字面值创建对象. eslint: [`no-new-object`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-new-object.html)

```
// bad
const item = new Object();

// good
const item = {};
复制代码
```

- 3.2 当创建一个带有动态属性名的对象时，将定义的所有属性放在对象的一个地方。

```
function getKey(k) {
  return `a key named ${k}`;
}

// bad
const obj = {
  id: 5,
  name: 'San Francisco',
};
obj[getKey('enabled')] = true;

// good getKey('enabled')是动态属性名
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};
复制代码
```

- 3.3 方法简写. eslint: [`object-shorthand`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fobject-shorthand.html)

```
// bad
const atom = {
  value: 1,

  addValue: function (value) {
    return atom.value + value;
  },
};

// good
const atom = {
  value: 1,

  // 对象的方法
  addValue(value) {
    return atom.value + value;
  },
};
复制代码
```

- 3.4 属性值缩写. eslint: [`object-shorthand`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fobject-shorthand.html)

```
const lukeSkywalker = 'Luke Skywalker';

// bad
const obj = {
  lukeSkywalker: lukeSkywalker,
};

// good
const obj = {
  lukeSkywalker
};
复制代码
```

- 3.5 将属性的缩写放在对象声明的开头。

```
const anakinSkywalker = 'Anakin Skywalker';
const lukeSkywalker = 'Luke Skywalker';

// bad
const obj = {
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  lukeSkywalker,
  episodeThree: 3,
  mayTheFourth: 4,
  anakinSkywalker,
};

// good
const obj = {
  lukeSkywalker,
  anakinSkywalker,
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  episodeThree: 3,
  mayTheFourth: 4,
};
复制代码
```

- 3.6 只对那些无效的标示使用引号 `''`. eslint: [`quote-props`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fquote-props.html)

> 一般来说，我们认为它在主观上更容易阅读。它改进了语法突出显示，并且更容易被JS引擎优化。

```
// bad
const bad = {
  'foo': 3,
  'bar': 4,
  'data-blah': 5,
};

// good
const good = {
  foo: 3,
  bar: 4,
  'data-blah': 5,
};
复制代码
```

- 3.7 不要直接调用`Object.prototype`上的方法，如`hasOwnProperty`, `propertyIsEnumerable`, `isPrototypeOf`。

> 在一些有问题的对象上， 这些方法可能会被屏蔽掉 - 如：`{ hasOwnProperty: false }` - 或这是一个空对象`Object.create(null)`

```
// bad
console.log(object.hasOwnProperty(key));

// good
console.log(Object.prototype.hasOwnProperty.call(object, key));

// best
const has = Object.prototype.hasOwnProperty; // 在模块作用内做一次缓存
/* or */
import has from 'has'; // https://www.npmjs.com/package/has
// ...
console.log(has.call(object, key));
复制代码
```

- 3.8 对象浅拷贝时，更推荐使用扩展运算符 `...`，而不是[`Object.assign`](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FObject%2Fassign)。解构赋值获取对象指定的几个属性时，推荐用 rest 运算符，也是 `...`。

```
// very bad
const original = { a: 1, b: 2 };
const copy = Object.assign(original, { c: 3 }); 
delete copy.a; // so does this  改变了 original

// bad
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original, { c: 3 }); // copy => { a: 1, b: 2, c: 3 }

// good
const original = { a: 1, b: 2 };
const copy = { ...original, c: 3 }; // copy => { a: 1, b: 2, c: 3 }

const { a, ...noA } = copy; // noA => { b: 2, c: 3 }
复制代码
```

## 数组

- 4.1 用字面量赋值。 eslint: [`no-array-constructor`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-array-constructor.html)

```
// bad
const items = new Array();

// good
const items = [];
复制代码
```

- 4.2 用[Array#push](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Fpush) 向数组中添加一个值而不是直接用下标。

```
const someStack = [];

// bad
someStack[someStack.length] = 'abracadabra';

// good
someStack.push('abracadabra');
复制代码
```

- 4.3 用扩展运算符做数组浅拷贝，类似上面的对象浅拷贝

```
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i += 1) {
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
复制代码
```

- 4.4 推荐用 `...` 运算符而不是[`Array.from`](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Ffrom)来将一个类数组转换成数组。

```
const foo = document.querySelectorAll('.foo');

// good
const nodes = Array.from(foo);

// best
const nodes = [...foo];
复制代码
```

- 4.5 用 [`Array.from`](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Ffrom) 去将一个类数组对象转成一个数组。

```
const arrLike = { 0: 'foo', 1: 'bar', 2: 'baz', length: 3 };

// bad
const arr = Array.prototype.slice.call(arrLike);

// good
const arr = Array.from(arrLike);
复制代码
```

- 4.6 用 [`Array.from`](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Ffrom) 而不是 `...` 运算符去迭代。 这样可以避免创建一个中间数组。

```
// bad
const baz = [...foo].map(bar);

// good
const baz = Array.from(foo, bar);
复制代码
```

- 4.7 在数组方法的回调函数中使用 return 语句。 如果函数体由一条返回一个表达式的语句组成， 并且这个表达式没有副作用， 这个时候可以忽略return，详见 [8.2](https://juejin.im/post/5d5d5197518825237330552d#arrows--implicit-return). eslint: [`array-callback-return`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Farray-callback-return)

```
// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});

// good 函数只有一个语句
[1, 2, 3].map(x => x + 1);

// bad 没有返回值， 导致在第一次迭代后acc 就变成undefined了
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item);
  acc[index] = flatten;
});

// good
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item);
  acc[index] = flatten;
  return flatten;
});

// bad
inbox.filter((msg) => {
  const { subject, author } = msg;
  if (subject === 'Mockingbird') {
    return author === 'Harper Lee';
  } else {
    return false;
  }
});

// good
inbox.filter((msg) => {
  const { subject, author } = msg;
  if (subject === 'Mockingbird') {
    return author === 'Harper Lee';
  }

  return false;
});
复制代码
```

- 4.8 如果一个数组有很多行，在数组的 `[` 后和 `]` 前换行。

```
// bad
const arr = [
  [0, 1], [2, 3], [4, 5],
];

const objectInArray = [{
  id: 1,
}, {
  id: 2,
}];

const numberInArray = [
  1, 2,
];

// good
const arr = [[0, 1], [2, 3], [4, 5]];

const objectInArray = [
  {
    id: 1,
  },
  {
    id: 2,
  },
];

const numberInArray = [
  1,
  2,
];
复制代码
```

## 解构

- 5.1 用对象的解构赋值来获取和使用对象某个或多个属性值。 eslint: [`prefer-destructuring`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fprefer-destructuring)

> 这样就不需要给这些属性创建临时/引用

```
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;

  return `${firstName} ${lastName}`;
}

// good
function getFullName(user) {
  const { firstName, lastName } = user;
  return `${firstName} ${lastName}`;
}

// best
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}
复制代码
```

- 5.2 数组解构.

```
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
复制代码
```

- 5.3 多个返回值用对象的解构，而不是数组解构。

> 不依赖于返回值的顺序，更可读

```
// bad
function processInput(input) {
  // 然后就是见证奇迹的时刻
  return [left, right, top, bottom];
}

const [left, __, top] = processInput(input);

// good
function processInput(input) {
  return { left, right, top, bottom };
}

const { left, top } = processInput(input);
复制代码
```

## 字符串

- 6.1 string 统一用单引号 `''` 。 eslint: [`quotes`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fquotes.html)

```
// bad
const name = "Capt. Janeway";

// bad - 模板应该包含插入文字或换行
const name = `Capt. Janeway`;

// good
const name = 'Capt. Janeway';
复制代码
```

- 6.2 不应该用 `+` 连接换行字符串。

> 不好用，且可读性差

```
// bad
const errorMessage = 'This is a super long error that was thrown because \
of Batman. When you stop to think about how Batman had anything to do \
with this, you would get nowhere \
fast.';

// bad
const errorMessage = 'This is a super long error that was thrown because ' +
  'of Batman. When you stop to think about how Batman had anything to do ' +
  'with this, you would get nowhere fast.';

// good
const errorMessage = 'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.';
复制代码
```

- 6.3 用字符串模板而不是 `+` 来拼接字符串。 eslint: [`prefer-template`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fprefer-template.html) [`template-curly-spacing`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Ftemplate-curly-spacing)

> 模板字符串更具可读性、语法简洁、字符串插入参数。

```
// bad
function sayHi(name) {
  return 'How are you, ' + name + '?';
}

// bad
function sayHi(name) {
  return ['How are you, ', name, '?'].join();
}

// bad
function sayHi(name) {
  return `How are you, ${ name }?`;
}

// good
function sayHi(name) {
  return `How are you, ${name}?`;
}
复制代码
```

- 6.4 永远不要在字符串中用`eval()`，漏洞太多。 eslint: [`no-eval`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-eval)

- [6.5](https://juejin.im/post/5d5d5197518825237330552d#strings--escaping) 不要使用不必要的转义字符。eslint: [`no-useless-escape`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-useless-escape)

> 反斜线可读性差，只在必要时使用

```
// bad
const foo = '\'this\' \i\s \"quoted\"';

// good
const foo = '\'this\' is "quoted"';

//best
const foo = `my name is '${name}'`;
复制代码
```

## 函数

- 7.1 用命名函数表达式而不是函数声明。eslint: [`func-style`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Ffunc-style)

> 函数声明作用域会提升，降低了代码可读性和可维护性。如果你发现一个函数又大又复杂，这个函数妨碍这个文件其他部分的理解性，这可能就是时候把这个函数单独抽成一个模块了。([Discussion](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fairbnb%2Fjavascript%2Fissues%2F794))

```
// bad
function foo() {
  // ...
}

// bad
const foo = function () {
  // ...
};

// good

const short = function longUniqueMoreDescriptiveLexicalFoo() {
  // ...
};
复制代码
```

- 7.2 把立即执行函数包裹在圆括号里。 eslint: [`wrap-iife`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fwrap-iife.html)

> 一个立即调用的函数表达式是一个单元 - 把它和他的调用者（圆括号）包裹起来。当然，现代模块开发中，你基本用不到。

```
// immediately-invoked function expression (IIFE)
(function () {
  console.log('Welcome to the Internet. Please follow me.');
}());
复制代码
```

- 7.3 不要在非函数块（if、while等等）内声明函数。而是把这个函数分配给一个变量。浏览器会允许你这样做，但浏览器解析方式不同，结果也许会有差异。【详见`no-loop-func`】 eslint: [`no-loop-func`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-loop-func.html)
- 7.4 **注意:** 在ECMA-262中 [块 `block`] 的定义是： 一系列的语句； 但是函数声明不是一个语句。 函数表达式是一个语句。

```
// bad
if (currentUser) {
  function test() {
    console.log('Nope.');
  }
}

// good
let test;
if (currentUser) {
  test = () => {
    console.log('Yup.');
  };
}
复制代码
```

- 7.5 永远不要用`arguments`命名参数。它的优先级高于每个函数作用域自带的 `arguments` 对象， 所以会导致函数自带的 `arguments` 值被覆盖。

```
// bad
function foo(name, options, arguments) {
  // ...
}

// good
function foo(name, options, args) {
  // ...
}
复制代码
```

- 7.6 优先使用rest语法`...`，而不是 `arguments`。 eslint: [`prefer-rest-params`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fprefer-rest-params)

> `...` 更明确你想用哪些参数。

```
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}
复制代码
```

- 7.8 使用默认参数语法，而不是在函数里对参数重新赋值。

```
// really bad
function handleThings(opts) {
  // 虽然你想这么写， 但是这个会带来一些细微的bug
  // 如果 opts 的值为 false, 它会被赋值为 {}
  opts = opts || {};
  // ...
}

// still bad
function handleThings(opts) {
  if (opts === void 0) {
    opts = {};
  }
  // ...
}

// good
function handleThings(opts = {}) {
  // ...
}
复制代码
```

- 7.8 使用默认参数时，需要避免副作用

```
var b = 1;
// bad
function count(a = b++) {
  console.log(a);
}
count();  // 1
count();  // 2
count(3); // 3
count();  // 3
// 很容易让人懵逼
复制代码
```

- 7.9 把默认参数赋值放在最后

```
// bad
function handleThings(opts = {}, name) {
  // ...
}

// good
function handleThings(name, opts = {}) {
  // ...
}
复制代码
```

- 7.10 不要用 Function 创建函数。 eslint: [`no-new-func`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-new-func)

```
// bad
var add = new Function('a', 'b', 'return a + b');

// still bad
var subtract = Function('a', 'b', 'return a - b');
复制代码
```

- 7.11 函数签名部分要有空格。eslint: [`space-before-function-paren`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fspace-before-function-paren) [`space-before-blocks`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fspace-before-blocks)

```
// bad
const f = function(){};
const g = function (){};
const h = function() {};

// good
const x = function () {};
const y = function a() {};
复制代码
```

- 7.12 永远不要改参数. eslint: [`no-param-reassign`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-param-reassign.html)

> 特别注意引用类型的操作，保证数据的不可变性

```
// bad
function f1(obj) {
  obj.key = 1;
};

// good
function f2(obj) {
  const key = Object.prototype.hasOwnProperty.call(obj, 'key') ? obj.key : 1;
};
复制代码
```

- 7.13 不要对参数重新赋值。 eslint: [`no-param-reassign`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-param-reassign.html)

```
// bad
function f1(a) {
  a = 1;
  // ...
}

function f2(a) {
  if (!a) { a = 1; }
  // ...
}

// good
function f3(a) {
  const b = a || 1;
  // ...
}

function f4(a = 1) {
  // ...
}
复制代码
```

- 7.14 活用 `...`。 eslint: [`prefer-spread`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fprefer-spread)

> Why? 这样更清晰，你不必提供上下文，而且你不能轻易地用`apply`来组成`new`

```
// bad
const x = [1, 2, 3, 4, 5];
console.log.apply(console, x);

// good
const x = [1, 2, 3, 4, 5];
console.log(...x);

// bad
new (Function.prototype.bind.apply(Date, [null, 2016, 8, 5]));

// good
new Date(...[2016, 8, 5]);
复制代码
```

- 7.15 多个参数的函数应该像这个指南里的其他多行代码写法一样： 每行只有一个参数，每行逗号结尾。

```
// bad
function foo(bar,
             baz,
             quux) {
  // ...
}

// good
function foo(
  bar,
  baz,
  quux,
) {
  // ...
}

// bad
console.log(foo,
  bar,
  baz);

// good
console.log(
  foo,
  bar,
  baz,
);
复制代码
```

## 箭头函数

- 8.1 如果要用匿名函数做回调，最好使用箭头函数 eslint: [`prefer-arrow-callback`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fprefer-arrow-callback.html), [`arrow-spacing`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Farrow-spacing.html)

> 它创建了一个在上下文中执行的函数，这通常是您想要的，并且是一种更简洁的语法。

```
// bad
[1, 2, 3].map(function (x) {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
复制代码
```

- 8.2 如果函数体由一个没有副作用的[表达式](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FJavaScript%2FGuide%2FExpressions_and_Operators%23Expressions)的单个语句组成，去掉大括号和 return。否则，保留大括号且使用 `return` 语句。 eslint: [`arrow-parens`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Farrow-parens.html), [`arrow-body-style`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Farrow-body-style.html)

```
// bad
[1, 2, 3].map(number => {
  const nextNumber = number + 1;
  `A string containing the ${nextNumber}.`;
});

// good
[1, 2, 3].map(number => `A string containing the ${number}.`);

// good
[1, 2, 3].map((number) => {
  const nextNumber = number + 1;
  return `A string containing the ${nextNumber}.`;
});

// good
[1, 2, 3].map((number, index) => ({
  [index]: number
}));

// 表达式有副作用就不要用隐式返回
function foo(callback) {
  const val = callback();
  if (val === true) {
    // Do something if callback returns true
  }
}

let bool = false;

// bad
foo(() => bool = true);

// good
foo(() => {
  bool = true;
});
复制代码
```

- 8.3 如果表达式有多行，首尾放在圆括号里更可读。

```
// bad
['get', 'post', 'put'].map(httpMethod => Object.prototype.hasOwnProperty.call(
    httpMagicObjectWithAVeryLongName,
    httpMethod
  )
);

// good
['get', 'post', 'put'].map(httpMethod => (
  Object.prototype.hasOwnProperty.call(
    httpMagicObjectWithAVeryLongName,
    httpMethod
  )
));
复制代码
```

- 8.4 为了清晰和一致，始终在参数周围加上括号 eslint: [`arrow-parens`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Farrow-parens.html)

```
// bad
[1, 2, 3].map((x) => x * x);

// good
[1, 2, 3].map(x => x * x);

// good
[1, 2, 3].map(number => (
  `A long string with the ${number}. It’s so long that we don’t want it to take up space on the .map line!`
));

// bad
[1, 2, 3].map(x => {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
复制代码
```

- [8.5](https://juejin.im/post/5d5d5197518825237330552d#arrows--confusing) 避免箭头函数语法 `=>` 和比较操作符 `<=, >=` 混淆. eslint: [`no-confusing-arrow`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-confusing-arrow)

```
// bad
const itemHeight = item => item.height > 256 ? item.largeSize : item.smallSize;

// bad
const itemHeight = (item) => item.height > 256 ? item.largeSize : item.smallSize;

// good
const itemHeight = item => (item.height > 256 ? item.largeSize : item.smallSize);

// good
const itemHeight = (item) => {
  const { height, largeSize, smallSize } = item;
  return height > 256 ? largeSize : smallSize;
};
复制代码
```

- 8.6 使用隐式返回时强制约束函数体在箭头后面。 eslint: [`implicit-arrow-linebreak`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fimplicit-arrow-linebreak)

```
// bad
(foo) =>
  bar;

(foo) =>
  (bar);

// good
(foo) => bar;
(foo) => (bar);
(foo) => (
   bar
)
复制代码
```

## 类和构造函数

- 9.1 始终用`class`，避免直接操作`prototype`

```
// bad
function Queue(contents = []) {
  this.queue = [...contents];
}
Queue.prototype.pop = function () {
  const value = this.queue[0];
  this.queue.splice(0, 1);
  return value;
};


// good
class Queue {
  constructor(contents = []) {
    this.queue = [...contents];
  }
  pop() {
    const value = this.queue[0];
    this.queue.splice(0, 1);
    return value;
  }
}
复制代码
```

- 9.2 使用`extends`实现继承

> 内置的方法来继承原型，而不会破坏 `instanceof`

```
// bad
const inherits = require('inherits');
function PeekableQueue(contents) {
  Queue.apply(this, contents);
}
inherits(PeekableQueue, Queue);
PeekableQueue.prototype.peek = function () {
  return this._queue[0];
}

// good
class PeekableQueue extends Queue {
  peek() {
    return this._queue[0];
  }
}
复制代码
```

- 9.3 方法可以返回`this`来实现方法链

```
// bad
Jedi.prototype.jump = function () {
  this.jumping = true;
  return true;
};

Jedi.prototype.setHeight = function (height) {
  this.height = height;
};

const luke = new Jedi();
luke.jump(); // => true
luke.setHeight(20); // => undefined

// good
class Jedi {
  jump() {
    this.jumping = true;
    return this;
  }

  setHeight(height) {
    this.height = height;
    return this;
  }
}

const luke = new Jedi();

luke.jump()
  .setHeight(20);
复制代码
```

- 9.4 允许写一个自定义的 toString() 方法，但是要保证它是可以正常工作且没有副作用

```
class Jedi {
  constructor(options = {}) {
    this.name = options.name || 'no name';
  }

  getName() {
    return this.name;
  }

  toString() {
    return `Jedi - ${this.getName()}`;
  }
}
复制代码
```

- 9.5 如果没有特殊说明，类有默认的构造方法。不用特意写一个空的构造函数或只是代表父类的构造函数。 eslint: [`no-useless-constructor`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-useless-constructor)

```
// bad
class Jedi {
  constructor() {}

  getName() {
    return this.name;
  }
}

// bad
class Rey extends Jedi {
  // 这种构造函数是不需要写的
  constructor(...args) {
    super(...args);
  }
}

// good
class Rey extends Jedi {
  constructor(...args) {
    super(...args);
    this.name = 'Rey';
  }
}
复制代码
```

- 9.6 避免重复类的成员。 eslint: [`no-dupe-class-members`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-dupe-class-members)

> 重复类成员会默默的执行最后一个，有重复肯定就是一个错误

```
// bad
class Foo {
  bar() { return 1; }
  bar() { return 2; }
}

// good
class Foo {
  bar() { return 1; }
}

// good
class Foo {
  bar() { return 2; }
}
复制代码
```

## 模块

- 10.1 在非标准模块系统上使用(`import`/`export`)。或者随时换成其他的首选模块系统。

```
// bad
const AirbnbStyleGuide = require('./AirbnbStyleGuide');
module.exports = AirbnbStyleGuide.es6;

// ok
import AirbnbStyleGuide from './AirbnbStyleGuide';
export default AirbnbStyleGuide.es6;

// best
import { es6 } from './AirbnbStyleGuide';
export default es6;
复制代码
```

- 10.2 不要用 import * 这种通配符

```
// bad
import * as AirbnbStyleGuide from './AirbnbStyleGuide';

// good
import AirbnbStyleGuide from './AirbnbStyleGuide';
复制代码
```

- 10.3 不要直接从 import 中直接 export

> 看起来简洁，但是影响可读性

```
// bad
// filename es6.js
export { es6 as default } from './AirbnbStyleGuide';

// good
// filename es6.js
import { es6 } from './AirbnbStyleGuide';
export default es6;
复制代码
```

- 10.4 一个入口只 import 一次。 eslint: [`no-duplicate-imports`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-duplicate-imports)

> Why? 从同一个路径下import多行会使代码难以维护

```
// bad
import foo from 'foo';
// … some other imports … //
import { named1, named2 } from 'foo';

// good
import foo, { named1, named2 } from 'foo';

// good
import foo, {
  named1,
  named2,
} from 'foo';
复制代码
```

- 10.5 不要导出可变的绑定 eslint: [`import/no-mutable-exports`](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fbenmosher%2Feslint-plugin-import%2Fblob%2Fmaster%2Fdocs%2Frules%2Fno-mutable-exports.md)

> 尽量减少状态，保证数据的不可变性。虽然在某些场景下可能需要这种技术，但总的来说应该导出常量。

```
// bad
let foo = 3;
export { foo }

// good
const foo = 3;
export { foo }
复制代码
```

- 10.6 在只有一个导出的模块里，用 `export default` 更好。 eslint: [`import/prefer-default-export`](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fbenmosher%2Feslint-plugin-import%2Fblob%2Fmaster%2Fdocs%2Frules%2Fprefer-default-export.md)

> 鼓励使用更多文件，每个文件只做一件事情并导出，这样可读性和可维护性更好。

```
// bad
export function foo() {}

// good
export default function foo() {}
复制代码
```

- 10.7 `import` 放在其他所有语句之前。 eslint: [`import/first`](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fbenmosher%2Feslint-plugin-import%2Fblob%2Fmaster%2Fdocs%2Frules%2Ffirst.md)

> 防止意外行为。

```
// bad
import foo from 'foo';
foo.init();

import bar from 'bar';

// good
import foo from 'foo';
import bar from 'bar';

foo.init();
复制代码
```

- 10.8 多行 import 应该缩进，就像多行数组和对象字面量

```
// bad
import {longNameA, longNameB, longNameC, longNameD, longNameE} from 'path';

// good
import {
  longNameA,
  longNameB,
  longNameC,
  longNameD,
  longNameE,
} from 'path';
复制代码
```

- 10.9 在 import 语句里不允许 Webpack loader 语法 eslint: [`import/no-webpack-loader-syntax`](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fbenmosher%2Feslint-plugin-import%2Fblob%2Fmaster%2Fdocs%2Frules%2Fno-webpack-loader-syntax.md)

> 最好是在`webpack.config.js`里写

```
// bad
import fooSass from 'css!sass!foo.scss';
import barCss from 'style!css!bar.css';

// good
import fooSass from 'foo.scss';
import barCss from 'bar.css';
复制代码
```

## 迭代器和生成器

- 11.1 不要用迭代器。用 JavaScript 高阶函数代替`for-in`、 `for-of`。 eslint: [`no-iterator`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-iterator.html)[`no-restricted-syntax`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-restricted-syntax)

> 不可变原则，处理纯函数的返回值比处理副作用更容易。

> 数组的迭代方法： `map()` / `every()` / `filter()` / `find()` / `findIndex()` / `reduce()` / `some()` / ... , 对象的处理方法 ：`Object.keys()` / `Object.values()` / `Object.entries()`去产生一个数组， 这样你就能去遍历对象了。

```
const numbers = [1, 2, 3, 4, 5];

// bad
let sum = 0;
for (let num of numbers) {
  sum += num;
}
sum === 15;

// good
let sum = 0;
numbers.forEach(num => sum += num);
sum === 15;

// best (use the functional force)
const sum = numbers.reduce((total, num) => total + num, 0);
sum === 15;

// bad
const increasedByOne = [];
for (let i = 0; i < numbers.length; i++) {
  increasedByOne.push(numbers[i] + 1);
}

// good
const increasedByOne = [];
numbers.forEach(num => increasedByOne.push(num + 1));

// best (keeping it functional)
const increasedByOne = numbers.map(num => num + 1);
复制代码
```

- 11.2 现在不要用 generator

> 兼容性不好

- 11.3 如果你一定要用，或者你忽略[我们的建议](https://juejin.im/post/5d5d5197518825237330552d#generators--nope), 请确保它们的函数签名之间的空格是正确的。 eslint: [`generator-star-spacing`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fgenerator-star-spacing)

> `function` 和 `*` 是同一概念，关键字 `*`不是`function`的修饰符，`function*`是一个和`function`不一样的独特结构

```
// bad
function * foo() {
  // ...
}

// bad
const bar = function * () {
  // ...
}

// bad
const baz = function *() {
  // ...
}

// bad
const quux = function*() {
  // ...
}

// bad
function*foo() {
  // ...
}

// bad
function *foo() {
  // ...
}

// very bad
function
*
foo() {
  // ...
}

// very bad
const wat = function
*
() {
  // ...
}

// good
function* foo() {
  // ...
}

// good
const foo = function* () {
  // ...
}
复制代码
```

## 属性

- 12.1 访问属性时使用点符号. eslint: [`dot-notation`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fdot-notation.html)

```
const luke = {
  jedi: true,
  age: 28,
};

// bad
const isJedi = luke['jedi'];

// good
const isJedi = luke.jedi;
复制代码
```

- 12.2 获取的属性是变量时用方括号`[]`

```
const luke = {
  jedi: true,
  age: 28,
};

function getProp(prop) {
  return luke[prop];
}

const isJedi = getProp('jedi');
复制代码
```

- 12.3 做幂运算时用幂操作符 `**` 。 eslint: [`no-restricted-properties`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-restricted-properties).

```
// bad
const binary = Math.pow(2, 10);

// good
const binary = 2 ** 10;
复制代码
```

## 变量

- 13.1 始终用 `const` 或 `let` 声明变量。如果你不想遇到一对变量提升、全局变量的 bug 的话。 eslint: [`no-undef`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-undef) [`prefer-const`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fprefer-const)

```
// bad
superPower = new SuperPower();

// good
const superPower = new SuperPower();
复制代码
```

- 13.2 每个变量单独用一个 `const` 或 `let`。 eslint: [`one-var`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fone-var.html)

```
// bad
const items = getItems(),
    goSportsTeam = true,
    dragonball = 'z';

// bad
// (compare to above, and try to spot the mistake)
const items = getItems(),
    goSportsTeam = true;
    dragonball = 'z';

// good
const items = getItems();
const goSportsTeam = true;
const dragonball = 'z';
复制代码
```

- 13.3 `const`放一起，`let`放一起

> 新变量依赖之前的变量或常量时，是有帮助的

```
// bad
let i, len, dragonball,
    items = getItems(),
    goSportsTeam = true;

// bad
let i;
const items = getItems();
let dragonball;
const goSportsTeam = true;
let len;

// good
const goSportsTeam = true;
const items = getItems();
let dragonball;
let i;
let length;
复制代码
```

- 13.4 变量声明放在合理的位置

```
// bad - unnecessary function call
function checkName(hasName) {
  const name = getName();

  if (hasName === 'test') {
    return false;
  }

  if (name === 'test') {
    this.setName('');
    return false;
  }

  return name;
}

// good
function checkName(hasName) {
  if (hasName === 'test') {
    return false;
  }

  // 在需要的时候分配
  const name = getName();

  if (name === 'test') {
    this.setName('');
    return false;
  }

  return name;
}
复制代码
```

- 13.5 不要使用连续变量分配。 eslint: [`no-multi-assign`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-multi-assign)

> Why? 链接变量分配创建隐式全局变量。

```
// bad
(function example() {
  // JavaScript 将其解释为
  // let a = ( b = ( c = 1 ) );
  // let 只对变量 a 起作用; 变量 b 和 c 都变成了全局变量
  let a = b = c = 1;
}());

console.log(a); // undefined
console.log(b); // 1
console.log(c); // 1

// good
(function example() {
  let a = 1;
  let b = a;
  let c = a;
}());

console.log(a); // undefined
console.log(b); // undefined
console.log(c); // undefined

// `const` 也一样
复制代码
```

- 13.6 不要使用一元递增递减运算符（`++`， `--`）. eslint [`no-plusplus`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-plusplus)

> 根据 eslint 文档，一元递增和递减语句受到自动分号插入的影响，并且可能会导致应用程序中的值递增或递减的静默错误。 使用num += 1 而不是 num++ 或代替语句来改变你的值也更具表现力。禁止一元递增和递减语句也会阻止您无意中预先递增/预递减值，从而减少程序出现意外行为。

```
  // bad

  let array = [1, 2, 3];
  let num = 1;
  num++;
  --num;

  let sum = 0;
  let truthyCount = 0;
  for(let i = 0; i < array.length; i++){
    let value = array[i];
    sum += value;
    if (value) {
      truthyCount++;
    }
  }

  // good

  let array = [1, 2, 3];
  let num = 1;
  num += 1;
  num -= 1;

  const sum = array.reduce((a, b) => a + b, 0);
  const truthyCount = array.filter(Boolean).length;
复制代码
```

- 13.7 避免在 `=` 前/后换行。 如果你的语句超出 [`max-len`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fmax-len.html)， 那就用`()`把这个值包起来再换行。 eslint [`operator-linebreak`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Foperator-linebreak.html).

```
// bad
const foo =
  superLongLongLongLongLongLongLongLongFunctionName();

// bad
const foo
  = 'superLongLongLongLongLongLongLongLongString';

// good
const foo = (
  superLongLongLongLongLongLongLongLongFunctionName()
);

// good
const foo = 'superLongLongLongLongLongLongLongLongString';
复制代码
```

- 13.8 不允许有未使用的变量。 eslint: [`no-unused-vars`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-unused-vars)

```
// bad

var some_unused_var = 42;

// 定义了没有使用
var y = 10;
y = 5;

// 不会将用于修改自身的读取视为已使用
var z = 0;
z = z + 1;

// 参数定义了但未使用
function getX(x, y) {
    return x;
}

// good
function getXPlusY(x, y) {
  return x + y;
}

var x = 1;
var y = a + 2;

alert(getXPlusY(x, y));

// 'type' 即使没有使用也可以被忽略， 因为这个有一个 rest 取值的属性。
// 这是从对象中抽取一个忽略特殊字段的对象的一种形式
var { type, ...coords } = data;
// 'coords' 现在就是一个没有 'type' 属性的 'data' 对象
复制代码
```

## 提升

- 14.1 var 声明被提升。const 和 let 声明被赋予一个所谓的新概念[Temporal Dead Zones (TDZ)](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FJavaScript%2FReference%2FStatements%2Flet%23Temporal_dead_zone_and_errors_with_let)。 重要的是要知道为什么 [typeof不再安全](https://link.juejin.im/?target=http%3A%2F%2Fes-discourse.com%2Ft%2Fwhy-typeof-is-no-longer-safe%2F15).

```
function example() {
  console.log(notDefined); // => throws a ReferenceError
}

// 在变量声明之前使用会正常输出，是因为变量声明提升，值没有。
function example() {
  console.log(declaredButNotAssigned); // => undefined
  var declaredButNotAssigned = true;
}

// 表现同上
function example() {
  let declaredButNotAssigned;
  console.log(declaredButNotAssigned); // => undefined
  declaredButNotAssigned = true;
}

// 用 const， let 不会发生提升
function example() {
  console.log(declaredButNotAssigned); // => throws a ReferenceError
  console.log(typeof declaredButNotAssigned); // => throws a ReferenceError
  const declaredButNotAssigned = true;
}
复制代码
```

- 14.2 匿名函数表达式和 `var` 情况相同

```
function example() {
  console.log(anonymous); // => undefined

  anonymous(); // => TypeError anonymous is not a function

  var anonymous = function () {
    console.log('anonymous function expression');
  };
}
复制代码
```

- 14.3 已命名的函数表达式提升他的变量名，而不是函数名或函数体

```
function example() {
  console.log(named); // => undefined

  named(); // => TypeError named is not a function

  superPower(); // => ReferenceError superPower is not defined

  var named = function superPower() {
    console.log('Flying');
  };
}

// 函数名和变量名相同也是一样
function example() {
  console.log(named); // => undefined

  named(); // => TypeError named is not a function

  var named = function named() {
    console.log('named');
  };
}
复制代码
```

- 14.4 函数声明则提升了函数名和函数体

```
function example() {
  superPower(); // => Flying

  function superPower() {
    console.log('Flying');
  }
}
复制代码
```

- 更多信息前往[JavaScript Scoping & Hoisting](https://link.juejin.im/?target=http%3A%2F%2Fwww.adequatelygood.com%2F2010%2F2%2FJavaScript-Scoping-and-Hoisting%2F) by [Ben Cherry](https://link.juejin.im/?target=http%3A%2F%2Fwww.adequatelygood.com%2F).

## 比较和相等

- 15.1 使用 `===` 和 `!==` 而不是 `==` 和 `!=`. eslint: [`eqeqeq`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Feqeqeq.html)
- 15.2 `if` 等条件语句使用强制 `ToBoolean` 抽象方法来评估它们的表达式，并且始终遵循以下简单规则：
- **Objects** => **true**
- **Undefined** => **false**
- **Null** => **false**
- **Booleans** => **the value of the boolean**
- **Numbers**
  - **+0, -0, or NaN** => **false**
  - 其他 => **true**
- **Strings**
  - `''` => **false**
  - 其他 => **true**

```
if ([0] && []) {
  // true
  // 数组（即使是空数组）是对象，对象会计算成 true
}
复制代码
```

- 15.3 布尔值比较可以省略，但是字符串和数字要显示比较

```
// bad
if (isValid === true) {
  // ...
}

// good
if (isValid) {
  // ...
}

// bad
if (name) {
  // ...
}

// good
if (name !== '') {
  // ...
}

// bad
if (collection.length) {
  // ...
}

// good
if (collection.length > 0) {
  // ...
}
复制代码
```

- 15.4 `switch case` 中，在 `case` 和 `default` 分句里用大括号创建一个块(如：`let`, `const`, `function`, and `class`). eslint rules: [`no-case-declarations`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-case-declarations.html).

> 词汇声明在整个 switch 块中都是可见的，但只有在分配时才会被初始化，这只有在 case 达到时才会发生。当多个 case 子句尝试定义相同的事物时，会出现问题。

```
// bad
switch (foo) {
  case 1:
    let x = 1;
    break;
  case 2:
    const y = 2;
    break;
  case 3:
    function f() {
      // ...
    }
    break;
  default:
    class C {}
}

// good
switch (foo) {
  case 1: {
    let x = 1;
    break;
  }
  case 2: {
    const y = 2;
    break;
  }
  case 3: {
    function f() {
      // ...
    }
    break;
  }
  case 4:
    bar();
    break;
  default: {
    class C {}
  }
}
复制代码
```

- 15.5 三元表达式不应该嵌套，通常是单行表达式。

eslint rules: [`no-nested-ternary`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-nested-ternary.html).

```
// bad
const foo = maybe1 > maybe2
  ? "bar"
  : value1 > value2 ? "baz" : null;

// better
const maybeNull = value1 > value2 ? 'baz' : null;

const foo = maybe1 > maybe2
  ? 'bar'
  : maybeNull;

// best
const maybeNull = value1 > value2 ? 'baz' : null;

const foo = maybe1 > maybe2 ? 'bar' : maybeNull;
复制代码
```

- 15.7 避免不需要的三元表达式

eslint rules: [`no-unneeded-ternary`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-unneeded-ternary.html).

```
// bad
const foo = a ? a : b;
const bar = c ? true : false;
const baz = c ? false : true;

// good
const foo = a || b;
const bar = !!c;
const baz = !c;
复制代码
```

- 15.8 混合操作符时，要放在 `()` 里，只有当它们是标准的算术运算符(`+`, `-`, `*`, & `/`)， 并且它们的优先级显而易见时，可以不用。 eslint: [`no-mixed-operators`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-mixed-operators.html)

```
// bad
const foo = a && b < 0 || c > 0 || d + 1 === 0;

// bad
const bar = a ** b - 5 % d;

// bad
if (a || b && c) {
  return d;
}

// good
const foo = (a && b < 0) || c > 0 || (d + 1 === 0);

// good
const bar = (a ** b) - (5 % d);

// good
if (a || (b && c)) {
  return d;
}

// good
const bar = a + b / c * d;
复制代码
```

## 块

- 16.1 用大括号 `{}` 包裹多行代码块。 eslint: [`nonblock-statement-body-position`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fnonblock-statement-body-position)

```
// bad
if (test)
  return false;

// good
if (test) return false;

// good
if (test) {
  return false;
}

// bad
function foo() { return false; }

// good
function bar() {
  return false;
}
复制代码
```

- 16.2 `else` 和 `if` 的大括号保持在一行。 eslint: [`brace-style`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fbrace-style.html)

```
// bad
if (test) {
  thing1();
  thing2();
}
else {
  thing3();
}

// good
if (test) {
  thing1();
  thing2();
} else {
  thing3();
}
复制代码
```

- 16.3 如果 `if` 语句都要用 `return` 返回， 那后面的 `else` 就不用写了。 如果 `if` 块中包含 `return`， 它后面的 `else if` 块中也包含了 `return`， 这个时候就可以把 `else if` 拆开。 eslint: [`no-else-return`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-else-return)

```
// bad
function foo() {
  if (x) {
    return x;
  } else {
    return y;
  }
}

// bad
function cats() {
  if (x) {
    return x;
  } else if (y) {
    return y;
  }
}

// bad
function dogs() {
  if (x) {
    return x;
  } else {
    if (y) {
      return y;
    }
  }
}

// good
function foo() {
  if (x) {
    return x;
  }

  return y;
}

// good
function cats() {
  if (x) {
    return x;
  }

  if (y) {
    return y;
  }
}

// good
function dogs(x) {
  if (x) {
    if (z) {
      return y;
    }
  } else {
    return z;
  }
}
复制代码
```

## 控制

- 17.1 当你的控制语句 `if`, `while` 等太长或者超过最大长度限制的时候，把每个判断条件放在单独一行里，逻辑运算符放在行首。

```
// bad
if ((foo === 123 || bar === 'abc') && doesItLookGoodWhenItBecomesThatLong() && isThisReallyHappening()) {
  thing1();
}

// bad
if (foo === 123 &&
  bar === 'abc') {
  thing1();
}

// bad
if (foo === 123
  && bar === 'abc') {
  thing1();
}

// bad
if (
  foo === 123 &&
  bar === 'abc'
) {
  thing1();
}

// good
if (
  foo === 123
  && bar === 'abc'
) {
  thing1();
}

// good
if (
  (foo === 123 || bar === 'abc')
  && doesItLookGoodWhenItBecomesThatLong()
  && isThisReallyHappening()
) {
  thing1();
}

// good
if (foo === 123 && bar === 'abc') {
  thing1();
}
复制代码
```

- 17.2 不要用选择操作符代替控制语句。

```
// bad
!isRunning && startRunning();

// good
if (!isRunning) {
  startRunning();
}
复制代码
```

## 注释

- 18.1 多行注释用 `/** ... */`

```
// bad
// make() returns a new element
// based on the passed in tag name
//
// @param {String} tag
// @return {Element} element
function make(tag) {

  // ...

  return element;
}

// good
/**
 * make() returns a new element
 * based on the passed-in tag name
 */
function make(tag) {

  // ...

  return element;
}
复制代码
```

- 18.2 单行注释用`//`，将单行注释放在被注释区域上方。如果注释不是在第一行，就在注释前面加一个空行

```
// bad
const active = true;  // is current tab

// good
// is current tab
const active = true;

// bad
function getType() {
  console.log('fetching type...');
  // set the default type to 'no type'
  const type = this._type || 'no type';

  return type;
}

// good
function getType() {
  console.log('fetching type...');

  // set the default type to 'no type'
  const type = this._type || 'no type';

  return type;
}

// also good
function getType() {
  // set the default type to 'no type'
  const type = this._type || 'no type';

  return type;
}
复制代码
```

- 18.3 所有注释开头加一个空格，方便阅读。 eslint: [`spaced-comment`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fspaced-comment)

```js
// bad
//is current tab
const active = true;

// good
// is current tab
const active = true;

// bad
/**
 *make() returns a new element
 *based on the passed-in tag name
 */
function make(tag) {

  // ...

  return element;
}

// good
/**
 * make() returns a new element
 * based on the passed-in tag name
 */
function make(tag) {

  // ...

  return element;
}
```

- 18.4 在注释前加上 `FIXME' 或`TODO` 前缀， 这有助于其他开发人员快速理解你指出的问题， 或者您建议的问题的解决方案。

```js
class Calculator extends Abacus {
  constructor() {
    super();

    // FIXME: shouldn't use a global here
    total = 0;
  }
}

class Calculator extends Abacus {
  constructor() {
    super();

    // TODO: total should be configurable by an options param
    this.total = 0;
  }
}
```

## 空格

- 19.1 Tab 使用两个空格（或者 4 个，你开心就好，但是团队统一是必须的）。 eslint: [`indent`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Findent.html)

```js
// bad
function foo() {
  const name;
}

// bad
function bar() {
  const name;
}

// good
function baz() {
  const name;
}
```

- 19.2 在大括号 `{}` 前空一格。 eslint: [`space-before-blocks`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fspace-before-blocks.html)

```js
// bad
function test(){
  console.log('test');
}

// good
function test() {
  console.log('test');
}

// bad
dog.set('attr',{
  age: '1 year',
  breed: 'Bernese Mountain Dog',
});

// good
dog.set('attr', {
  age: '1 year',
  breed: 'Bernese Mountain Dog',
});
```

- 19.3 在控制语句 `if`, `while` 等的圆括号前空一格。在函数调用和定义时，函数名和圆括号之间不空格。 eslint: [`keyword-spacing`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fkeyword-spacing.html)

```js
// bad
if(isJedi) {
  fight ();
}

// good
if (isJedi) {
  fight();
}

// bad
function fight () {
  console.log ('Swooosh!');
}

// good
function fight() {
  console.log('Swooosh!');
}
```

- 19.4 用空格来隔开运算符。 eslint: [`space-infix-ops`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fspace-infix-ops.html)

```js
// bad
const x=y+5;

// good
const x = y + 5;
```

- 19.5 文件结尾空一行. eslint: [`eol-last`](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Feslint%2Feslint%2Fblob%2Fmaster%2Fdocs%2Frules%2Feol-last.md)

```js
// bad
import { es6 } from './AirbnbStyleGuide';
  // ...
export default es6;

// bad
import { es6 } from './AirbnbStyleGuide';
  // ...
export default es6;↵
↵
// good
import { es6 } from './AirbnbStyleGuide';
  // ...
export default es6;↵
```

- 19.6 当出现长的方法链（一般超过两个的时候）时换行。用点开头强调该行是一个方法调用，而不是一个新的语句。eslint: [`newline-per-chained-call`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fnewline-per-chained-call) [`no-whitespace-before-property`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-whitespace-before-property)

```js
// bad
$('#items').find('.selected').highlight().end().find('.open').updateCount();

// bad
$('#items').
  find('.selected').
    highlight().
    end().
  find('.open').
    updateCount();

// good
$('#items')
  .find('.selected')
    .highlight()
    .end()
  .find('.open')
    .updateCount();

// bad
const leds = stage.selectAll('.led').data(data).enter().append('svg:svg').classed('led', true)
    .attr('width', (radius + margin) * 2).append('svg:g')
    .attr('transform', `translate(${radius + margin},${radius + margin})`)
    .call(tron.led);

// good
const leds = stage.selectAll('.led')
    .data(data)
  .enter().append('svg:svg')
    .classed('led', true)
    .attr('width', (radius + margin) * 2)
  .append('svg:g')
    .attr('transform', `translate(${radius + margin},${radius + margin})`)
    .call(tron.led);

// good
const leds = stage.selectAll('.led').data(data);
```

- 19.7 在一个代码块之后，下一条语句之前空一行。

```js
// bad
if (foo) {
  return bar;
}
return baz;

// good
if (foo) {
  return bar;
}

return baz;

// bad
const obj = {
  foo() {
  },
  bar() {
  },
};
return obj;

// good
const obj = {
  foo() {
  },

  bar() {
  },
};

return obj;

// bad
const arr = [
  function foo() {
  },
  function bar() {
  },
];
return arr;

// good
const arr = [
  function foo() {
  },

  function bar() {
  },
];

return arr;
```

- 19.8 不要故意留一些没必要的空白行。 eslint: [`padded-blocks`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fpadded-blocks.html)

```js
// bad
function bar() {

  console.log(foo);

}

// also bad
if (baz) {

  console.log(qux);
} else {
  console.log(foo);

}

// good
function bar() {
  console.log(foo);
}

// good
if (baz) {
  console.log(qux);
} else {
  console.log(foo);
}
```

- 19.9 圆括号里不要加空格。 eslint: [`space-in-parens`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fspace-in-parens.html)

```js
// bad
function bar( foo ) {
  return foo;
}

// good
function bar(foo) {
  return foo;
}

// bad
if ( foo ) {
  console.log(foo);
}

// good
if (foo) {
  console.log(foo);
}
```

- 19.10 方括号里不要加空格。看示例。 eslint: [`array-bracket-spacing`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Farray-bracket-spacing.html)

```js
// bad
const foo = [ 1, 2, 3 ];
console.log(foo[ 0 ]);

// good， 逗号后面要加空格
const foo = [1, 2, 3];
console.log(foo[0]);
```

- 19.11 花括号 `{}` 里加空格。 eslint: [`object-curly-spacing`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fobject-curly-spacing.html)

```js
// bad
const foo = {clark: 'kent'};

// good
const foo = { clark: 'kent' };


// bad
function foo() {return true;}
if (foo) { bar = 0;}

// good
function foo() { return true; }
if (foo) { bar = 0; }
复制代码
```

- 19.12 避免一行代码超过 100 个字符（包含空格、纯字符串就不要换行了）。

```js
// bad
const foo = jsonData && jsonData.foo && jsonData.foo.bar && jsonData.foo.bar.baz && jsonData.foo.bar.baz.quux && jsonData.foo.bar.baz.quux.xyzzy;

// bad
$.ajax({ method: 'POST', url: 'https://airbnb.com/', data: { name: 'John' } }).done(() => console.log('Congratulations!')).fail(() => console.log('You have failed this city.'));

// good
const foo = jsonData
  && jsonData.foo
  && jsonData.foo.bar
  && jsonData.foo.bar.baz
  && jsonData.foo.bar.baz.quux
  && jsonData.foo.bar.baz.quux.xyzzy;

// good
$.ajax({
  method: 'POST',
  url: 'https://airbnb.com/',
  data: { name: 'John' },
})
  .done(() => console.log('Congratulations!'))
  .fail(() => console.log('You have failed this city.'));
```

- 19.13 `,` 前避免空格， `,` 后需要空格。 eslint: [`comma-spacing`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fcomma-spacing)

```js
// bad
var foo = 1,bar = 2;
var arr = [1 , 2];

// good
var foo = 1, bar = 2;
var arr = [1, 2];
```

- 19.14 在对象的属性中， 键值之间要有空格。 eslint: [`key-spacing`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fkey-spacing)

```js
// bad
var obj = { "foo" : 42 };
var obj2 = { "foo":42 };

// good
var obj = { "foo": 42 };
```

- 19.15 行末不要空格。 eslint: [`no-trailing-spaces`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-trailing-spaces)
- 19.16 避免出现多个空行。 在文件末尾只允许空一行。 eslint: [`no-multiple-empty-lines`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-multiple-empty-lines)

```js
// bad
var x = 1;

var y = 2;

// good
var x = 1;

var y = 2;
```

## 逗号

- 20.1 不要前置逗号。 eslint: [`comma-style`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fcomma-style.html)

```js
// bad
const story = [
    once
  , upon
  , aTime
];

// good
const story = [
  once,
  upon,
  aTime,
];

// bad
const hero = {
    firstName: 'Ada'
  , lastName: 'Lovelace'
  , birthYear: 1815
  , superPower: 'computers'
};

// good
const hero = {
  firstName: 'Ada',
  lastName: 'Lovelace',
  birthYear: 1815,
  superPower: 'computers',
};
```

- 20.2 结尾额外加逗号，看团队习惯吧 eslint: [`comma-dangle`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fcomma-dangle.html)

```js
// bad - 没有结尾逗号的 git diff
const hero = {
     firstName: 'Florence',
-    lastName: 'Nightingale'
+    lastName: 'Nightingale',
+    inventorOf: ['coxcomb chart', 'modern nursing']
};

// good - 有结尾逗号的 git diff
const hero = {
     firstName: 'Florence',
     lastName: 'Nightingale',
+    inventorOf: ['coxcomb chart', 'modern nursing'],
};

// bad
const hero = {
  firstName: 'Dana',
  lastName: 'Scully'
};

const heroes = [
  'Batman',
  'Superman'
];

// good
const hero = {
  firstName: 'Dana',
  lastName: 'Scully',
};

const heroes = [
  'Batman',
  'Superman',
];

// bad
function createHero(
  firstName,
  lastName,
  inventorOf
) {
  // does nothing
}

// good
function createHero(
  firstName,
  lastName,
  inventorOf,
) {
  // does nothing
}

// good (note that a comma must not appear after a "rest" element)
function createHero(
  firstName,
  lastName,
  inventorOf,
  ...heroArgs
) {
  // does nothing
}

// bad
createHero(
  firstName,
  lastName,
  inventorOf
);

// good
createHero(
  firstName,
  lastName,
  inventorOf,
);

// good (note that a comma must not appear after a "rest" element)
createHero(
  firstName,
  lastName,
  inventorOf,
  ...heroArgs
)
```

## 分号

- 21.1 当 JavaScript 遇到没有分号的换行符时，它会使用[`Automatic Semicolon Insertion`](https://link.juejin.im/?target=https%3A%2F%2Ftc39.github.io%2Fecma262%2F%23sec-automatic-semicolon-insertion)这一规则来决定行末是否加分号。但是，ASI 包含一些古怪的行为，如果 JavaScript 弄错了你的换行符，你的代码就会破坏。所以明确地使用分号，会减少这种不确定性。

```js
// bad
(function () {
  const name = 'Skywalker'
  return name
})()

// good
(function () {
  const name = 'Skywalker';
  return name;
}());

// good
;(() => {
  const name = 'Skywalker';
  return name;
}());
```

[更多](https://link.juejin.im/?target=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F7365172%2Fsemicolon-before-self-invoking-function%2F7365214%237365214).

## 类型

- 22.1 在声明开头执行强制类型转换。
- 22.2 String eslint: [`no-new-wrappers`](https://link.juejin.im/?target=https%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-new-wrappers)

```js
// => this.reviewScore = 9;

// bad
const totalScore = new String(this.reviewScore); // typeof totalScore is "object" not "string"

// bad
const totalScore = this.reviewScore + ''; // invokes this.reviewScore.valueOf()

// bad
const totalScore = this.reviewScore.toString(); // 不保证返回string

// good
const totalScore = String(this.reviewScore);
```

- 22.3 Number eslint: [`radix`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fradix)

```js
const inputValue = '4';

// bad
const val = new Number(inputValue);

// bad
const val = +inputValue;

// bad
const val = inputValue >> 0;

// bad
const val = parseInt(inputValue);

// good
const val = Number(inputValue);

// good
const val = parseInt(inputValue, 10);
```

- 22.4 请在注释中解释为什么要用移位运算，无论你在做什么，比如由于 `parseInt` 是你的性能瓶颈导致你一定要用移位运算。 请说明这个是因为[性能原因](https://link.juejin.im/?target=https%3A%2F%2Fjsperf.com%2Fcoercion-vs-casting%2F3),

```js
// good
/**
 * parseInt 导致代码运行慢
 * Bitshifting the String 将其强制转换为数字使其快得多。
 */
const val = inputValue >> 0;
```

- 22.5 **注意:** 使用 bitshift 操作时要小心。数字表示为 64 位值，但 bitshift 操作始终返回 32 位整数。对于大于32位的整数值，Bitshift可能会导致意外行为。

```js
2147483647 >> 0 //=> 2147483647
2147483648 >> 0 //=> -2147483648
2147483649 >> 0 //=> -2147483647
```

- 22.6 Booleans

```js
const age = 0;

// bad
const hasAge = new Boolean(age);

// good
const hasAge = Boolean(age);

// best
const hasAge = !!age;
```

## 命名约定

- 23.1 避免用一个字母命名，让你的命名更加语义化。 eslint: [`id-length`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fid-length)

```js
// bad
function q() {
  // ...
}

// good
function query() {
  // ...
}
```

- 23.2 用 camelCase 命名你的对象、函数、实例。 eslint: [`camelcase`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fcamelcase.html)

```js
// bad
const OBJEcttsssss = {};
const this_is_my_object = {};
function c() {}

// good
const thisIsMyObject = {};
function thisIsMyFunction() {}
```

- 23.3 用 PascalCase 命名类。 eslint: [`new-cap`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fnew-cap.html)

```js
// bad
function user(options) {
  this.name = options.name;
}

const bad = new user({
  name: 'nope',
});

// good
class User {
  constructor(options) {
    this.name = options.name;
  }
}

const good = new User({
  name: 'yup',
});
```

- 23.4 不要用前置或后置下划线。 eslint: [`no-underscore-dangle`](https://link.juejin.im/?target=http%3A%2F%2Feslint.org%2Fdocs%2Frules%2Fno-underscore-dangle.html)

> JavaScript 没有私有属性或方法的概念。尽管前置下划线通常的概念上意味着 “private”，但其实，这些属性是完全公开的，因此这部分也是你的 API 的内容。这一概念可能会导致开发者误以为更改这个不会导致崩溃或者不需要测试。

```js
// bad
this.__firstName__ = 'Panda';
this.firstName_ = 'Panda';
this._firstName = 'Panda';

// good
this.firstName = 'Panda';
复制代码
```

- 23.5 不要保存 `this` 的引用，使用箭头函数或硬绑定。

```
// bad
function foo() {
  const self = this;
  return function () {
    console.log(self);
  };
}

// bad
function foo() {
  const that = this;
  return function () {
    console.log(that);
  };
}

// good
function foo() {
  return () => {
    console.log(this);
  };
}
```

- 23.6 文件名应与默认导出（`export default`）的名称完全匹配

```js
// file 1 contents
class CheckBox {
  // ...
}
export default CheckBox;

// file 2 contents
export default function fortyTwo() { return 42; }

// file 3 contents
export default function insideDirectory() {}

// in some other file
// bad
import CheckBox from './checkBox'; // PascalCase import/export, camelCase filename
import FortyTwo from './FortyTwo'; // PascalCase import/filename, camelCase export
import InsideDirectory from './InsideDirectory'; // PascalCase import/filename, camelCase export

// bad
import CheckBox from './check_box'; // PascalCase import/export, snake_case filename
import forty_two from './forty_two'; // snake_case import/filename, camelCase export
import inside_directory from './inside_directory'; // snake_case import, camelCase export
import index from './inside_directory/index'; // requiring the index file explicitly
import insideDirectory from './insideDirectory/index'; // requiring the index file explicitly

// good
import CheckBox from './CheckBox'; // PascalCase export/import/filename
import fortyTwo from './fortyTwo'; // camelCase export/import/filename
import insideDirectory from './insideDirectory'; // camelCase export/import/directory name/implicit "index"
// ^ supports both insideDirectory.js and insideDirectory/index.js
```

- 23.7 默认导出（`export default`）一个函数时，函数名、文件名统一。

```js
function makeStyleGuide() {
  // ...
}

export default makeStyleGuide;
```

- 23.8 当你 export 一个构造函数/类/单例/函数库对象时用 PascalCase。

```js
const AirbnbStyleGuide = {
  es6: {
  }
};

export default AirbnbStyleGuide;
```

- 23.9 简称和首字母缩写应该全部大写或全部小写。

> 名字是给人看的，不是给电脑看的。

```js
// bad
import SmsContainer from './containers/SmsContainer';

// bad
const HttpRequests = [
  // ...
];

// good
import SMSContainer from './containers/SMSContainer';

// good
const HTTPRequests = [
  // ...
];

// best
import TextMessageContainer from './containers/TextMessageContainer';

// best
const Requests = [
  // ...
];
```

- 23.10 全大写字母定义用来导出的常量

```js
// bad
const PRIVATE_VARIABLE = 'should not be unnecessarily uppercased within a file';

// bad
export const THING_TO_BE_CHANGED = 'should obviously not be uppercased';

// bad
export let REASSIGNABLE_VARIABLE = 'do not use let with uppercase variables';

// ---

// allowed but does not supply semantic value
export const apiKey = 'SOMEKEY';

// better in most cases
export const API_KEY = 'SOMEKEY';

// ---

// bad - unnecessarily uppercases key while adding no semantic value
export const MAPPING = {
  KEY: 'value'
};

// good
export const MAPPING = {
  key: 'value'
};
```

## 访问器

- 24.1 不需要使用属性的访问器函数。
- 24.2 不要使用 JavaScript 的 getters/setters，因为他们会产生副作用，并且难以测试、维护和理解。如果必要，你可以用 getVal()和 setVal() 去构建。

```js
// bad
class Dragon {
  get age() {
    // ...
  }

  set age(value) {
    // ...
  }
}

// good
class Dragon {
  getAge() {
    // ...
  }

  setAge(value) {
    // ...
  }
}
```

- 24.3 如果属性/方法是一个 `boolean`， 请用 `isVal()` 或 `hasVal()`。

```js
// bad
if (!dragon.age()) {
  return false;
}

// good
if (!dragon.hasAge()) {
  return false;
}
```

- 24.4 可以用 get() 和 set() 函数，但是要保持一致。

```js
class Jedi {
  constructor(options = {}) {
    const lightsaber = options.lightsaber || 'blue';
    this.set('lightsaber', lightsaber);
  }

  set(key, val) {
    this[key] = val;
  }

  get(key) {
    return this[key];
  }
}
```

## Events

- 25.1 给事件或其他传递数据时，不直接使用原始值，而是通过对象包装。这样在未来需要增加或减少参数，不必找到每个使用中的处理器。

```
// bad
$(this).trigger('listingUpdated', listing.id);

...

$(this).on('listingUpdated', (e, listingId) => {
  // do something with listingId
});
```

prefer:

```
// good
$(this).trigger('listingUpdated', { listingId: listing.id });

...

$(this).on('listingUpdated', (e, data) => {
  // do something with data.listingId
});
```

## 小结

所谓规范，更多的还是为了代码的可读性，毕竟我们的代码更重要的是给人看。同时，合理的规范，也会帮助我们规避很多不必要的 bug。

## 交流群

> 关注微信公众号：前端发动机，回复：加群。

## 后记

如果你看到了这里，且本文对你有一点帮助的话，希望你可以动动小手支持一下作者，感谢🍻。文中如有不对之处，也欢迎大家指出，共勉。好了，又耽误大家的时间了，感谢阅读，下次再见！

- **文章仓库** [🍹🍰fe-code](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwuyawei%2Ffe-code)
- **社交聊天系统（vue + node + mongodb）- 💘🍦🙈Vchat**

感兴趣的同学可以关注下我的公众号 **前端发动机**，好玩又有料。
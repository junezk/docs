# Typescript 学习笔记

## 基本类型

- 布尔类型（boolean）
- 数字类型（number）
- 字符串类型（string）
- 数组类型（array）
- 元组类型（tuple）
- 枚举类型（enum）
- 任意值类型（any）
- null 和 undefined
- void 类型
- never 类型

其中元组、枚举、任意值、void类型和 never类型是TypeScript有别与JavaScript的特有类型

### 布尔型

```typescript
let flag: boolean = true;
flag = 1; // 报错
```

### 数字类型
在TypeScript中，数字都是浮点型。TypeScript同时支持二进制、八进制、十进制和十六进制字面量，示例代码如下

```typescript
let binaryLiteral: number = 0b1010; // 二进制
let octalLiteral: number = 0o744;    // 八进制
let decLiteral: number = 6;    // 十进制
let hexLiteral: number = 0xf00d;    // 十六进制
```

### 字符串类型

TypeScript支持使用单引号（'）或双引号（"）,还支持使用模板字符串反引号（`）来定义多行文本和内嵌表达式。使用${ expr }的形式嵌入变量或表达式，在处理拼接字符串的时候很有用，示例如下：

```typescript
let name: string = "Angular";
let years: number = 5;
let words: string = `您好，今年是 ${ name } 发布 ${ years + 1} 周年`;
```

### 数组类型

TypeScript有两种数组定义方式，示例代码如下：

```typescript
// 在元素类型后面加上[]
let arr: number[] = [1, 2];

// 或者使用数组泛型
let arr: Array<number> = [1, 2];
```

### 元组类型

元祖类型用来表示已知元素数量和类型的数组，个元素的类型不必相同。下面定义了一组值分别为字符串和数字类型的元祖，示例代码如下：

```typescript
let x: [string, number];
x = ['Angular', 25];    // 运行正常
x = [25, 'Angular'];    // 报错
console.log(x[0]);    // 输出 Angular
```

### 枚举类型

枚举是一个可被命名的整型常数的集合，枚举类型为集合成员赋予有意义的名称，增强可读性，示例代码如下：

```typescript
enum gender{
    male,
    female
}
var sex:gender;

//sex='123'//报错
sex=gender.male;//对
```

### 任意值类型

任意值是 TypeScript 针对编程时类型不明确的变量使用的一种数据类型，它常用于一下三种情况。

变量的值会动态改变时，比如来自用户的输入，任意值类型可以让这些变量跳过编译阶段的类型检查，示例代码如下：
```typescript
let x: any = 1;    // 数字类型
x = 'I am who I am';    // 字符串类型
x = false;    // 布尔类型
```

改写现有代码时，任意值允许在编译时可选择地包含或移除类型检查，示例代码如下：

```typescript
let x: any = 4;
x.ifItExists();    // 正确，ifItExists方法在运行时可能存在，但这里并不会检查
x.toFixed();    // 正确
```
定义存储各种类型数据的数组时，示例代码如下：

```typescript
let arrayList: any[] = [1, false, 'fine'];
arrayList[1] = 100;
```

### null 和 undefined

默认情况下，null 和 undefined 是其它类型的子类型，可以赋值给其它类型，如数字类型，此时，赋值后的类型会变成 null 或 undefined。而在TypeScript中启用严格的空校验（--strictNullChecks）特性，就可以使得null 和 undefined 只能被赋值给 void 或本身对应的类型，它们都是只有一种值.示例代码如下：

```typescript
// 启用 --strictNullChecks
let x: number;
x = 1; // 运行正确
x = undefined;    // 运行错误
x = null;    // 运行错误
```

上面的例子中变量 x 只能是数字类型。如果一个类型可能出行 null 或 undefined， 可以用 | 来支持多种类型，示例代码如下：

```typescript
// 启用 --strictNullChecks
let x: number | null | undefined;
x = 1; // 运行正确
x = undefined;    // 运行正确
x = null;    // 运行正确
```

### void 类型

在 TypeScript 中，使用 void 表示没有任何类型。 例如一个函数没有返回值时，意味着返回值类型是 void，示例代码如下：

```typescript
function hello(): void {
    alert("Hello Angular");
}
```

对于可忽略返回值的回调函数来说，使用 void 类型会比任意值类型更安全一些，示例代码如下：

```typescript
function func(foo:() => void) {
    let f = foo();    // 使用函数 foo 的返回值
    f.doSth();    // 报错， void 类型不存在 doSth() 方法， 此时换成任意值类型都不回报错
}
```

### never 类型

never 是其它类型（包括 null 和 undefined）的子类型，代表从不会出现的值。这意味着声明为 never 类型的变量只能被 never 类型所赋值，在函数中它通常表现为抛出异常或无法执行到终止点（例如无线循环），示例代码如下：

```typescript
let x: never;
let y: number;

// 运行错误，数字类型不能转为 never 类型
x = 123;

// 运行正确，never 类型可以赋值给 never类型
x = (()=>{ throw new Error('exception')})();

// 运行正确，never 类型可以赋值给 数字类型
y = (()=>{ throw new Error('exception')})();

// 返回值为 never 的函数可以是抛出异常的情况
function error(message: string): never {
    throw new Error(message);
}

// 返回值为 never 的函数可以是无法被执行到的终止点的情况
function loop(): never {
    while (true) {}
}
```

### 类型推测

```
var a=12 等于 var a:number
var a; 等于 var a:any
```

### 联合类型

联合类型表示取可以为多种类型中的一种，可以通过管道(|)将变量设置为多种类型，赋值时可以根据设置的类型来赋值。

```
let numType:string|number = "hello word";
numType = 20;  // 不会报错
```

### 函数类型

1.参数: 有类型 签名检查--个数、类型 得对 外部声明变量: declare
2.返回值

### 复合类型(Object Type)

```
var a:{x:number,y;number,z?}
a={x:12,y:5}
```

### 接口(interface)

和java的接口不一样。TS中的接口可以理解为一种类型，一个规范，一个约束，可以对数组、对象、类进行约束，接口中可以定义选属性、只读属性、任意属性，这样在开发代码时便于调试。

面向对象:
class写法、extend、多继承...

访问修饰符:
  1.public 公有 任何人可以访问
  2.private 私有 只有类内部可以访问
  3.protected 受保护(友元) 只有子类能用
  4.readonly 只读 创建时一次赋值，以后不能修改
  5.? 可选属性  有没有都行
  6.接口可以动态添加属性

```typescript
interface Istate {  // 接口关键字为 interface  接口名字必须为大写的I字开头  state为接口名
  name:string;
  age:number;
}

let user:Istate = {
  name: '张三';
  age: 10;
}
```

### 泛型: 别跟any搞混了

any:类型随便

泛型:类型随便 假如是类型number 里面必须所有都是number

Array:内部其实是泛型

## Typescript 中的接口

TypeScript 的核心原则之一是对值所具有的结构进行类型检查。我们使用接口（Interfaces）来定义对象的类型。接口是对象的状态(属性)和行为(方法)的抽象(描述)

```typescript
/* 
需求: 创建人的对象, 需要对人的属性进行一定的约束
  id是number类型, 必须有, 只读的
  name是string类型, 必须有
  age是number类型, 必须有
  sex是string类型, 可以没有
*/

// 定义人的接口
interface IPerson {
  readonly id: number
  name: string
  age: number
  sex?: string
}

const person1: IPerson = {
  id: 1,
  name: 'tom',
  age: 20,
  sex: '男'
}
person1.id = 2 // error

```

### 函数类型的接口

接口也可以描述函数类型。

```typescript
/* 
接口可以描述函数类型(参数的类型与返回的类型)
*/

interface SearchFunc {
  (source: string, subString: string): boolean
}

const mySearch: SearchFunc = function (source: string, sub: string): boolean {
  return source.search(sub) > -1
}
```

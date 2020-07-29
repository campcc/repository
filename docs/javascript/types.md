---
title: 内置类型
order: 1
nav:
  title: JavaScript
  order: 4
---

# 内置类型

## 值与类型

JavaScript 提供了 7 种内置类型，七种内置类型又分为两大类型：**基本类型 和 对象**。

基本类型有 6 种：`null`，`undefined`，`string`，`boolean`，`number`，`symbol`。

引用类型特指对象：`object`。

基本类型与引用类型在存储上有一定区别，基本类型的变量，其标识符和值都存储在 **栈** 中，因为它们占据空间小、大小固定，属于被频繁使用数据，基本类型时**按值访问的**。

引用类型的值存储在 **堆** 中，由于 JavaScript 不允许直接访问内存中的位置，为了访问引用类型的值，会在 **栈** 中存放值的指针（对应堆中的起始地址），所以引用类型其实是**按地址访问的**，当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

**JavaScript 中的值有类型，但变量无类型**。所以内置类型是相对于 JavaScript 中的值而言的，而变量只是这些值的容器。

## 类型判断

JavaScript 提供了一个 `typeof` 运算符，可以用来查看值的类型，但 `typeof` 在类型判断时并不准确。

### typeof

用一句话来总结 `typeof` ：**对于基本类型，除了 null 都可以显示正确的类型，对于对象，除了函数都会显示 object**。

```js
typeof undefined; // "undefined"
typeof 'str'; // "string"
typeof true; // "boolean"
typeof 857; // "number"
typeof Symbol(); // "symbol"

typeof null; // "object"

typeof []; // "object"
typeof {}; // "object"
typeof console.log; // "function"
```

关于 `typeof null`，会返回 `object` 的原因？

这是 JavaScript 存在了一个很久的 bug。因为在 JS 的最初版本中，使用的是 32 位系统，为了性能考虑使用低位存储了变量的类型信息，000 开头代表是对象，然而 `null` 表示为全零，所以将它错误的判断为 `object`。虽然现在的内部类型判断代码已经改变了，但是对于这个 Bug 却是一直流传下来。

你可能会疑问既然是 bug，为什么不修复？

因为 Web 上的太多代码都依赖于这个 bug，因此修复它可能会导致大量的新 bug，所以这似乎是一个永远不会被修复的 bug

### instanceof

** 想判断一个对象的类型，可以考虑使用 instanceof。**

`instnaceof` 的机制是，通过判断对象的原型链中是否能找到类型的 `prototype`：

```js
var obj = {};
obj instanceof Object; // true
```

但是由于是基于原型链的查找，所以对于所有的引用类型，它们都是对象的实例，

```js
[] instanceof Array // true
[] instanceof Object // true，
```

我们也可以试着简单实现一下 `instanceof`，

```js
function _instanceof(left, right) {
  let prototype = right.prototype; // 获得类型的原型
  left = left.__proto__; // 获得对象的原型
  while (true) {
    // 判断对象的类型是否等于类型的原型
    if (left === null) return false;
    if (prototype === left) return true;
    left = left.__proto__;
  }
}
```

### constructor

我们知道当创建一个函数 **F** 时，JavaScript 引擎会为 **F** 添加 `prototype` 属性，然后在 `prototype` 属性上添加一个 `constructor` 属性，让它指向 **F** 的引用：

```js
function F() {}
F.prototype.constructor === F; // true
```

这是引擎默认的行为，目的是表明**对象是由哪个函数构造的**，在 JavaScript 中，`function` 其实就是一个语法糖，所有的函数本质上都是一个 `Function` 对象。利用这一特性，我们可以通过 `constructor` 来判断对象的数据类型，因为 JavaScript 为我们提供了很多的内置对象：

```js
[].constructor === Array // true
''.constructor === String // true
false.constructor === Boolean // true
... ...
```

但是 `constructor` 是不可靠的，因为对象的原型是可以被修改的，比如：

```js
function F() {}
F.prototype = new Array();
var f = new F();
f.constructor === F; // false
f.constructor === Array; // true
```

### Object.prototype.toString

JavaScript 类型转换中有一种相当重要的种类叫：**装箱转换**。

每一种基本类型 `Number、String、Boolean、Symbol` 在对象中都有对应的类，所谓装箱转换，正是把基本类型转换为对应的对象。

使用内置的 `Object` 函数，我们可以在 JavaScript 代码中显式调用装箱能力，而在 JavaScript 中，没有任何方法可以更改私有的 `Class` 属性，因此 `Object.prototype.toString` 是可以准确识别对象对应的基本类型的方法。

每一类装箱对象皆有私有的 `Class` 属性，这些属性可以用 `Object.prototype.toString` 获取：

```js
Object.prototype.toString.call(''); // [object String]
Object.prototype.toString.call(null); // [object Null]
Object.prototype.toString.call(undefined); // [object Undefined]
Object.prototype.toString.call(NaN); // [object Number]
```

需要注意的是，`call` 本身会产生装箱操作，所以需要配合 `typeof` 来区分基本类型还是对象。

## 类型 3 问

### 1. 为什么有的编程规范要求用 void 0 代替 undefined ？

`undefined` 类型表示未定义，它的类型只有一个值，就是 `undefined`。

任何变量在赋值前是 `undefined` 类型、值为 `undefined`，一般我们可以用全局变量 `undefined`（就是名为 `undefined` 的这个变量）来表达这个值，或者 `void` 运算来把任意一个表达式变成 `undefined` 值。

因为 JavaScript 的代码 `undefined` 是一个变量，而并非是一个关键字，这是 JavaScript 语言公认的设计失误之一，所以，我们为了避免无意中被篡改，建议使用 `void 0` 来获取 `undefined` 值

### 2. 为什么 0.1 + 0.2 不等于 0.3 ？

JavaScript 采用 IEEE 754-2008 双精度版本（64 位），其实只要采用 IEEE 754 的语言都有该问题。

首先，计算机表示十进制是采用二进制表示的，所以 0.1，0.2 在二进制分别表示为,

```js
// (0011) 表示循环
0.1 = 2 ^ (-4 * 1.10011(0011));
0.2 = 2 ^ (-3 * 1.10011(0011));
```

IEEE 754 六十四位中符号位占一位，整数位占十一位，其余五十二位都为小数位。

因为 0.1 和 0.2 都是无限循环的二进制，所以相加时在小数位末尾处需要判断是否进位，由于 IEEE 754 中小数位有 52 位，所以 0.1，0.2 进位后转化为，

```js
0.1 = 2^-4 * 1.10011(0011 * 12次)010
0.2 = 2^-3 * 1.10011(0011 * 12次)010
```

相加后结果为，

```js
0.1 + 0.2 = 2^-2 * 1.0011(0011 * 11次)0100
```

这个结果换算成十进制就是：`0.30000000000000004`，最终导致 0.1 + 0.2 != 0.3

解决方案，使用原生 parseFloat，

```js
parseFloat((0.1 + 0.2).toFixed(10));
```

此外，根据浮点数的定义，非整数的 Number 类型无法用 ==（=== 也不行） 来比较，

正确的浮点数比较方法为：**检查等式左右两边差的绝对值是否小于最小精度**：

```js
Math.abs(0.1 + 0.2 - 0.3) <= Number.EPSILON; // 检查等式左右两边差的绝对值是否小于最小精度
```

### 3. 为什么给对象添加的方法能用在基本类型上 ？

JavaScript 语言设计上试图模糊对象和基本类型之间的关系，我们日常代码可以把对象的方法在基本类型上使用，比如：

```js
console.log('abc'.charAt(0)); //a
```

甚至我们在原型上添加方法，都可以应用于基本类型，比如以下代码，在 Symbol 原型上添加了 hello 方法，在任何 Symbol 类型变量都可以调用。

```js
Symbol.prototype.hello = () => console.log('hello');
var a = Symbol('a');
console.log(typeof a); //symbol，a并非对象
a.hello(); //hello，有效
```

所以答案就是，运算符提供了装箱操作，它会根据基础类型构造一个临时对象，使得我们能在基础类型上调用对应对象的方法。

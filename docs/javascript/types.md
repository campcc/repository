---
title: 内置类型
order: 1
nav:
  title: JavaScript
  order: 4
---

# 内置类型

这是《JavaScript》系列的第一篇文章，今天我们来学习 JavaScript 的内置类型。

![image.png](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fb6dee11d724c189dbeb50d86c15615~tplv-k3u1fbpfcp-zoom-1.image)

准备好了吗？滴滴滴 ~

## 值与类型

最新的 ECMAScript 标准（ES2019 / ES10）定义了 **8** 种数据类型。

按存储方式的不同，这 8 种类型又分为两大类型：**原始类型 和 引用类型**。

原始类型有 7 种：

- String
- Number
- Boolean
- Null
- Undefined
- Symbol（ES2015 新增）
- BigInt（ES2019 新增）

引用类型特指对象：

- Object

下面我们从两者的一些概念和区别深入地理解这两大类型。

### 原始值（primitive values）与 对象（object）

所有的原始类型都是不可变的（值本身无法被改变），例如 JavaScript 中，我们对字符串的操作一定返回了一个新字符串，但原始字符串并没有被改变，这与其他语言中字符串的操作可能不同。所有这些不可变类型的值叫原始值（primitive values）。

什么是对象呢？

是梦见苍颜、荒芜、衰老、黑暗... ...都不及在梦里松开我的手，更让我想要醒来的那个人吗？

![image.png](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b960b112b1d74d47803fa59bad86f77b~tplv-k3u1fbpfcp-zoom-1.image)

咳咳... ...显然不是 😝 ，在计算机科学中, 对象是指 **内存中的可以被标识符引用的一块区域**。

而在 JavaScript 中，对象也可以被看作是 **一组属性和方法的集合**。比如我们用对象字面量语法来定义一个对象时，对象本身就具有了 `constructor`, `hasOwnProperty` 等属性和方法，

```js
var obj = {};

obj.constructor; // ƒ Object() { [native code] }

obj.hasOwnProperty; // ƒ hasOwnProperty() { [native code] }
```

而且这些属性可以被增删减，属性使用键来标识，键值可以是一个字符串或者符号值（Symbol），属性的值可以是任意类型（包括具有复杂数据结构的对象本身）。所以对象也可以看做 **一组键值对的集合**，这里的值是可变的，对于这种特殊的可变类型我们称为引用类型。果然，对象都是善变的 (>\_<) ，呸 ~

### 存储上的区别

JavaScript 中对于可变和不可变类型的存储，有一定的区别。

原始类型的变量，其标识符和值都存储在 **栈** 中，为什么？

因为它们 **占据空间小、大小固定，属于被频繁使用数据**，所以存储在栈中 **按值访问**。

引用类型的值大小不固定，被存储在 **堆** 中，但是由于 JavaScript 不允许直接访问内存中的位置，为了访问引用类型的值，还会在 **栈** 中存放值的指针（**对应堆中的起始地址**）。

所以引用类型其实是 **按地址访问的**，当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

![image.png](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2914e458253485cb1545ccf9717626a~tplv-k3u1fbpfcp-zoom-1.image)

## 类型判断

JavaScript 提供了 `typeof` 运算符用来判断变量的类型，但 `typeof` 在类型判断时并不准确。

### typeof

用一句话来总结 `typeof` 就是，

**对于基本类型，除了 null 都可以显示正确的类型，对于对象，除了函数都会显示 object**。

```js
typeof undefined; // "undefined"
typeof ''; // "string"
typeof true; // "boolean"
typeof 857857; // "number"
typeof Symbol(); // "symbol"
typeof 10n; // "bigint"

typeof null; // "object"

typeof []; // "object"
typeof {}; // "object"
typeof console.log; // "function"
```

关于 `typeof null`，会返回 `object` 的原因？

这是 JavaScript 存在了一个很久的 bug。因为在 JS 的最初版本中，使用的是 32 位系统，为了性能考虑使用低位存储了变量的类型信息，000 开头代表是对象，然而 `null` 表示为全零，所以将它错误的判断为 `object`。虽然现在的内部类型判断代码已经改变了，但是对于这个 Bug 却是一直流传下来。

你可能会疑问既然是 bug，为什么不修复呢？

因为 Web 上太多的代码都依赖于这个 bug，因此修复它可能会导致大量的新 bug，所以这可能是一个永远不会被修复的 bug 🤣 ，无奈...

### instanceof

想判断一个对象的类型，可以考虑使用 instanceof。

`instnaceof` 的内部机制是，**通过判断对象的原型链中能否找到类型的 prototype**：

```js
var obj = {};
obj instanceof Object; // true
```

但是由于是基于原型链的查找，所以对于所有的引用类型，它们其实都是对象的实例，

```js
[] instanceof Array // true
[] instanceof Object // true，
```

基于规范 [ECMAScript-262 edition 3](https://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262,%203rd%20edition,%20December%201999.pdf) 中的定义，

```html
11.8.6 The instanceof operator The production RelationalExpression:
RelationalExpression instanceof ShiftExpression is evaluated as follows: 1.
Evaluate RelationalExpression. 2. Call GetValue(Result(1)). 3. Evaluate
ShiftExpression. 4. Call GetValue(Result(3)). 5. If Result(4) is not an object,
throw a TypeError exception.
```

我们也可以试着简单实现一下 `instanceof`，

```js
function _instanceof(left, right) {
  let prototype = right.prototype; // 获得类型的 prototype
  left = left.__proto__; // 获得对象的原型
  while (true) {
    // 遍历对象的原型链，看能否够找到类型的原型
    if (left === null) return false; // 对象的原型链的终点为 null，如果遍历完还是没能找到类型的 prototype，就返回 false
    if (prototype === left) return true; // 注意这里是严格相等，这就是导致为什么 Number instanceof Number 返回 false 的原因
    left = left.__proto__;
  }
}
```

### constructor

什么！？构造函数也可以用来做类型判断吗？没错 😋 。

我们知道当创建一个函数 **F** 时，JavaScript 引擎会为 **F** 添加 `prototype` 属性，然后在 `prototype` 属性上添加一个 `constructor` 属性，让它指向 **F** 的引用：

```js
function F() {}
F.prototype.constructor === F; // true
```

这是引擎默认的行为，目的是表明 **对象是由哪个函数构造的**，在 JavaScript 中，`function` 其实就是一个语法糖，所有的函数本质上都是 `Function` 的实例。利用这一特性，我们可以通过 `constructor` 来判断对象的数据类型，因为 JavaScript 为我们提供了很多的内置对象：

```js
[].constructor === Array // true
''.constructor === String // true
false.constructor === Boolean // true
... ...
```

但 `constructor` 是不可靠的，因为对象的原型是可以被修改的，比如：

```js
function F() {}
F.prototype = new Array();
var f = new F();
f.constructor === F; // false
f.constructor === Array; // true
```

### Object.prototype.toString

JavaScript 类型转换中有一种相当重要的种类叫：**装箱转换**。

所谓装箱转换，就是把 **基本类型转换为对应的对象**

我们知道很多原始类型 `Number、String、Boolean、Symbol` 在对象中都有对应的类。使用内置的 `Object` 函数搭配 `call` 方法，可以在 JavaScript 代码中显式调用装箱能力。而在 JavaScript 中，没有任何方法可以更改私有的 `Class` 属性，所以 `Object.prototype.toString` 是可以准确识别对象对应的原始类型的方法。

每一类装箱对象皆有私有的 `Class` 属性，这些属性可以用 `Object.prototype.toString` 获取：

```js
Object.prototype.toString.call(''); // [object String]
Object.prototype.toString.call(null); // [object Null]
Object.prototype.toString.call(undefined); // [object Undefined]
Object.prototype.toString.call(NaN); // [object Number]
... ...
```

需要注意的是，`call` 本身会产生装箱操作，所以需要配合 `typeof` 来区分基本类型还是对象。

## 类型 3 问

PS: 以下内容部分摘自 Winter 的《重学前端》

### 1. 为什么有的编程规范要求用 void 0 代替 undefined ？

`undefined` 类型表示未定义，它的类型只有一个值，就是 `undefined`。

任何变量在赋值前是 `undefined` 类型、值为 `undefined`，一般我们可以用全局变量 `undefined`（就是名为 `undefined` 的这个变量）来表达这个值，或者 `void` 运算来把任意一个表达式变成 `undefined` 值。

因为 JavaScript 的代码 `undefined` 是一个变量，而并非是一个关键字，这是 JavaScript 语言公认的设计失误之一，所以，我们为了避免无意中被篡改，建议使用 `void 0` 来获取 `undefined` 值

### 2. 为什么 0.1 + 0.2 不等于 0.3 ？

JS 采用 [IEEE 754-2008](https://standards.ieee.org/standard/754-2008.html) 双精度版本（64 位），只要采用 IEEE 754 的语言都有该问题。

首先，计算机表示十进制是采用二进制表示的，所以 0.1，0.2 在二进制分别表示为,

```js
// (0011) 表示循环
0.1 = 2 ^ (-4 * 1.10011(0011));
0.2 = 2 ^ (-3 * 1.10011(0011));
```

IEEE 754 六十四位中符号位占一位，整数位占十一位，其余五十二位都为小数位。

因为 0.1 和 0.2 都是无限循环的二进制，所以相加时在小数位末尾处需要判断是否进位，

由于 IEEE 754 中小数位有 52 位，所以 0.1，0.2 进位后转化为，

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

所以正确的浮点数比较方法应该为：**检查等式左右两边差的绝对值是否小于最小精度**：

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
console.log(typeof a); // symbol，a并非对象
a.hello(); // hello，有效
```

所以答案就是，运算符也提供了装箱操作，它会根据基础类型构造一个临时对象，使得我们能在基础类型上调用对应对象的方法。

（完）

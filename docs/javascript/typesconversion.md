---
title: 类型转换
order: 2
---

# 类型转换

[上一篇](https://campcc.github.io/repository/javascript/types) 我们学习了 JavaScript 的内置类型，本篇我们来学习类型之间的转换规则。

全文约 2000 字，阅读完大约需要 3 分钟。看完本文，你可以知道：

1. 什么是强制转换，强制转换有哪些规则 ？
2. 强制转换时，有哪些坑点需要特别注意 ？
3. JavaScript 中的隐式类型转换有哪些 ？
4. 使用比较运算符时，不同类型之间是如何进行比较的 ？

![image.png](https://i.loli.net/2020/09/11/tGyhxcsaJ2E8mwH.png)

## 强制转换

强制转换指使用 `String`、`Boolean` 和 `Number` 和 三个函数，手动将各种类型的值，分别转换成数字、字符串或者布尔值。

JavaScript 中有且仅有上述的三种情况会触发强制转换。

根据原始类型和对象两大类型的不同，强制转换可以分为以下 **4** 种情况。

### 原始类型转字符串

调用 String() 函数，可以将原型类型的值转为字符串，转换的规则如下。

- 字符串：转换后还是原来的值
- 布尔值：true 转换为 "true"，false 转换为 "false"
- 数值：转为数值对应的字符串
- undefined：转为字符串 "undefined"
- null：转为字符串 "null"
- Symbol：转为对应的 Symbol 值
- BigInt：转为大整数对应的字符串

```js
String('str'); // "str"
String(true); // "true"
String(false); // "false"
String(857); // "857"
String(undefined); // "undefined"
String(null); // "null"
String(Symbol('symbol')); // "Symbol(symbol)"
String(10n); // "10"
```

### 原始类型转布尔值

调用 Boolean() 函数，可以将原始类型的值转为布尔值，它的规则相对简单。

除了以下 8 个值的转换结果为 `false` 外，其他的值全部为 `true`：

- undefined
- null
- +0
- -0
- 0n
- -0n
- NaN
- ''

```js
Boolean(undefined); // false
Boolean(null); // false
Boolean(0); // false
Boolean(-0); // false
Boolean(0n); // false
Boolean(-0n); // false
Boolean(NaN); // false
Boolean(''); // false
```

### 原始类型转数值

调用 Number() 函数，可以将原始类型的值转为数值，转换的规则如下。

- 字符串：如果可以被解析为数值，则转换为相应的数值，否则转为 NaN
- 布尔值：true 转换为 1，false 转换为 0
- 数值：转换后还是原来的值
- undefined：转为 NaN
- null：转为 0
- Symbol：报错，Symbol 只能强制转换为字符串或者数值
- BigInt：转为大整数对应的数值

```js
Number('857'); // 857
Number(''); // 0
Number('str'); // NaN
Number(true); // 1
Number(false); // 0
Number(857); // 857
Number(undefined); // NaN
Number(null); // 0
Number(Symbol()); // Uncaught TypeError: Cannot convert a Symbol value to a number
Number(10n); // 10
```

### 对象转原始类型

首先明确，按照强制转换的规则，对象仅可转为：字符串，布尔值和数值。所有的对象转布尔值结果都为 `true`。

对象转换为原始类型的基本原理为：**调用自身的 toString, valueOf 方法**。

默认先调用 toString 方法，只有在转数值时，会优先调用 valueOf。在 ES2015 新增 Symbol 后，也可以重写 `Symbol.toPrimitive`，该方法在转原始类型时调用优先级最高。

```js
var obj = {
  toString() {}, // 默认先调用此方法
  valueOf() {}, // 如果转数值，valueOf 会优先调用
  [Symbol.toPrimitive]() {}, // 如果重写了此方法，转原始类型时会忽略 toString 和 valueOf，直接调用此方法
};
```

下面是重写 Symbol.toPrimitive 的一个例子：

```js
var obj = {
  toString() {
    return '';
  },
  valueOf() {
    return 1;
  },
  [Symbol.toPrimitive]() {
    return '0';
  },
};

Number(obj); // 0，先调用 Symbol.toPrimitive，返回字符串 '0'，继续转数值得到结果 0
String(obj); // '0'
Boolean(obj); // true，不管是否重写了内部方法，所有的对象转布尔值都是 true
```

下面是对象转原始类型时，一些需要注意的点：

```js
Number({}); // NaN，参数是普通对象时，返回 NaN
Number([1]); // 1，参数是数组时，首先会调用 valueOf，再调用 toString，最后再进行转换
Number(['1']); // 1
Number([]); // 0
Number([null]); // 0，先调用 valueOf 返回 [null]，再调用 toString，返回 ""，字符串转数值得到 0
Number([undefined]); // 0
String({}); // "[object Object]"，参数是普通对象时，返回 "[object Object]"
String([1, 2, 3]); // "1,2,3"，参数是数组时，返回该数组的字符串形式
String([1, null, undefined, {}]); // "1,,,[object Object]"
```

## 隐式转换

JavaScript 在以下情况中，会进行隐式转换。

### 四则运算

加法运算时，当其中一方是字符串时，另一方会隐式转为字符串；当其中一方是对象时，会尝试先将对象隐式转为字符串。

```js
'5' + 20; // '520'
{
}
+'5'; // 5，这里是否有疑惑 ？其实是运算符优先级的问题导致的，并没有违背我们的规则，尝试给 {} 加上圆括号或者以变量的方式声明对象
({} + '5'); // '[object Object]5'
5 + {}; // '5[object Object]'，先尝试将 {} 转为字符串 "[object Object]"，然后是字符串加法，数值会被隐式转换为字符串
```

四则中的其他运算（`-，*，/`），当其中一方是数值时，另一方会隐式转换为数值，

```js
2 * '2'; // 4
2 - []; // 2
2 / {}; // NaN
```

### 非布尔型的数据求布尔值

JavaScript 遇到预期为布尔值的地方，就会将非布尔值的参数隐式转换为布尔值，系统内部会自动调用 Boolean 函数。

常见的场景有：

- if 语句的条件部分
- 三元运算符、非运算符中的表达式

```js
if ('abc') console.log('abc'); // 'abc'
({} ? true : false); // true
!![]; // true
```

### 非数值型的数据使用一元运算符(`+, -`)

JavaScript 中对非数值型的数据使用一元运算符时，也会触发隐式转换，转换的规则为：**默认转为数值，只有预期为字符串时，才会转为字符串**。

```js
-[1, 2, 3]; // NaN
-{ foo: 'foo' }; // NaN
```

这种隐晦的自动转换常常得到出乎意料的结果，而且不易除错，通常建议在预期为布尔值、数值、字符串的地方，全部使用 `Boolean`、`Number` 和 `String` 函数进行显式转换。

## 比较运算

接下来我们来看 JavaScript 中比较运算中的类型转换。

首先，严格相等（`===`）运算符不会进行类型转换，仅当操作数严格相等时返回 true，类型转换发生在相等（`==`）运算符中。

下面是相等（`==`）运算符的运算规则，

![image.png](https://i.loli.net/2020/09/11/rxgki15hVUJBsGf.png)

其中，`ToPrimitive` 指对象转换为原始类型。

## 写在最后

本文首发于我的 [博客](https://campcc.github.io/repository/javascript/typesconversion)，才疏学浅，难免有错误，文章有误之处还望不吝指正！

如果有疑问或者发现错误，可以在评论区进行提问和勘误，

如果喜欢或者有所启发，欢迎 star，对作者也是一种鼓励。

（完）

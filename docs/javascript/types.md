---
title: 内置类型
order: 1
nav:
  title: JavaScript
  order: 2
---

# 内置类型

JavaScript 分为 7 种内置类型，七种内置类型又分为两大类型：**基本类型 和 对象**。

基本类型有 6 种：`null`，`undefined`，`string`，`boolean`，`number`，`symbol`。

引用类型特指对象：`object`。

**JavaScript 中的值有类型，但变量无类型**。所以内置类型是相对于 JavaScript 中的值而言的，而变量只是这些值的容器。

## 类型判断

JavaScript 提供了一个 `typeof` 运算符，该运算符可以用来查看值的类型，但 `typeof` 在类型判断时并不准确，下面是你需要了解的类型判断方式。

### typeof

用一句话来总结 `typeof` ：**对于基本类型，除了 null 都可以显示正确的类型，对于对象，除了函数都会显示 object**。

```js
// 基本类型除了 null 以外，typeof 都可以返回正确的类型值
typeof undefined; // "undefined"
typeof 'str'; // "string"
typeof true; // "boolean"
typeof 857; // "number"
typeof Symbol(); // "symbol"

// 对于 null，会返回 object，这是 JavaScript 存在了一个很久的 bug。因为在 JS 的最初版本中，使用的是 32 位系统，为了性能考虑使用低位存储了变量的类型信息，000 开头代表是对象，然而 null 表示为全零，所以将它错误的判断为 object 。虽然现在的内部类型判断代码已经改变了，但是对于这个 Bug 却是一直流传下来。你可能会疑问既然是 bug，为什么不修复，因为 Web 上的太多代码都依赖于这个 bug，因此修复它可能会导致大量的新 bug，所以这似乎是一个永远不会被修复的 bug
typeof null; // "object"

// 对于引用类型，除了函数外都会返回 object
typeof []; // "object"
typeof {}; // "object"
typeof console.log; // "function"
```

### instanceof

`instnaceof` 的内部机制是，通过判断对象的原型链中是否能找到类型的 `prototype`，所以 `instanceof` 可以用来判断对象的类型：

```js
var obj = {};
obj instanceof Object; // true
```

但是由于是基于原型链的查找，所以对于所有的引用类型，它们都是对象的实例，

```js
[] instanceof Array // true
[] instanceof Object // true，
```

我们也可以试着实现一下 `instanceof`，

```js
function _instanceof(left, right) {
  let prototype = right.prototype; // 获得类型的原型
  left = left.__proto__; // 获得对象的原型
  // 判断对象的类型是否等于类型的原型
  while (true) {
    if (left === null) return false;
    if (prototype === left) return true;
    left = left.__proto__;
  }
}
```

### constructor

### Object.prototype.toString

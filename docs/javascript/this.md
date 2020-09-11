---
title: this
order: 3
---

# this

**this** 其实并不难，本文我们从找对象的角度，看如何确定 `this` 的指向。

## 找对象

首先， **this 在大多数情况下是一个对象，也有可能是 undefined 或其他值**。

什么情况下，`this` 是 `undefined` ？函数运行在严格模式下，应用默认绑定规则的时候：

```js
var a = 1;

function foo() {
  'use strict';
  console.log(this.a);
}

foo(); // Uncaught TypeError: Cannot read property 'a' of undefined
```

原理其实很简单，因为规范定义了严格模式下，不能将全局对象 `Window` 用于默认绑定。而大多数情况下，我们说的 `this`，其实就是一个对象，所以确定 `this` 的指向，本质上就是要**找到这个对象**。

**所以接下来我们就来看看如何 “找对象” 🤣 。**

## 绑定规则

找对象最重要的是什么？是不是得先通过各种途径（社交，搭讪，相亲...）去认识对象，途径越多，我们找到对象的几率就越大，对吧，这里也是一样，所以我们需要尽可能的了解 this 的绑定规则。

[ECMAScript 5 规范](http://es5.github.io/) 定义的 `this` 的绑定规则，有 **4** 种。

### 默认绑定

教科书会告诉我们，几乎所有的规则都会有一个默认的情况，`this` 绑定也不例外，默认绑定的规则为：

**非严格模式下，this 指向全局对象，严格模式下，this 会绑定到 undefined**。

```js
var a = 1;

function foo() {
  console.log(this.a);
}

function bar() {
  'use strict';
  console.log(this.a);
}

foo(); // 1，非严格模式下，this 指向全局对象 Window，这里相当于 Window.a

bar(); // Uncaught TypeError: Cannot read property 'a' of undefined，严格模式下，this 会绑定到 undefined，尝试从 undefined 读取属性会报错
```

### 隐式绑定

**如果函数在调用位置有上下文对象，this 就会隐式地绑定到这个对象上**，

说起来有点晦涩，直接看例子：

```js
var a = 1;

function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo, // <-- foo 的调用位置
};

obj.foo(); // 2，foo 在调用位置有上下文对象 obj，this 会隐式地绑定到 obj，this.a 相当于 obj.a
```

这个规则可能会让你想起关于 `this` 经常听到的一句话，**this 依赖于调用函数前的对象**。

需要注意的是，隐式绑定在某些情况下可能会导致绑定丢失，具体来说有两种情况，

第一种是使用函数别名调用时：

```js
var a = 1;

function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo,
};

var bar = obj.foo;

bar(); // 1，赋值并不会改变引用本身，使用函数别名调用时，bar 虽然是 obj.foo 的一个引用，但是实际上引用的还是 foo 函数本身，所以这里隐式绑定并没有生效， this 应用的是默认绑定
```

第二种是函数作为参数传递时：

```js
function foo() {
  console.log(this.a);
}

function bar(fn) {
  fn(); // <-- 调用位置
}

var a = 1;

var obj = {
  a: 2,
  foo: foo,
};

bar(obj.foo); // 1, 参数传递也是一种隐式赋值，即使传入的是函数，这里相当于 fn = obj.foo，所以 fn 实际上引用的还是 foo 函数本身，this 应用默认绑定
```

### 显式绑定

我们知道 `call`，`apply`，`bind` 等方法可以改变 `this` 的指向，通过传入参数就可以指定 `this` 的绑定值，够不够显式 ？这种明目张胆的绑定 `this` 的规则就叫显式绑定。

`call` 和 `apply` 的区别只是接受的参数格式不同，`call` 接受一个参数列表，`apply` 接受一个参数数组，但两者的第一个参数都是相同的，都是 **绑定的 this 值**：

```js
function foo() {
  console.log(this.a);
}

var a = 1;

var obj = { a: 2 };

foo.call(obj); // 2，调用时显式地将 foo 的 this 绑定为 obj 对象，所以这里的 this.a 相当于 obj.a

foo.apply(obj); // 2，同理
```

前文我们提到隐式绑定可能会导致绑定丢失，显式绑定也不例外，

思考一下，如何才能解决绑定丢失的问题？

答案其实很简单，只需要**在调用函数的内部使用显式绑定**，强制地将 `this` 绑定到对象：

```js
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo,
};

function bar(fn) {
  fn.call(obj);
}

var a = 1;

bar(obj.foo); // 2，
```

**这其实就是 bind 的实现原理**，与 `call`，`apply` 不同，`bind` 调用后不会执行，而是会**返回一个硬绑定的函数**，所以通过 `bind` 可以解决绑定丢失的问题。`bind` 也是显式绑定，我们来回顾下 `bind` 的用法：

```js
function foo() {
  console.log(this.a);
}

var obj = { a: 2 };

var a = 1;

var bar = foo.bind(obj);

bar(); // 2，bar 是通过 bind 返回后的一个硬绑定函数，其内部应用了显式绑定
```

此外，需要注意的是，将 `null`，`undefined` 作为第一个参数传入 `call`，`apply`，`bind` ，调用时会被忽略，实际应用的是默认绑定规则，即严格模式下，`this` 为 `undefined`，非严格模式下为全局对象。

### new 绑定

先来回顾下 new 的实现原理，

```js
function _new() {
  let obj = new Object(); // 1. 创建一个空对象
  let Con = [].shift.call(arguments); // 2. 获得构造函数
  obj.__proto__ = Con.prototype; // 3. 链接到原型
  let result = Con.apply(obj, arguments); // 4. 绑定 this，执行构造函数
  return typeof result === 'object' ? result : obj; // 5. 返回 new 出来的对象
}
```

所以，在使用 `new` 来调用函数时，会创建一个链接到函数原型的对象，并把它绑定到函数调用的 `this`。

```js
function foo(a) {
  this.a = a;
}

var bar = new foo(2);

bar.a; // 2，new 会返回一个对象，这个对象绑定到构造函数的 this
```

**应用了 new 绑定规则后，不会被任何方式修改 this 指向**。

### 箭头函数中的 this

ES6 中新增了一种函数类型，[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，箭头函数中 `this` 不会应用上述规则，而是**根据最外层的词法作用域来确定 this**，简单来说，箭头函数的 `this` 就是它**外面第一个不是箭头函数的函数的 this**：

```js
function foo() {
  return () => {
    return () => {
      console.log(this.a);
    };
  };
}

foo()(); // undefined，箭头函数调用时，this 取决于最外层的第一个不是箭头函数的函数，这里就是 foo 函数，非严格模式下，默认绑定全局对象 Window，this.a 相当于 Window.a，输出 undefined
```

## 优先级

**new 绑定 > 显式绑定 > 隐式绑定 > 默认绑定**

## 判断模式

根据绑定规则和优先级，我们可以总结出 `this` 判断的通用模式，

1. 函数是否通过 new 调用？
2. 是否通过 call，apply，bind 调用？
3. 函数的调用位置是否在某个上下文对象中？
4. 是否是箭头函数？
5. 函数调用是在严格模式还是非严格模式下？

## 总结

- **this 的绑定规则有四种：默认绑定，隐式绑定，显式绑定，new 绑定**
- **无法应用其他 3 种规则时就是默认绑定，严格模式下 this 为 undefined，非严格模式下为全局对象**
- **函数在调用位置有上下文对象时，this 会隐式绑定到这个对象**
- **可以通过 call，apply，bind 显式地改变 this 的指向**
- **通过 new 调用时，this 会绑定到调用函数，new 绑定是优先级最高的绑定**
- **箭头函数中的 this 继承至它外层第一个不是箭头函数的函数**

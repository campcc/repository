---
title: 原型与原型链
order: 4
---

# 原型与原型链

[上一篇](https://campcc.github.io/repository/javascript/jsoo) 文章中，我们介绍了 JavaScript 语言的历史，对象，面向对象以及 JavaScript 中实现面向对象的两种方式：基于原型和基于类。

本篇我们来深入了解 JavaScript 中的原型与原型链。

## 什么是原型

原型是顺应人类自然思维的产物，现实生活中我们经常使用 “原型” 来描述事物。例如，中文中有个成语叫做 “照猫画虎”，这里的猫看起来就是虎的原型。计算机语言本质上就是对现实世界的一种描述，在 JavaScript 中，我们使用 **对象来抽象事物**，使用 **原型来描述对象**。

所以，我们先明确一点：JavaScript 中，原型就是用来描述对象的。怎么个描述法？

在[《面向对象》](https://campcc.github.io/repository/javascript/jsoo)一节中，我们提到：在不同的编程语言中，设计者利用各种不同的语言特性来抽象描述对象。

其中最为成功的流派是使用 “类” 的方式来描述对象，这诞生了诸如 C++、Java 等流行的编程语言。这个流派叫做基于类的编程语言。还有一种就是基于原型的编程语言，它们利用原型来描述对象。JavaScript 就是其中代表，当然 JavaScript 并非第一个使用原型的语言，在它之前，[self](<https://en.wikipedia.org/wiki/Self_(programming_language)>)、[kevo](<https://en.wikipedia.org/wiki/Kevo_(programming_language)>) 等语言已经开始使用原型来描述对象了。

[Brendan Eich](https://en.wikipedia.org/wiki/Brendan_Eich) 当年选择了原型系统，作为 JavaScript 描述对象的基础。原型系统抽象对象的形式为：**通过 “复制” 的方式来创建新对象**。“复制操作” 有两种实现思路：

- 一种是并不真的去复制一个原型对象，而是使得新对象持有一个原型的引用
- 另一种是切实地复制对象，从此两个对象再无关联

历史上的基于原型语言因此产生了两个流派，显然，JavaScript 选择了前一种方式。

## JavaScript 中的原型

在 “基于类” 的编程语言中，提倡使用一个关注分类和类之间关系开发模型，在这个模型中，总是先有类，再从类去实例化一个对象。类与类之间又可能会形成继承、组合等关系。类又往往与语言的类型系统整合，形成一定编译时的能力。

与 “基于类” 的抽象风格不同，JavaScript 中的原型系统更提倡先去关注一系列对象实例的行为，而后才去关心如何将这些对象，划分到最近的使用方式相似的原型对象，而不是将它们分成类。

如果我们抛开 JavaScript 用于模拟 Java 类的复杂语法设施（如 new、Function Object、函数的 prototype 属性等），JavaScript 的原型系统可以说相当简单，可以用两条概括：

- 如果所有对象都有私有字段 `[[prototype]]` ，就是对象的原型
- 读一个属性，如果对象本身没有，则会继续访问对象的原型，直到原型为空或者找到为止

这其实就对应了 JavaScript 中的两个概念：原型和原型链。所以当我们了解了一门语言的设计历史后，其中的一些语言特性自然就清晰明了了。

这个模型在 ES 的各个历史版本中并没有太大改变，但从 ES6 以来，JavaScript 提供了一系列内置函数，以便更为直接地访问操纵原型：

- Object.create 根据指定的原型创建新对象，原型可以是 null
- Object.getPrototypeOf 获得一个对象的原型
- Object.setPrototypeOf 设置一个对象的原型

利用这些方法，我们可以完全抛开类的思维，利用原型来实现抽象和复用。

以上是基本的模型，接下来我们通过 JavaScript 原型系统中的一些概念，来理解运行时的原型工作原理。

### 构造器

不管是 “基于类” 还是 “基于原型” 的面向对象实现，首先第一步就是要生成对象。

JavaScript 中可以通过构造器来生成对象，构造器本质上就是一个普通的函数，描述了一类实例对象的基本结构。只是当我们使用 `new` 操作符来调用这个函数时，它就可以被称为构造器（构造函数）。

```js
function People(name) {
  this.name = name;
  this.sayName = function() {
    console.log(this.name);
  };
}

var People = function(name) {
  this.name = name;
  this.sayName = function() {
    console.log(this.name);
  };
};
```

上述两种方式都可以定义构造器，为了与普通函数区别，我们通常会有一个约定：**构造器的首字母需要大写**。

此外，构造器还具有以下特点：

- 函数体内的 `this` 关键字，指代了要生成的对象实例
- 使用构造器生成对象时，必须使用 `new` 来调用构造器

下面是通过构造器生成对象的一个例子：

```js
var lilei = new People('lilei');
var hanmeimei = new People('hanmeimei');

console.log(lilei); // { name: "lilei", sayName: ƒ }
lilei.name; // "lilei"
hanmeimei.sayName(); // "hanmeimei"
```

### 原型对象（prototype）

JavaScript 中，**所有函数都有一个属性 `prototype`（注意与私有字段 `[[prototype]]` 的区分），叫原型对象**。

一般来说，函数的 `prototype` 是一个对象，但是也有一个例外。`Function` 的 `prototype` 是一个函数，

```js
typeof Function.prototype; // "function"

consolo.log(Function.prototype); // ƒ () { [native code] }
```

原型对象里有一个属性 `constructor` 指向构造器，表明了原型对象是由哪个函数构造的。

```js
function Engineer(name) {
  this.name = name;
}

Engineer.prototype; // { constructor: ƒ }
Engineer.prototype.constructor === Engineer; // true
```

### 实例对象

JavaScript 提供了两种方式来创建一个实例对象，一种是通过 `new` 调用构造器。

```js
function Engineer(name) {
  this.name = name;
}

var engineer = new Engineer('engineer');

engineer.__proto__ === Engineer; // true
```

有时候我们可能没有构造器，而是希望从现有对象来实例化另一个对象，这时可以使用 `Object.create` 方法，该方法允许传入一个对象或 null 作为原型，创建一个新的实例对象。

```js
var Engineer = {
  name: 'engineer',
};

var engineer = Object.create(Engineer);

engineer.__proto__ === Engineer; // true
```

一般来说，**每个实例对象都有 `__proto__` 属性，指向构造器的原型对象**。但是也有一个例外，Object.create 传入 null 作为参数时，会创建一个原型为 null 的实例对象。

```js
var obj = Object.create(null);

obj.__proto__; // undefined，传入 null 作为参数时，可以创建一个原型为 null 的对象，没有 __proto__ 属性
```

所以，理解 JavaScript 运行时的原型工作原理的关键就是，理清**构造器，原型对象与实例对象之间的关系**。以前面 Engineer 为例，

![image.png](https://i.loli.net/2020/09/23/sHfAnPJRSXz7ujV.png)

（图片说明：构造器，原型对象，实例对象之间的关系）

## 原型链

前面我们提到，JavaScript 中，**读一个属性时，如果对象本身没有，则会继续访问对象的原型，直到原型为空或者找到为止**。这其实描述的就是原型链，下面我们通过一个例子来深入地理解下。

### 什么是原型链

我们先用构造函数实例化一个对象，然后尝试调用实例对象的几个方法，

```js
function Engineer(name) {
  this.name = name;
}

Engineer.prototype.coding = function() {
  console.log('write less, do more.');
};

var campcc = new Engineer('campcc');

campcc.coding(); // "write less, do more."
campcc.toString(); // "[object Object]"
```

可以看到，我们并没有在实例对象里面定义 `coding` 和 `toString` 方法，但是它们却能够被成功调用。这是因为，在 JavaScript 中，当我们试图访问一个属性时，如果对象本身没有，则会继续访问对象的原型，直到原型为空或者找到为止。

如果我们尝试访问一个原型链上不存在的方法会怎样？

```js
campcc.map(); // "Uncaught TypeError: engineer.map is not a function"
```

报错了，因为原型链上找不到此方法，原型链的查找会一直持续，直到找到名字匹配的属性或者到达原型链的末尾，而根据定义，**原型链的末尾是 null**。

```js
Object.prototype.__proto__ === null; // true，原型链的末尾是 null
```

所以，**原型链其实也是一条查找链，它由原型对象（prototype）和原型对象的 `__proto__` 组成**。

几乎所有的 JavaScript 对象都是位于原型链顶端的 Object 的实例，但是有两个例外。

- Object.prototype，根据定义，它的原型是 null
- Object.create(null) 创建的对象，它的原型为 null

### 属性遮蔽（property shadowing）

**原型链的查找过程中，如果遇到同名的属性，位于原型链底端的属性会被优先应用，这叫做 “属性遮蔽（property shadowing）”**。

以 Engineer 为例，我们在构造器内部和构造器的原型对象上初始化一个同名的属性，

```js
function Engineer(name) {
  this.name = name;
}

Engineer.prototype.name = 'engineer';

var engineer = new Engineer('campcc');

engineer.name; // "campcc"，根据属性遮蔽原则，在原型链的查找过程中，如果有同名属性，位于原型链底端的属性会被优先应用
```

可以直观的看到，这里不会打印 `"engineer"`，因为在原型链查找的过程中，实例对象中就已经存在 `name` 属性了。

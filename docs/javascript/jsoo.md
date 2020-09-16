---
title: 面向对象
order: 3
---

# 面向对象

JavaScript [语言标准](http://www.ecma-international.org/ecma-262/6.0/)明确地告诉我们，**JavaScript 是一门面向对象的语言**。

但令人疑惑的是，JavaScript 中确没有 “类” 的概念（在 ES6 之前），这与其他一些面向对象的语言不同。所以你可能会发现，在早期的社区，大量的 JavaScript 程序员试图在 JavaScript 特有的原型体系上，把 JavaScript 变得更像是基于类的编程，进而产生了很多所谓的“框架”，比如 [PrototypeJS](https://github.com/prototypejs/prototype)、[Dojo](https://github.com/dojo/dojo)。

不可否认的是，在描述对象的编程语言中，最为成功的流派是使用 “类” 的方式来描述对象，这诞生了诸如 C++、Java 等流行的编程语言。但实际上，**“基于类” 并非面向对象的唯一形态**，JavaScript 也不需要去模拟类，因为 JavaScript 的原型系统，本就是一个非常优秀的抽象对象的形式。

想要理解 JavaScript 中的面向对象，就得深扒一下语言的设计思路（语言的历史），在这之前，我们先来聊一聊什么是对象。

## 什么是对象

在 [计算机科学](https://en.wikipedia.org/wiki/Computer_science) 中，**对象是一个具有唯一标识符的内存地址**。

这其实是对象的本质特征之一，在 [《面向对象分析与设计》](https://book.douban.com/subject/11509672/) 一书中，Grandy Booch 认为对象应该具有以下特点，

- 具有唯一标识性：即使完全相同的两个对象，也并非同一个对象。
- 具有状态：同一个对象可能处于不同的状态之下。
- 具有行为：即对象的状态，可能因为它的行为产生变迁。

在不同的语言中，**对象的唯一标识性都是通过内存地址来体现的**。比如在 JavaScript 中，下面两个看似相同的对象是不相等的，因为它们具有不同的唯一标识的内存地址，

```js
var o1 = { foo: 'foo' };
var o2 = { foo: 'foo' };
o1 == o2; // false
```

除了唯一标识，对象还应该具有 **“状态和行为”**，这一点在不同的编程语言中可能会使用不同的术语来描述。比如 C++ 中叫 “成员变量” 和 “成员函数”，Java 中叫 “属性” 和 “方法”。但在 JavaScript 中，状态和行为被统一抽象为 **“属性”**，因为 JavaScript 中的函数本质上也是一个特殊对象。

```js
var obj = {
  name: '',
  setName(name) {
    this.name = name;
  },
};

obj.setName('obj');
obj.name; // obj
```

比如上面的 `obj` 对象，尽管这里的 `setName` 是一个函数，但 `name` 和 `setName` 其实都是它的两个普通属性，它们共同体现了对象的两个特征：**状态和行为**。（我们可以通过 `setName` 方法改变对象 `obj` 的 `name` 属性的值）

值得一提的是，与其他的基于类的、静态的对象设计语言不同，**JavaScript 中允许运行时向对象添加属性**，这使得 JavaScript 中的对象具有高度的动态性。比如下面这段代码，我们完全可以先定义一个对象，然后在为它添加属性，

```js
var obj = {};
obj.foo = 'foo';
console.log(obj); // { foo: 'foo' }
```

尽管 JavaScript 中的对象和我们理解一些主流的 “对象” 在设计上可能不同，但是它们都很好的体现了对象的几个基本特征：**唯一标识，状态和行为**。你可能有疑惑为什么 JavaScript 会这样去设计对象，这得从当年 JavaScript 的创始人 [Brendan Eich](https://en.wikipedia.org/wiki/Brendan_Eich) 说起。

## Brendan Eich 与 JavaScript

1995 年，Netscape（网景）公司雇佣了程序员 Brendan Eich 开发一种网页脚本语言，用来控制浏览器的行为。

Brendan Eich 有很强的函数式编程（FP）背景，最初希望以 [Scheme](<https://en.wikipedia.org/wiki/Scheme_(programming_language)>) 语言（函数式语言鼻祖 LISP 语言的一种方言）为蓝本，实现这种新语言。Brendan Eich 只用了 10 天的时间，就完成了语言第一版的设计，它是一门基于原型的语言。

当时正值 Java 火热（Sun 公司 1994 年发布 Java，并在市场推广活动中非常成功），因为一些公司的政治原因，JavaScript 推出之时，管理层就要求它去模仿 Java。不得已，Brendan Eich 在 **“原型运行时”** 的基础上又引入了 new、this 等语言特性，使之 **“看起来语法更像 Java”**，而 Java 正是基于类的面向对象的代表语言之一。所以，你会发现 JavaScript 的编程风格其实是函数式编程（FP）和面向对象编程（OOP）的一种混合体。

我们不得不承认，JavaScript 确实有点半吊子出家的感觉，但也正因为如此，JavaScript 拥有了完全运行时的对象系统，这使得它可以模仿多数面向对象编程范式（JavaScript 是世界上最好的编程语言，滑稽 ):），接下来我们就来具体看下，JavaScript 如何基于原型或者基于类实现面向对象。

## 面向对象

面向对象编程（OOP）通常与面向过程编程（OPP）相对比，是的，理解一个编程范式最好的办法就是通过对比的方式举个例子。

下面是一个经典的例子。

假设现在的需求是，我们要实现一个 **五子棋** 游戏，

**面向过程（OP） 的设计思路**是，将问题拆解为一个个过程（步骤），

1. 开始游戏
2. 黑子先走
3. 绘制画面
4. 判断输赢
5. 轮到白子
6. 绘制画面
7. 判断输赢
8. 返回步骤 2
9. 输出最终结果

OPP 更注重过程，解决问题时，通常会把问题拆分为一个个的函数（每个函数看作一个过程）和数据（函数的参数），然后按照一定的顺序，执行完这些函数，问题就解决了。

**面向对象（OO） 的设计思路**是，将问题拆解为一个个类别（对象），

1. 黑白棋子，它们的行为基本是一模一样的
2. 棋盘系统，负责绘制画面
3. 规则系统，负责判定诸如犯规、输赢等

上面的每一个类别都可以看做一个对象，（棋子对象）负责接受用户输入，并告知（棋盘对象）棋子布局的变化，（棋盘对象）接收到了棋子的变化后，在屏幕上面绘制画面，同时利用（规则系统）来对棋局进行判定。

OOP 更注重对象，解决问题时，通常会把问题中的事物抽象为对象，每个对象有自己的属性和方法（对应我们之前讲的状态和行为），然后让对象去执行自己的方法，问题就解决了。

所以，面向对象的一个核心就是**如何去描述对象**，JavaScript 面向对象也不例外，描述对象的方式中最常见的有 “基于原型” 和 “基于类” 两种，我们先来了解下 JavaScript 中基于原型的面向对象实现。

### 基于原型

“基于原型” 的编程提倡程序员去关注一系列对象实例的行为，而后才去关心如何将这些对象，划分到最近的使用方式相似的原型对象，而不是将它们分成类。

在这种编程范式下，面向对象的实现是**通过 “复制” 的方式来创建新对象**。这种 “复制操作” 有两种实现方式，

- 一个是并不真的去复制一个原型对象，而是使得新对象持有一个原型的引用；
- 另一个是切实地复制对象，从此两个对象再无关联。

历史上的基于原型语言因此产生了两个流派，而 JavaScript 选择了前一种方式。

#### 原型系统

我们前面介绍由于 JavaScript 语言的历史，使得 JavaScript 中的原型系统相对复杂，如果我们抛开 JavaScript 用于模拟 Java 类的复杂语法设施（如 new、Function Object、函数的 prototype 属性等），原型系统可以说相当简单，可以用两条概括：

- 如果所有对象都有私有字段 [[prototype]] ，就是对象的原型；
- 读一个属性，如果对象本身没有，则会继续访问对象的原型，直到原型为空或者找到为止。

#### JavaScript 的原型

并无例外，JavaScript 的原型就是基于上述的原型系统实现的，只是在 ES6 之后的版本，JavaScript 提供了一些内置函数，可以直接地访问或操纵原型，

- Object.create，根据指定的原型创建新对象，原型可以是 null
- Object.getPrototypeOf，获得一个对象的原型
- Object.setPrototypeOf，设置一个对象的原型

利用这些方法，我们可以基于原型来实现面向对象，

```js
var car = {
  type: 'car',
  drive() {
    console.log('drive car');
  },
};

var lamborghini = Object.create(car, {
  drive: {
    writable: true,
    configurable: true,
    enumerable: true,
    value: function() {
      console.log('drive lamborghini');
    },
  },
});

lamborghini.drive(); // "drive lamborghini"

var gallardo = Object.create(lamborghini, {
  drive: {
    value: function() {
      console.log('drive gallardo');
    },
  },
});

gallardo.drive(); // "drive gallardo"
```

我们定义了一个汽车对象，又经过汽车做了一些修改创建了兰博基尼，随后创建了盖拉多，这里我们完全抛开了 “类” 的思维，利用 JavaScript 提供的原型来实现了抽象和复用。

### 基于类

“基于类” 的编程提倡使用一个关注分类和类之间关系开发模型。

在这类语言中，总是先有类，再从类去实例化一个对象。类与类之间又可能会形成继承、组合等关系。类又往往与语言的类型系统整合，形成一定编译时的能力。

早期版本的 JavaScript 中，“类” 的定义是一个私有属性 `[[class]]`。

语言标准为内置类型诸如 Number、String、Date 等指定了 `[[class]]`属性，以表示它们的类。

唯一可以访问 `[[class]]` 属性的方式是 `Object.prototype.toString`，

```js
Object.prototype.toString.call(String('foo')); // "[object String]"
```

因此，在 ES3 和之前的版本，JavaScript 中类的概念是相当弱的，它仅仅是**运行时的一个字符串属性**。

从 ES5 开始，`[[class]]` 私有属性被 `Symbol.toStringTag` 代替。

我们可以通过 `Symbol.toStringTag` 来自定义 `Object.prototype.toString` 的行为，

```js
var obj = { [Symbol.toStringTag]: 'MyObject' };
Object.prototype.toString.call(obj); // "[object MyObject]"
obj + ''; // "[object MyObject]"，在讲 JavaScript 内置类型的时候，我们说过也可以用字符串加法触发 Object.prototype.toString，对象在转原始类型（这里是字符串）时，会先调用自身的 toString 方法
```

直到 ES5，JavaScript 中还是没有显式的 “类” 语法。但 JavaScript 提供了 `new` 运算符，可以让函数对象在语法上跟类变得相似。

`new` 运算接受一个构造器和一组调用参数，实际上做了几件事：

- 以构造器的 prototype 属性为原型，创建新对象
- 将 this 和调用参数传给构造器，执行
- 如果构造器返回的是对象，则返回，否则返回第一步创建的对象

```js
function _new(constructor, ...arguments) {
  const obj = Object.create(constructor.prototype); // 1. 以构造器的 prototype 为原型，创建一个对象
  const result = constructor.apply(obj, arguments); // 2. 绑定 this，执行构造函数
  return typeof result === 'object' ? result : obj; // 3. 返回 new 出来的对象
}
```

基于 `new` 运算，我们可以在构造器中添加属性，或者在构造器的 `prototype` 上添加属性，来模拟 “类”：

```js
function constructor(name) {
  this.name = name;
  this.sayName = function() {
    console.log(this.name);
  };
}

constructor.prototype.sayHi = function() {
  console.log('Hi');
};

var instance = new constructor('foo');
instance.name; // "foo"
instance.sayName(); // "foo"
instance.sayHi(); // "Hi"
```

在后来的 ES6 版本中，引入了 `class` 关键字，并且在标准中删除了所有 `[[class]]` 相关的私有属性描述，类的概念正式从属性升级成语言的基础设施，从此，基于类的编程方式成为了 JavaScript 的官方编程范式。`new` 跟 `function` 搭配模拟 “类” 的怪异行为我们不再需要了，我们可以光明正大的通过 `class` 定义类。

```js
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }

  get area() {
    return this.calcArea();
  }

  calcArea() {
    return this.height * this.width;
  }
}
```

虽然本质上 `class` 只是一个语法糖，但是我们仍然推荐，在任何场景，都使用 ES6 的语法来定义类，而令 `function` 回归原本的函数语义。

类的写法实际上也是由原型运行时来承载的，逻辑上 JavaScript 认为每个类是有共同原型的一组对象，此外，类提供了继承能力，我们使用 `class` 来重写前面的汽车的例子，

```js
class Car {
  constructor(type) {
    this.type = type;
  }

  drive() {
    console.log('drive car');
  }
}

class Lamborghini extends Car {
  drive() {
    console.log('drive lamborghini');
  }
}

class Gallardo extends Lamborghini {
  drive() {
    console.log('drive gallardo');
  }
}

var lamborghini = Lamborghini();
var gallardo = Gallardo();
lamborghini.type; // "car"
gallardo.drive(); // "drive gallardo"
```

比起早期的原型模拟方式，使用 extends 关键字自动设置了 constructor，并且会自动调用父类的构造函数，这是一种更少坑的设计。所以当我们使用类的思想来设计代码时，应该尽量使用 `class` 来声明类，而不是用旧语法，拿函数来模拟对象。

（完）

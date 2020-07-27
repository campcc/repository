---
order: 2
group:
  title: 浏览器
  order: 2
---

# 事件流

我们以[红宝书](https://www.ituring.com.cn/book/2472)的一个故事作为开端，

当浏览器发展到第四代时（IE4 和 Netscape Communicator 4）,浏览器团队遇到一个很有意思的问题：页面的哪一部分会拥有特定的事件？

想象下在一张纸上有一组同心圆，如果你把手指放在圆心上，那么你的手指指向的不是一个圆，而是一组圆。两家公司的开发团队在看待浏览器事件方面还是一致的。如果你单击了某个按钮，那么同时你也单击了按钮的容器元素，甚至整个页面。我们这里说的**事件流**描述的是**从页面接收到事件的顺序**，但有意思的是，IE 和 Netscape 的开发团队居然提出了差不多是完全相反的事件流的概念。

**IE 的事件流是事件冒泡流，而 Netscape Communicator 的事件流是事件捕获流**。

好了，了解了背景故事，接下来让我们从 [DOM](https://www.w3.org/DOM/DOMTR) 开始一步步揭开事件流的神秘面纱。

## DOM

[DOM](https://www.w3.org/DOM/DOMTR)的全称叫文档对象模型，是 HTML 和 XML 文档的编程接口，它可以用多种语言来实现。我们知道 JavaScript 可以访问和操作 DOM，所以对于 Web 页面，他们之间的关系可能是这样的，

```html
API (Web 或 XML 页面) = DOM + JavaScript (脚本语言)
```

毋庸置疑，JavaScript 是最常用于 DOM 操作的语言，我们假定脚本语言就是 JavaScript，那么其实你还可以这么理解，

**DOM 就是一个接口规范，它的作用就是将网页转换成 JavaScript 对象，从而可以用脚本进行各种操作（比如增删改查）**。

## 节点

**节点是 DOM 的最小组成单位**。浏览器提供了一个原生的节点对象 **Node**，通过继承 Node 对象，节点根据类型又分为 7 种，

- Document：整个文档树的顶层节点
- DocumentType：doctype 标签（比如 `<!DOCTYPE html>`）
- Element：网页的各种 HTML 标签（比如 `<body>`、`<a>`等）
- Attr：网页元素的属性（比如 class="right"）
- Text：标签之间或标签包含的文本
- Comment：注释
- DocumentFragment：文档的片段

由节点组成的文档树形结构，就是我们说的 DOM 树，而节点相当于树的叶子。

## 事件

事件是指在编程时，系统内发生的动作或事情。对于 DOM 事件来说，可以理解为 **HTML 文档与脚本语言 JavaScript 之间的交互**。

## 事件流

事件流描述的是从页面中接收事件的顺序，也就是事件在页面中传播的顺序。背景故事中我们提到 IE 和 Netscape 提出了完全相反的两种事件流概念，下面我们来具体的了解下这两种事件流。

### 事件冒泡流

IE 提出的事件流叫事件冒泡（Event Bubbling），事件传播的顺序为：**从事件开始的节点，逐级往上传播至根节点**。

![bubbling.png](https://i.loli.net/2020/06/19/TosJ8KyYqCW1P5h.png)

### 事件捕获流

Netscape 提出的事件流叫事件捕获（Event Capture），事件传播的顺序为：**从根节点逐级往下传播，一直到目标节点**。

![captrue.png](https://i.loli.net/2020/06/19/qpd1x9eDaRhUCML.png)

## DOM 事件流

天下大势，分久必合，在后来的[DOM2 级](https://www.w3.org/TR/2000/CR-DOM-Level-2-20000510/)规范中，同时支持了冒泡和捕获两种事件流。DOM 事件级别到目前为止有 0-3 级，但是我们平常说的事件流一般是指 DOM 2 级事件，所以我们接下来详细地了解下 DOM 2 级事件流。

## DOM 2 级事件流

DOM 2 级规定的事件流分为三个阶段，依次为：**事件捕获阶段，处于目标阶段和事件冒泡阶段**。

![eventModel.png](https://i.loli.net/2020/06/19/WqFTLl15JgoCD3U.png)

除了整合了 IE 和 Netscape 事件流外，DOM 2 级还规范了**事件对象**和**事件处理函数**。

### 事件对象

DOM 事件发生后，会产生一个事件对象，并作为参数传递为事件处理函数。浏览器原生提供一个 Event 对象，其他所有的事件都是继承至这个对象。

通过 Event 构造的实例对象具有以下的属性和方法。

属性，

- type，返回一个字符串，表示事件类型
- bubbles，返回一个布尔值，表示当前事件是否会冒泡
- eventPhase，返回一个整数常量，表示事件目前所处的阶段
- timeStamp，返回一个毫秒时间戳，表示事件相对于网页加载成功后发生的时间
- target，返回原始触发事件的节点，不会随着事件传播改变
- currentTarget，返回事件当前所在的节点，随着事件传播改变
- cancelable，返回一个布尔值，表示事件是否可以取消
- cancelBubble，布尔值属性，如果设置为 true，相当于调用 Event.stopPropagation 方法
- defaultPrevented，返回一个布尔值，表示该事件是否调用过 Event.preventDefault 方法
- isTrusted，返回一个布尔值，表示该事件是否由真实的用户行为产生，比如用户点击

方法，

- preventDefault，取消浏览器对当前事件的默认行为，比如链接调整，空格滚动等
- stopPropagation，阻止事件在 DOM 中继续传播，防止再触发定义在别的节点上的监听函数，但是不包括在当前节点上其他的事件监听函数
- stopImmediatePropagation，阻止同一个事件的其他监听函数被调用，包含当前节点的监听函数
- composedPath，返回一个数组，成员是事件的最底层节点和依次冒泡经过的所有上层节点

### 事件处理函数

DOM 2 级规范还提供了两个标准的事件处理函数：**addEventListener，removeEventListener**。

```js
element.addEventListener(event, function, useCapture); // 用于绑定事件处理函数，当事件发生时，会执行监听函数 function

element.removeEventListener(event, function, useCapture); // 移除事件处理函数
```

方法接收三个参数，

- event，必须，事件名称，大小写敏感
- function，必须，监听函数，事件发生时调用
- useCaptrue，可选的布尔值，表示监听函数是否在捕获阶段（capture）触发，默认为 false（监听函数只在冒泡阶段被触发）

第三个参数除了布尔值 useCapture，还可以是一个属性配置对象。该对象有以下属性，

- capture：布尔值，表示该事件是否在捕获阶段触发监听函数
- once：布尔值，表示监听函数是否只触发一次，然后就自动移除
- passive：布尔值，表示监听函数不会调用事件的 preventDefault 方法。如果监听函数调用了，浏览器将忽略这个要求，并在监控台输出一行警告。

比如，如果希望监听函数只调用一次，可以打开属性配置对象的 once 属性：

```js
function handler() {
  console.log('once'); // 由于配置对象的 once 属性打开，监听函数只会执行一次
}

element.addEventListener('click', handler, { once: true });
```

## 事件代理

事件代理，也叫事件委托，是 DOM 性能优化常用的一种技术手段。

如果一个节点中有多个子节点需要注册事件，为了减少 DOM 操作，我们通常会将子节点的事件注册在父节点上，这就是**事件代理**。

```html
<ul id="list">
  <li>item 1</li>
  <li>item 2</li>
  <li>item 3</li>
  ......
  <li>item n</li>
</ul>
```

```js
// 通过事件代理统一处理子节点事件

document.getElementById('list').addEventListener('click', function(e) {
  // 兼容性处理
  var event = e || window.event;
  var target = event.target || event.srcElement;
  // 判断是否匹配目标元素
  if (target.nodeName.toLocaleLowerCase === 'li') {
    console.log('current target: ', target);
  }
});
```

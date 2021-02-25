---
title: DOM优化
order: 8
---

## 核心关注点

DOM 优化要解决的两个核心问题就是：“DOM 为什么这么慢” 以及 “如何使 DOM 变快”。

## DOM 为什么慢

《高性能 JavaScript》中有这样一句描述：把 DOM 和 JavaScript 各自想象成一个岛屿，它们之间用收费桥梁连接。

这是其中一个原因，JS 引擎和渲染引擎需要通信。每次操作 DOM 都要过一次桥，过桥次数多了，自然就有性能问题，因此 “减少 DOM 操作”的建议，并非空穴来风。

此外，对 DOM 的修改可能还会引发布局样式的更迭，也就是 **回流** 或 **重绘**。本质上是因为我们的 DOM 操作触发了渲染树(Rendering Tree)的变化。

## 回流与重绘

**回流**

DOM 修改引发的 DOM 元素尺寸的变化，比如修改宽高，隐藏元素等，也叫重排。

浏览器需要重新计算元素的几何属性（其他元素的几何属性和位置也会因此受到影响），然后再将计算的结果绘制出来。

**重绘**

DOM 修改引发的 DOM 元素样式的变化，比如修改了颜色或背景。

浏览器不需重新计算元素的几何属性、直接为该元素绘制新的样式。

回流较重绘开销较大，不过这两个说到底都是吃性能的，都不是什么善茬。后面我们专门开一小节来聊聊这两个“不善茬”。

## DOM 提速

知道了慢的原因，接下来就要想办法提提速了，减少 DOM 操作的通用思路就是，

**利用缓存少交 “过路费”，利用 JS 分压或 DOM Fragment 避免过度渲染**

比如下面这个可能有点极端但能够很好诠释的例子 🌰 ，往页面某个标签下插入 10000 个节点。

```js
for (var count = 0; count < 10000; count++) {
  document.getElementById('container').innerHTML +=
    '<span>我是一个小测试</span>';
}
```

有两个问题，第一，过路费交太多了，每一次循环都调用 DOM 接口重新获取了一次 container 元素，相当于每次循环都交了一次过路费。前后交了 10000 次过路费，但其中 9999 次过路费都可以用缓存变量的方式节省下来：

```js
// 只获取一次container
let container = document.getElementById('container');
for (let count = 0; count < 10000; count++) {
  container.innerHTML += '<span>我是一个小测试</span>';
}
```

第二，不必要的 DOM 更改太多了，10000 次循环里，修改了 10000 次 DOM 树，所以我们需要让 JS 分压，JS 层面的事情，JS 自己去处理，处理好了，再来找 DOM 打报告。

```js
let container = document.getElementById('container');
let content = '';
for (let count = 0; count < 10000; count++) {
  // 先对内容进行操作
  content += '<span>我是一个小测试</span>';
}
// 内容处理好了,最后再触发DOM的更改
container.innerHTML = content;
```

此外， DOM Fragment 类似于上述 content 变量，作为缓存批量化的 DOM 操作的一个优化手段，这一点在 jQuery、Vue 等优秀前端框架的源码中均有体现。

DOM Fragment 对象允许我们像操作真实 DOM 一样去调用各种各样的 DOM API，当我们试图将其 append 进真实 DOM 时，它会在乖乖交出自身缓存的所有后代节点后全身而退，完美地完成一个容器的使命，而不会出现在真实的 DOM 结构中。

```js
let container = document.getElementById('container');
// 创建一个DOM Fragment对象作为容器
let content = document.createDocumentFragment();
for (let count = 0; count < 10000; count++) {
  // span此时可以通过DOM API去创建
  let oSpan = document.createElement('span');
  oSpan.innerHTML = '我是一个小测试';
  // 像操作真实DOM一样操作DOM Fragment对象
  content.appendChild(oSpan);
}
// 内容处理好了,最后再触发真实DOM的更改
container.appendChild(content);
```

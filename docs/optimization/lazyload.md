---
title: 懒加载
order: 11
---

## Lazy-Load

懒加载是针对图片加载时机的优化。应用场景是一些图片量较大的网站。

原理是：页面打开的时候只加载首屏图片，后面的图片等待用户下拉的瞬间再即时去请求、即时呈现。

这样一来，性能的压力小了，用户的体验却没有变差——这个延迟加载的过程，就是 Lazy-Load。

我们以掘金为例。

给懒加载元素一个特写，

```html
<div
  data-v-b2db8566=""
  data-v-009ea7bb=""
  data-v-6b46a625=""
  data-src="https://user-gold-cdn.xitu.io/2018/9/27/16619f449ee24252?imageView2/1/w/120/h/120/q/85/format/webp/interlace/1"
  class="lazy thumb thumb"
  style="background-image: none; background-size: cover;"
></div>
```

我们注意到 style 内联样式中，背景图片设置为了 none。也就是说这个 div 是没有内容的，它只起到一个占位的作用。

其实不止掘金，很多网站都会有这样的骨架屏占位。

一旦我们通过滚动使得这个 div 出现在了可见范围内，那么 div 元素的内容就会发生变化，style 内联样式中的背景图片属性从 none 变成了一个在线图片的 URL。

这就是懒加载的实现思路。

## 手摸手简单实现

假设我们现在有一堆图片，

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Lazy-Load</title>
    <style>
      .img {
        width: 200px;
        height: 200px;
        background-color: gray;
      }
      .pic {
        // 必要的img样式
      }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="img">
        // 注意我们并没有为它引入真实的src
        <img class="pic" alt="加载中" data-src="./images/1.png" />
      </div>
      <div class="img">
        <img class="pic" alt="加载中" data-src="./images/2.png" />
      </div>
      <div class="img">
        <img class="pic" alt="加载中" data-src="./images/3.png" />
      </div>
      <div class="img">
        <img class="pic" alt="加载中" data-src="./images/4.png" />
      </div>
      <div class="img">
        <img class="pic" alt="加载中" data-src="./images/5.png" />
      </div>
      <div class="img">
        <img class="pic" alt="加载中" data-src="./images/6.png" />
      </div>
      <div class="img">
        <img class="pic" alt="加载中" data-src="./images/7.png" />
      </div>
      <div class="img">
        <img class="pic" alt="加载中" data-src="./images/8.png" />
      </div>
      <div class="img">
        <img class="pic" alt="加载中" data-src="./images/9.png" />
      </div>
      <div class="img">
        <img class="pic" alt="加载中" data-src="./images/10.png" />
      </div>
    </div>
  </body>
</html>
```

懒加载实现中，有两个关键的数值：**当前可视区域的高度**，**元素距离可视区域顶部的高度**。

**当前可视区域的高度**

现代浏览器及 IE9 以上的浏览器中，用 window.innerHeight，低版本 IE 中用 document.documentElement.clientHeight，

```js
const viewHeight = window.innerHeight || document.documentElement.clientHeight;
```

**元素距离可视区域顶部的高度**

这里我们选取 **getBoundingClientRect** 方法，它返回的是元素的 DOMRect 对象，兼容性也不错。

DOMRect 对象包含了一组用于描述边框的只读属性——left、top、right 和 bottom，单位为像素，其中 top 属性正好代表了元素距离可视区域顶部的高度。

冲 ~

```js
const images = document.getElementByTagName('img');
// 获取可视区域高度
const viewHeight = window.innerHeight || document.documentElement.clientHeight;

// num 用于统计当前显示到了哪一张图片，避免每次都从第一张图片开始检查
let num = 0;

function lazyload() {
  for (let i = num, len = images.length; i < len; i++) {
    // 计算元素是否到达可视区域
    const distance = viewHeight - images[i].getBoundingClientRect().top;
    // 可视区域的高度比元素距离可视区域顶部的高度大，说明元素已经显示了
    if (distance >= 0) {
      // 写入图片路径
      images[i].src = images[i].getAttribute('data-src');
      // 前 i 张图片已经加载完毕，下次从第 i+1 张开始
      num = i + 1;
    }
  }
}

// 监听滚动事件
window.addEventListener('scroll', lazyload, false);
```

我们实现了一个最基本的懒加载功能，但是可以注意到，scroll 事件是一个很危险的事件，它会频繁触发，所以上面的懒加载还需要优化，对于 lazyload 我们需要节流。

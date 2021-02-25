---
title: 防抖和节流
order: 12
---

## 防抖和节流的本质

前面我们提到，实现懒加载时，我们需要监听 scroll 事件，但是 scroll 事件会频繁触发回调导致大量的计算，引发页面抖动甚至卡顿。

为了规避这种情况，我们需要一些手段来控制事件被触发的频率，防抖和节流就是在这样的背景下出现的。

**本质是什么**

这两个东西都以**闭包**的形式存在。

通过对事件对应的回调函数进行包裹、以自由变量的形式缓存时间信息，最后用 setTimeout 来控制事件的触发频率。

## Throttle：第一个人说了算

节流的思路为：**在某段时间内，不管你触发了多少次回调，我都只认第一次，并在计时结束时给予响应**。

例子 🌰 ：机场大巴(假设 10 分钟一趟，第一个人上车，司机开始计时，10 分钟后不管还有没有人上车，都必须开走)。

```js
function throttle(fn, interval) {
  // 上次触发回调的时间，设置为 0 而不是 new Date()，是为了第一次一定会触发回调
  let last = 0;
  // 返回一个闭包函数，什么时候执行呢？
  return function() {
    // 缓存我们回调函数需要的上下文环境和参数
    const context = this;
    const args = arguments;
    // 记录本次触发回调的时间，强转为数值用于后面比较
    const now = +new Date();
    // 如果触发时间间隔大于阈值，更新上次触发回调的时间，执行回调
    if (now - last >= interval) {
      last = now;
      fn.apply(context, args);
    }
  };
}
```

## Debounce： 最后一个人说了算

防抖的思路为：**我会等你到底。在某段时间内，不管你触发了多少次回调，我都只认最后一次**。

例子 🌰 ：私家车(司机比较有耐心，上来一个人开始计时，10 分钟内又上来一个人，重新计时，直到后续 10 分钟没有人上车了再开走)

所以防抖会为每个回调函数设定新的定时器，执行与否由最后一个回调说了算,

```js
function debounce(fn, delay) {
  // 初始化定时器
  let timer = null;
  // 返回一个闭包函数，什么时候执行呢？
  return function() {
    // 缓存回调执行需要的环境和参数
    const context = this;
    const args = arguments;
    // 每次事件触发时，都清除上一个定时器，重新生成一个定时器
    if (timer) clearTimeout(timer);
    timer = setTimeout(function() {
      fn.apply(context, args);
    }, delay);
  };
}
```

## 节流优化防抖

通过上面的实现，我们发现 debounce(防抖) 存在一个的问题，他“太有耐心”了。

想象这么一个场景，如果用户的操作十分频繁，每次都没等 debounce 设置的 delay(延时) 时间结束就进行了下一次操作，这时会发生什么？

是不是回调函数被延迟了无数次，但是就是迟迟没有执行，用户迟迟得不到响应，同样会产生“页面是不是卡死了”的体验。

所以，为了避免弄巧成拙，我们需要借助 throttle 的思想，将 debounce 优化为一个有底线的防抖，

```js
// 究极进化版防抖节流，我们就叫他节流吧
function throttle(fn, delay) {
  let timer = null;
  let last = 0;
  // 返回一个闭包函数，什么时候执行呢？
  return function() {
    // 缓存回调执行的环境和参数
    const context = this;
    const args = arguments;
    const now = +new Date();
    // 如果时间间隔小于阈值，重置定时器(清除上一个，生成一个新的)，delay 时间后触发回调
    if (now - last < delay) {
      window.clearTimeout(timer);
      timer = setTimeout(function() {
        last = now;
        fn.apply(context, args);
      }, delay);
    } else {
      // 反之，如果时间间隔超出了我们设定的时间间隔阈值，那就不等了，无论如何要反馈给用户一次响应
      last = now;
      fn.apply(context, args);
    }
  };
}
```

---
title: EventEmitter
order: 1
nav:
  title: 博文
  order: 8
---

# EventEmitter

<Alert>参考链接：[深入浅出搞定 React，数据是如何在 React 组件之间流动的？（上）](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=510#/detail/pc?id=4853)</Alert>

本文我们用常用的设计模式 “发布-订阅” 实现一个组件通信的 EventEmitter。

## “发布-订阅”模式

“发布-订阅” 模式可谓是解决通信类问题的 “万金油”，在前端世界的应用非常广泛，比如：

- socket.io 模块
- Node.js 中的 EventEmitter 基类
- Vue.js 中的全局事件总线
- ... ...

“发布-订阅” 机制早期最广泛的应用，应该是在浏览器的 DOM 事件中。原生 JavaScript 开发中，我们经常会这样去绑定事件监听：

```js
target.addEventListener(type, listener, useCapture);
```

通过调用 addEventListener 创建一个事件监听器（订阅），然后当事件被触发时（发布），监听函数会被执行。

## API 设计

“发布-订阅”模式中有两个最关键的动作：事件的监听（订阅）和事件的触发（发布），基于这个思路，我们可以自然地想到下面几个基本方法：

- on：负责注册事件监听器，指定事件触发时的回调函数
- emit：负责触发事件，可以通过传参使其在触发的时候携带数据
- off：负责移除监听器

## 基于 JavaScript 编码实现

我们给这个类取一个高大上的名字：EventEmitter，在写之前，先要捋清楚思路。EventEmitter 需要解决什么问题？

### 1.如何绑定事件和监听函数的对应关系？

提到 “对应关系”，首先想到的就是 “映射”，我们可以用一个 Map 来存储事件与监听函数之间的对应关系，

```js
class EventEmitter {
  constructor() {
    this.eventMap = {}; // eventMap 是一个对象，用于存储事件与监听函数的对应关系
  }
}
```

### 2.如何实现事件订阅？

所谓 “订阅”，就是注册事件监听函数的过程。这是一个 “写” 的操作，具体来说就是把事件和对应的监听函数写入到 eventMap 里面去，

```js
/**
 * @param type 事件名称
 * @param handler 监听函数
 */
on(type, handler) {
  if(!(handler instanceof Function)) {
    throw new Error("handler expect to be a function");
  }
  // 判断 type 事件对应的队列是否存在
  if(!this.eventMap[type]) {
    // 如果不存在，新建该队列
    this.eventMap[type] = [];
  }
  // 如果存在，直接往队列里推入 handler
  this.eventMap[type].push(handler);
}
```

### 3.如何实现事件发布？

订阅操作是一个 “写” 操作，相应的，发布操作就是一个 “读” 操作。发布的本质是触发安装在某个事件上的监听函数，所以我们需要做的就是找到这个事件对应的监听函数队列，将队列中的 handler 依次执行出队，

```js
/**
 * @param type 事件类型
 * @param payload 携带的数据
 */
emit(type, payload) {
  // 如果存在事件类型的队列，将监听函数依次出队
  if(this.eventMap[type]) {
    this.eventMapp[type].forEach(handler => handler(payload));
  }
}
```

### 4.如何移除事件？

只进不出总是不太合理的，最后我们补充一个 off 方法，必要的时候用它来删除用不到的监听器，

```js
/**
 * @param type 事件类型
 * @param handler 需要移除的监听函数
 */
off(type, handler) {
  const eventQueue = this.eventMap[type];
  if(eventQueue) {
    // 无符号右移用于兼容如果传入的监听函数不存在时，-1 >>> 0 = 4294967295，这个数足够大，相当于没有对原数组做任何处理
    eventQueue.splice(eventQueue.indexOf(handler) >>> 0, 1);
  }
}
```

### EventEmitter

```js
class EventEmitter {
  constructor() {
    this.eventMap = {}; // eventMap 是一个对象，用于存储事件与监听函数的对应关系
  }

  on(type, handler) {
    if (!(handler instanceof Function)) {
      throw new Error('handler expect to be a function');
    }
    // 判断 type 事件对应的队列是否存在
    if (!this.eventMap[type]) {
      // 如果不存在，新建该队列
      this.eventMap[type] = [];
    }
    // 如果存在，直接往队列里推入 handler
    this.eventMap[type].push(handler);
  }

  emit(type, payload) {
    // 如果存在事件类型的队列，将监听函数依次出队
    if (this.eventMap[type]) {
      this.eventMap[type].forEach(handler => handler(payload));
    }
  }

  off(type, handler) {
    const eventQueue = this.eventMap[type];
    if (eventQueue) {
      // 无符号右移用于兼容如果传入的监听函数不存在时，-1 >>> 0 = 4294967295，这个数足够大，相当于没有对原数组做任何处理
      eventQueue.splice(eventQueue.indexOf(handler) >>> 0, 1);
    }
  }
}
```

下面我们简单测试下 EventEmitter 是否具备 “发布-订阅” 的能力，

```js
// 实例化 eventEmitter
const eventEmitter = new EventEmitter();
// 编写一个简单的监听函数
const handler = params => {
  console.log(`handler is triggered, the payload is ${params}`);
};
// 订阅 test 类型的事件
eventEmitter.on('test', handler);
// 触发 test 类型的事件，同时传入希望监听函数感知的数据
eventEmitter.emit('test', 'newState'); // handler is triggered, the payload is newState
// 移除监听器后，再触发
eventEmitter.off('test', handler);
eventEmitter.emit('test'); // undefined，emit 函数执行了，但是没有触发监听器，说明我们移除成功了
```

（完）

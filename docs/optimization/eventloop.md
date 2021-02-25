---
title: 异步更新
order: 9
---

## EventLoop

EventLoop 是 JavaScript 的运行机制，先来简单回顾下。

**宏任务与微任务**

常见的 macro-task 有：script(整体代码)，定时器三兄弟(setTimeout, setInterval, setImmediate)，I/O 操作，UI 渲染等

常见的 micro-task 有：Promise，process.nextTick，MutationObserver 等

事件循环的过程，

- 初始调用栈(执行栈)为空，micro 队列为空，macro 队列有且仅有一个任务 script(整体代码，或者叫全局上下文)
- 整体代码推入执行栈，开始执行同步代码，执行过程中如果遇到宏任务推入 macro 队列，如果遇到微任务推入 micro 队列，同步代码执行完成后 script 出队
- 此时执行栈为空，查询是否有异步微任务待执行(micro 队列是否为空)，如果有，执行所有的微任务(micro 队列清空)
- **执行渲染操作，更新页面**
- 检查是否存在 Web Worker 任务，如果有则对其进行处理
- 开始下一轮事件循环，查询宏任务队列是否非空，如果非空，取其中一个宏任务推入执行栈

上述的过程会一直循环，直到两个队列都被清空。需要注意的是，宏任务是一个一个出队执行的，微任务是一队一队执行的。

EventLoop 中值得我们关注的是 “渲染的时机”。

## 异步更新

假设我们现在要在异步任务里更新 DOM，那么应该把它包装成 micro 还是 macro。

我们更新 DOM 的时间点，应该尽可能靠近渲染的时机，这样才能保证 render 是有效的。根据 EventLoop 的机制，不难发现，

**当我们需要在异步任务中实现 DOM 修改时，把它包装成 micro 任务是相对明智的选择。**

而所谓的异步更新，就是当我们使用 Vue 或 React 提供的接口去更新数据时，这个更新并不会立即生效，而是会被推入到一个队列里。待到适当的时机，队列中的更新任务会被批量触发。

**Vue 中任务更新原理**

Vue 中每产生一个状态更新任务，会通过 nextTick 被塞进一个叫 callbacks 的数组（此处是任务队列的实现形式）中。这个任务队列在被丢进 micro 或 macro 队列之前，会先去检查当前是否有异步更新任务正在执行（即检查 pending 锁）。如果确认 pending 锁是开着的（false），就把它设置为锁上（true），然后对当前 callbacks 数组的任务进行派发（丢进 micro 或 macro 队列）和执行。

此外，Vue 的异步任务默认情况下都是用 Promise 来包装的，也就是是说它们都是 micro-task。这一点和我们前面描述的渲染时机的分析不谋而合。

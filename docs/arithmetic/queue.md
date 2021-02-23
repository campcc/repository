---
title: 队列
order: 5
---

### 1.栈模拟队列

使用栈实现队列的下列操作：push, pop, peek, empty

```js
// 思路：队列元素刚好是栈的逆序，我们可以用两个栈来模拟
const Queue = function() {
  this.inStack = [];
  this.outStack = [];
};

Queue.prototype.push = function(x) {
  this.inStack.push(x);
};

Queue.prototype.pop = function() {
  // outStack 为空时，将 inStack 的元素依次出栈放入 outStack
  if (!this.outStack.length) {
    while (this.inStack.length) {
      this.outStack.push(this.inStack.pop());
    }
  }
  // outStack 非空，里面的元素刚好是逆序的，直接将栈顶元素出栈
  return this.outStack.pop();
};

Queue.prototype.peek = function() {
  // peek 与 pop 类似，区别在于 pop 会移除队头元素，peek 只是读取
  const len = this.outStack.length;
  if (!len) {
    while (this.inStack.length) {
      this.outStack.push(this.inStack.pop());
    }
  }
  return this.outStack[len - 1];
};

Queue.prototype.empty = function() {
  // inStack 和 outStack 都为空，队列为空
  return !this.inStack.length && !this.outStack.length;
};
```

### 2.滑动窗口最大值问题

给定一个数组 nums 和滑动窗口的大小 k，请找出所有滑动窗口里的最大值

示例:

输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3 输出: [3,3,5,5,6,7]

```js
// 思路：传统的解法是用双指针 + 遍历的方式，对于滑动窗口还有一种复杂度较低的解法，双端队列
// 维护一个值递减的双端队列，滑动窗口的最大值就是双端队列的队头元素
function maxSlidingWindow(nums, k) {
  const len = nums.length;
  // 初始化结果数组和双端队列
  const res = [];
  const deque = [];
  // 开始遍历
  for (let i = 0; i < len; i++) {
    // 如果队尾元素小于当前元素，将队尾元素依次出队，直到队尾元素大于等于当前元素
    // 为了便于后面对队头元素是否溢出滑动窗口作判断，这里我们在双端队列中维护索引值
    while (deque.length && nums[deque[deque.length - 1]] < nums[i]) {
      deque.pop();
    }
    deque.push(i);
    // 队头元素已经溢出滑动窗口时，将队头元素出队(这里是其索引)
    while (deque.length && deque[0] <= i - k) {
      deque.shift();
    }
    // 判断滑动窗口的状态，只有在被遍历的元素索引大于等于 k 时，才更新结果数组
    // 滑动窗口的最大值就是双端队列的队头元素
    if (i >= k) {
      res.push(nums[deque[0]]);
    }
  }
  return res;
}
```

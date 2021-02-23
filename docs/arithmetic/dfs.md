---
title: DFS & BFS
order: 6
---

## DFS(深度优先搜索)

贯彻了 “不撞南墙不回头” 的原则：只要没有碰壁，就决不选择其它的道路，而是坚持向当前道路的深处挖掘，将“深度”作为前进的第一要素的搜索方法。

核心思想：试图穷举所有的完整路径。

递归的本质就是栈。DFS 的实现依赖栈。

## BFS(广度优先搜索)

从第一层开始，丢弃已访问的坐标、记录新观察到的坐标(FIFO)，BFS 算法的实现过程，和队列有着密不可分的关系。

### 1.经典全排列问题

给定一个没有重复数字的序列，返回其所有可能的全排列。

```js
function permute(nums) {
  const res = [];
  const used = {};

  function dfs(path) {
    // 递归边界条件：路径数组的长度和 nums 相等，说明路径已经找到了
    if (path.length === nums.length) {
      res.push(path.slice());
      return;
    }
    for (let num of nums) {
      // 判断当前的数有没有用过，用过直接跳过
      if (used[num]) continue;
      // 如果没有用过，放入路径数组，打上标识
      path.push(num);
      used[num] = true;
      // 针对当前路径继续递归
      dfs(path);
      // 回溯
      path.pop();
      // 使用的标志也要清空
      used[num] = false;
    }
  }

  dfs([]);
  return res;
}
```

### 2.子集问题

给定一组不含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。
说明：解集不能包含重复的子集。

示例: 输入: nums = [1,2,3]
输出: [[3], [1], [2], [1,2,3], [1,3], [2,3], [1,2], []]

```js
function subsets(nums) {
  const res = [];
  const subset = [];
  const len = nums.length;

  function dfs(index) {
    res.push(subset.slice());
    for (let i = index; i < len; i++) {
      subset.push(nums[i]);
      dfs(i + 1);
      subset.pop();
    }
  }

  dfs(0);
  return res;
}
```

### 3.子集剪枝

给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。

示例: 输入: n = 4, k = 2
输出: [[2,4], [3,4], [2,3], [1,2], [1,3], [1,4]]

```js
function combine(n, k) {
  const res = [];
  const subset = [];

  function dfs(index) {
    if (subset.length === k) {
      res.push(subset.slice());
      return;
    }
    for (let i = index; i <= n; i++) {
      subset.push(i);
      dfs(i + 1);
      subset.pop();
    }
  }

  dfs(1);
  return res;
}
```

---
title: 数组
order: 1
nav:
  title: 算法
  order: 2
---

### 前置知识

1. 创建数组

```js
// 一维数组创建
const arr = [];
const arr = new Array();
const arr = new Array(7); // 指定长度
const arr = new Array(7).fill(1); // 指定填充

// 二维数组创建，不能使用 fill，fill 入参如果是引用类型，填充时填充的就是入参的引用，导致所有的填充值都指向了同一个地址
// 简单且性能的 for 循环
const len = arr.length;
for (let i = 0; i < len; i++) {
  arr[i] = [];
}
```

2. 数组遍历

```js
// 一维数组
// 1.for 循环(性能最优)
for (let i = 0, len = arr.length; i < len; i++) {
  console.log(arr[i]);
}
// 2.forEach
arr.forEach((item, index) => {
  console.log(item, index);
});
// 3.map
const newArr = arr.map((item, index) => {
  console.log(item, index);
  return item;
});

// 二维数组
// 双层 for 循环遍历
const outerLen = arr.length;
for (let i = 0; i < outerLen; i++) {
  const innerLen = arr[i].length;
  for (let j = 0; j < innerLen; j++) {
    console.log(arr[i][j], i, j);
  }
}
```

3. 常用方法

增删

- push, 添加一个或多个元素到数组尾部，并返回数组新长度(入队，入栈)
- pop, 删除并返回最后一个元素(删除栈顶元素出栈，删除队尾元素)
- unshift, 添加一个或多个元素到数组头部，并返回数组新长度
- shift, 删除并返回第一个元素(删除队头元素出队，删除栈底元素)
- splice, 添加，删除，替换数组元素

遍历相关

- forEach, 无返回值的遍历，过程中无法 return
- map, 返回一个调用迭代函数后的新数组
- reduce, 返回一个调用 reducer 函数后的单值
- keys, 返回一个包含索引的迭代器对象
- values, 返回一个包含值的迭代器对象
- entries, 返回一个包含键值对数组的迭代器对象

查找判断相关

- filter, 返回一个包含所有通过测试函数元素的新数组
- some, 有一个元素通过测试，结果就为 true
- every, 所有元素通过测试，结果才为 true
- find, 返回第一个满足测试函数的元素值，如果没有返回 undefined
- findIndex, 返回第一个满足测试函数的元素索引，如果没有返回 -1
- indexOf, 返回指定元素的第一个索引，如果没有返回 -1
- lastIndexOf, 返回指定元素的最后一个索引，如果没有返回 -1
- includes, 包含指定元素值，返回 true

其他

- slice, 返回一个指定范围的新数组
- concat, 返回一个合并后的新数组
- sort, 按指定排序函数对数组进行排序，默认规则是将元素转换为字符串，然后比较字符的 UTF-16 编码
- fill, 固定值填充数组
- join, 返回一个分割符连接的字符串
- reverse, 返回倒叙后原数组
- Array.from, 返回一个从类数组或可迭代对象创建的新数组

### JavaScript 中的数组

不同于其他计算机语言，JavaScript 中的数组可能不是真正意义上的数组。

数组定义中，要求 “存储在连续的内存空间里” 这一必要条件，由于 JavaScript 类型系统的原因，

如果我们定义的是一个只包含一种类型元素的数组，比如下面这个纯数字数组，

```js
const arr = [1, 2, 3, 4, 5];
```

它对应的确实是连续的类型，但是如果数组中元素有多种类型，比如，

```js
const arr = ['1', 2, { foo: 'foo' }];
```

此时，这个数组不再具有数组特征，其底层使用哈希映射来分配内存空间，也就是说，本质上是通过对象链表实现的。

### 1.两数求和问题

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例:

给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9 所以返回 [0, 1]

```js
// 思路是将求和问题转化为求差
// 遍历过程中增加一个对象或 Map 缓存已经遍历过的值和其对应的索引（这里需要缓存索引的原因是题设要求我们最后返回的是索引值）
// 每次遍历新值前，先去缓存中查询 target 与新值的差值是否存在（如果存在，就能查到其对应的索引）
// Map 版
function twoSum(nums, target) {
  const len = nums.length;
  const map = new Map();
  for (let i = 0; i < len; i++) {
    const cachedIdx = map.get(target - nums[i]);
    if (cachedIdx !== undefined) {
      return [cachedIdx, i];
    }
    map.set(nums[i], i);
  }
}

// 对象模拟版
function twoSum(nums, target) {
  const len = nums.length;
  const diffs = {};
  for (let i = 0; i < len; i++) {
    const cachedIdx = diffs[target - nums[i]];
    if (cachedIdx !== undefined) {
      return [cachedIdx, i];
    }
    diffs[nums[i]] = i;
  }
}

const nums = [1, 2, 5, 11, 7];

twoSum(nums, 9); // [1, 4]
```

### 2.合并两个有序数组

给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 nums1 成为一个有序数组。

说明: 初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。 你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。

示例:

输入:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6], n = 3
输出: [1,2,2,3,5,6]

```js
// 双指针法（关键词：数组，有序），时间复杂度O(n+m)，空间复杂度O(1)
function merge(nums1, m, nums2, n) {
  // 初始化指向 nums1， nums2 尾部的双指针 i 和 j，以及 nums1 的尾部索引 k
  let i = m - 1,
    j = n - 1,
    k = m + n - 1;
  // 需要考虑三种情况，① 两个数组都没有遍历完时，双指针同步移动
  while (i >= 0 && j >= 0) {
    // 取双指针对应位置中较大的值，从后往前补
    if (nums1[i] >= nums2[j]) {
      nums1[k] = nums1[i];
      i--;
      k--;
    } else {
      nums1[k] = nums2[j];
      j--;
      k--;
    }
  }
  // ② nums1 先遍历完，留下 nums2 时，说明 nums2 中剩余的都是有序的小值，此时 nums1 的头部空出来了，把 nums2 直接补到整个 nums1 的前面即可
  while (j >= 0) {
    nums1[k] = nums2[j];
    j--;
    k--;
  }
  // ③ nums2 先遍历完，留下 nums1 时，此时 nums1 中剩余的都是有序的小值，不用处理
}
```

### 3.三数求和问题

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。

示例：

给定数组 nums = [-1, 0, 1, 2, -1, -4]， 满足要求的三元组集合为： [ [-1, 0, 1], [-1, -1, 2] ]

```js
// 双指针（这里是“对撞指针”）思想
// 遍历数组，固定当前遍历值，将左指针指向下一个，右指针指向数组尾部
// 移动指针，判断双指针对应位置的值之和与当前遍历值相加是否为零
// 如果不满足条件，由于是有序数组，必定：
// ① 相加之和大于0，说明右侧的数偏大了，右指针左移
// ② 相加之和小于0，说明左侧的数偏小了，左指针右移
// 优化：跳过重复值
function threeSum(nums) {
  // 双指针用来降低有序数组复杂度，题设是无序，先转为有序
  nums = nums.sort((a, b) => a - b);
  // 缓存最终结果和数组长度
  const res = [];
  const len = nums.length;
  // 遍历除了当前固定值外的元素
  for (let i = 0; i < len - 2; i++) {
    let j = i + 1; // 左指针
    let k = len - 1; // 右指针
    if (i > 0 && nums[i] === nums[i - 1]) continue; // 跳过重复值
    // 开始指针对撞
    while (j < k) {
      // 计算相加之和
      const sum = nums[i] + nums[j] + nums[k];
      // 和 = 0，缓存结果，指针继续对撞
      if (sum === 0) {
        res.push([nums[i], nums[j], nums[k]]);
        j++;
        k--;
      }
      // 和 > 0，右侧数大了，右指针后退
      if (sum > 0) {
        k--;
      }
      // 和 < 0，左侧数小了，左指针前进
      if (sum < 0) {
        j++;
      }
    }
  }

  return res;
}

threeSum([-1, 0, 1, 2, -1, -4]);
```

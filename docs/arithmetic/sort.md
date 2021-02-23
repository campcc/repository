---
title: 排序
order: 8
---

## 基础排序算法

### 1.冒泡排序

思路：重复 n 轮操作，n 为数组长度，每一轮对相邻元素两两比较，如果后者较大，交换位置，反之不动。由于每一轮操作后最大的值都会被冒泡到数组末尾，名冒泡排序。

```js
// 基础实现是通过双层循环交换元素
// 基本优化是限定内层循环的范围
// 还有一个定向优化，增加一个标识，如果数组本身有序只需要外层循环O(n)的时间复杂度
// 最终版本的时间复杂度：最好 O(n)，最坏 O(n²)，平均 O(n²)
function bubbleSort(arr) {
  const len = arr.length;
  let flag = false;
  for (let i = 0; i < len; i++) {
    for (let j = 0; j < len - 1 - i; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        flag = true;
      }
    }
    if (!flag) return arr;
  }
  return arr;
}
```

### 2.选择排序

思路：循环遍历数组，每次找出范围最小值放头部，然后缩小范围直到数组完全有序。

```js
// 选择排序无论什么情况下都要走内层循环，因此最好，最坏和平均的时间复杂度都为 O(n²)
function selectSort(arr) {
  const len = arr.length;
  // i 是左边界，0 ~ 倒数第二个元素
  for (let i = 0; i < len - 1; i++) {
    // 初始化最小值索引为当前区间的第一个元素
    let minIndex = i;
    // 循环比较更新最小值索引
    // j 是右边界，i ~ 最后一个元素
    for (let j = i; j < len; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }
    // 如果最小值索引对应的元素不是当前的头部元素，将最小值元素交换到当前头部
    if (minIndex !== i) {
      [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
    }
  }
  return arr;
}
```

### 3.插入排序

思路：当前元素前面的序列是有序的，从后往前寻找当前元素在前面的有序序列中的正确位置。

```js
// 时间复杂度：最好 O(n)，最坏 O(n²)，平均 O(n²)
function insertSort(arr) {
  const len = arr.length;
  // temp 用于缓存当前需要插入的元素
  let temp;
  // 有序数组初始化时就是第一个元素 [arr[0]]，我们从第二个元素开始进行插入
  for (let i = 1; i < len; i++) {
    // j 用于辅助定位 temp 应该插入的索引，刚开始就是 temp 原有的位置
    temp = arr[i];
    let j = i;
    // 循环比较 temp 和 j 前面一个元素
    // 如果前面一个元素较大，后移一位，为 temp 留出坑位
    while (j > 0 && arr[j - 1] > temp) {
      arr[j] = arr[j - 1];
      j--;
    }
    // 找到坑位后将缓存的当前元素 temp 插入
    arr[j] = temp;
  }
  return arr;
}
```

## 分治思想

分治思想，将一个大问题分解为若干个子问题，针对子问题分别求解后，再将子问题的解整合为大问题的解。

一般分三步走：

1. 分解子问题
2. 求解子问题
3. 合并子问题的解，得出大问题的解

### 1.归并排序

思路：分隔数组，直到单个数组只有一个元素，然后两两合并，确保每次合并的数组都是有序的，直到长度与原数组相等。

```js
// 思路：重复让人想到递归或迭代，分隔后又合并不就是回溯吗，所以果断递归
// 时间复杂度：O(nlog(n))，把切分 + 归并看做一轮，对于规模为 n 的数组，需要切分 log(n) 次，合并 n 次，切分单次 O(1) 忽略
function mergeSort(arr) {
  const len = arr.length;
  // 边界情况
  if (len <= 1) {
    return arr;
  }
  // 准备开始分割
  const mid = Math.floor(len / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid, len));
  // 合并
  arr = mergeArr(left, right);
  return arr;
}

// 合并两个有序数组，用双指针
function mergeArr(arr1, arr2) {
  const len1 = arr1.length;
  const len2 = arr2.length;
  const res = [];
  let i = 0;
  let j = 0;
  while (i < len1 && j < len2) {
    if (arr1[i] < arr2[j]) {
      res.push(arr1[i]);
      i++;
    } else {
      res.push(arr2[j]);
      j++;
    }
  }
  // 处理不等长情况，由于是有序数组，如果其中一个数组先被合并完，那直接将另一个数组拼接到尾部即可
  if (i < len1) {
    return res.concat(arr1.slice(i, len1));
  } else {
    return res.concat(arr2.slice(j, len2));
  }
}

mergeSort([2, 1, 3, 5, 4]);
```

### 2.快速排序

思路：选取一个元素作为基准，以基准将原始的数组筛选成较小和较大的两个子数组，然后递归地排序两个子数组。

```js
// 不考虑空间复杂度，最简单的思路是直接用两个数组缓存每次排序后的子数组，然后在对子数组递归排序
function quickSort(arr) {
  const len = arr.length;
  if (len <= 1) return arr;
  const pivotValue = arr[Math.floor(len / 2)];
  const left = [];
  const right = [];
  for (let i = 0; i < len; i++) {
    if (arr[i] < pivotValue) {
      left.push(arr[i]);
    }
    if (arr[i] > pivotValue) {
      right.push(arr[i]);
    }
  }
  return [...quickSort(left), pivotValue, ...quickSort(right)];
}

quickSort([5, 1, 3, 6, 2, 0, 7]);
```

上面的实现虽然比较好理解，但是空间复杂度很高，为了降低空间复杂度，可以借用双指针对数组进行原地快排。

```js
// 时间复杂度：最好：O(nlog(n))，最坏：O(n²)，平均：O(nlog(n))
function quickSort(arr, left = 0, right = arr.length - 1) {
  // 递归边界
  if (arr.length <= 1) return arr;
  // 计算下一次分割的索引
  const lineIndex = partition(arr, left, right);
  // 如果左边子数组的长度不小于 1 ，递归快排左数组
  if (left < lineIndex - 1) {
    quickSort(arr, left, lineIndex - 1);
  }
  // 右边同理
  if (lineIndex < right) {
    quickSort(arr, lineIndex, right);
  }
  return arr;
}

// 思路：双指针
function partition(arr, left, right) {
  // 选取基准值
  const pivotValue = arr[Math.floor(left + (right - left) / 2)];
  // 初始化左右指针
  let i = left;
  let j = right;
  // 只要指针不越界，循环找到分割的索引地址
  while (i <= j) {
    // 左指针指向元素比基准值小，右移左指针
    while (arr[i] < pivotValue) {
      i++;
    }
    // 右指针指向元素比基准值大，右指针左移
    while (arr[j] > pivotValue) {
      j--;
    }
    // 对撞后，如果右指针比左指针小，说明基准值的左边存在较大的元素，或者基准值的右边存在较小的元素
    // 这个时候需要交换左右指针对应的元素，确保两侧有序
    // 交换后继续对撞
    if (i <= j) {
      swap(arr, i, j);
      i++;
      j--;
    }
  }
  // 返回下一次分割的索引，也就是左指针
  return i;
}

function swap(arr, i, j) {
  [arr[i], arr[j]] = [arr[j], arr[i]];
}

quickSort([5, 1, 3, 6, 2, 0, 7]);
```

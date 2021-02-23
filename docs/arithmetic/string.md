---
order: 2
title: 字符串
---

### 1.反转字符串

```js
function reverse(str) {
  return str
    .split('')
    .reverse()
    .join('');
}
```

### 2.判断是否是回文字符串

```js
// 反转字符串判断是否相等
function isPalindromicString(str) {
  return (
    str
      .split('')
      .reverse()
      .join('') === str
  );
}

// 利用对称性
function isPalindromicString(str) {
  for (let i = 0, len = str.length; i < len / 2; i++) {
    // 遍历前半部分，判断是否与后半部分对称
    if (str[i] !== str[len - i - 1]) return false;
  }
  return true;
}
```

### 3.回文字符串衍生判断

给定一个非空字符串 s，最多删除一个字符。判断是否能成为回文字符串。

```js
// 利用对称性和双指针

// 工具函数，判断给定起始位置的字符串是否是回文字符串
function isPalindrome(str, start, end) {
  while (start < end) {
    if (str[start] !== str[end]) return false;
    start++;
    end--;
  }
  return true;
}

function validPalindrome(str) {
  const len = str.length;
  // 初始化两个首尾指针
  let i = 0,
    j = len - 1;
  // 当首尾指针对应的值相等时，对撞
  while (i < j && str[i] === str[j]) {
    i++;
    j--;
  }
  // 如果首尾指针对应的值不相等，尝试跳过其中一个指针后判断字符串是否仍是回文
  if (isPalindrome(str, i + 1, j)) {
    return true;
  }
  if (isPalindrome(str, i, j - 1)) {
    return true;
  }
  // 如果尝试跳过后仍无法得到回文数，返回 false
  return false;
}
```

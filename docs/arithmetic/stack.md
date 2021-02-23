---
title: 栈
order: 4
---

### 1.经典括号问题

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
注意空字符串可被认为是有效字符串。

```js
// 思路：括号成立意味着对称，与栈的出入顺序是契合的
function isValid(s) {
  const len = s.length;
  if (!s) return true;
  if (len % 2 === 1) return false;

  const pairs = {
    '(': ')',
    '{': '}',
    '[': ']',
  };

  const stack = [];

  for (let i = 0; i < len; i++) {
    const ch = s[i];
    // 如果是左括号，对应的右括号入栈
    if (['(', '{', '['].includes(ch)) {
      stack.push(pairs[ch]);
    } else {
      // 如果不是左括号，那一定是栈顶的右括号，否则判定为无效
      if (stack.pop() !== ch) return false;
    }
  }
  // 如果所有的括号都能配对成功，那么最后栈应该是空的
  return !stack.length;
}

isValid('({[]})');
```

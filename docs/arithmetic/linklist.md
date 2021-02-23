---
title: 链表
order: 3
---

### 1.有序链表合并

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有结点组成的。

示例：

输入：1->2->4, 1->3->4 输出：1->1->2->3->4->4

```js
// 处理链表的本质，就是处理链表结点之间的指针关系
function ListNode(val) {
  this.val = val;
  this.next = null;
}

function mergeTwoLists(l1, l2) {
  // 定义头结点
  let head = new ListNode();
  // 穿线的指针，初始指向头结点
  let cur = head;
  // 开始穿线 ~
  while (l1 && l2) {
    // 确定接下来要穿的是哪一个
    if (l1.val <= l2.val) {
      // l1 结点值较小，穿 l1
      cur.next = l1;
      // l1 指针向前
      l1 = l1.next;
    }
    cur.next = l2;
    l2 = l2.next;
    // 穿线的指针往前，继续
    cur = cur.next;
  }
  // 穿完后，将剩余的较长的链表连接到尾部
  cur.next = l1 !== null ? l1 : l2;
  // 返回起始结点
  return head.next;
}
```

### 2.重复结点删除

给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。

示例:

输入: 1->1->2->3->3
输出: 1->2->3

```js
function deleteDuplicates(head) {
  // 设定 cur 指针，初始位置为链表的第一个结点
  let cur = head;
  // 遍历链表
  while (cur !== null && cur.next !== null) {
    // 如果当前结点值与下一个结点值相等(重复)
    if (cur.val === cur.next.val) {
      // 删除靠后的结点(去重)
      cur.next = cur.next.next;
    }
    // 如果不重复，继续遍历
    cur = cur.next;
  }
  return head;
}
```

给定一个排序链表，删除所有含有重复数字的结点，只保留原始链表中没有重复出现的数字。

示例:

输入: 1->2->3->3->4->4->5
输出: 1->2->5

```js
// 利用 dummy 结点(假想的第一个结点的前驱结点)
function deleteDuplicates3(head) {
  // 边界情况
  if (!head || !head.next) return head;
  // 构造虚拟的 dummy 结点
  let dummy = new ListNode();
  // dummy 结点永远指向链表的头部结点
  dummy.next = head;
  // 拷贝一个 cur 结点，开始遍历
  let cur = dummy;
  // 遍历条件为，至少有两个结点
  while (cur.next && cur.next.next) {
    // 对 cur 后面的两个结点进行比较
    if (cur.next.val === cur.next.next.val) {
      // 如果发现值重复，记录下这个值，用于后续比较
      let val = cur.next.val;
      while (cur.next && cur.next.val === val) {
        // 如果后一个结点的值与记录值重复，删除后一个结点
        cur.next = cur.next.next;
      }
    }
    // 没有重复值，继续遍历
    cur = cur.next;
  }
  // 返回链表的起始结点
  return dummy.next;
}
```

### 3.指定位置删除

给定一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

示例：

给定一个链表: 1->2->3->4->5, 和 n = 2.
当删除了倒数第二个结点后，链表变为 1->2->3->5.

```js
// 思路：删除操作比较好实现，关键在于定位，倒数第 n 个就是正数的第 len - n + 1 个，求链表长度是关键
// 求长度需要一次遍历，删除需要一次遍历，使用双指针可以解决
// 针对此题，还有一种更巧的解法，是将双指针变为一快一慢的快慢指针
// 快指针先前进 n 步，然后快慢指针一起前进，当快指针到达链表最后一个节点时，慢指针所在的位置刚好就是需要删除结点的前驱结点(想一下为什么是前驱)
function removeNthFromEnd(head, n) {
  const dummy = new ListNode();
  // 空间换时间
  dummy.next = head;
  let fast = dummy;
  let slow = dummy;
  // 快指针先走 n 步
  while (n !== 0) {
    fast = fast.next;
    n--;
  }
  // 快慢指针同时前进，直到快指针到达最后一个节点
  while (fast.next) {
    fast = fast.next;
    slow = slow.next;
  }
  // 删除慢指针的后一个结点
  slow.next = slow.next.next;
  // 返回头结点
  return dummy.next;
}
```

### 4.反转链表

定义一个函数，输入一个链表的头结点，反转该链表并输出反转后链表的头结点。

示例:

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL

```js
// 思路，当前结点指向前驱结点，当前结点的后一个节点指向当前结点，三个结点的爱恨情仇，所以我们需要三个指针，cur, prev, next
function reverseList(head) {
  // 前驱结点刚开始不存在
  let prev = null;
  // 当前结点默认指向头结点
  let cur = head;
  // 然后开始遍历，只要当前结点存在，就一直反转
  while (cur !== null) {
    // 先缓存当前结点的下一个结点，不然待会反转后它就不见了
    let next = cur.next;
    // 开始反转
    cur.next = prev;
    // 当前结点也需要缓存，不然下次遍历就不见了，反转后当前结点就变成了前驱结点
    prev = cur;
    // cur 前进一步，继续遍历
    cur = next;
  }
  // 最后前驱结点就是反转后的链表头结点
  return prev;
}
```

反转从位置 m 到 n 的链表。请使用一趟扫描完成反转。1 ≤ m ≤ n ≤ 链表长度。

示例:

输入: 1->2->3->4->5->NULL, m = 2, n = 4
输出: 1->4->3->2->5->NULL

```js
// 思路：反转操作我们已经 get 了，此题的关键在于定位到需要反转的区间，缓存区间的前驱结点和后一个结点，反转完成后再拼接
function reverseBetween(head, m, n) {
  let prev, cur, leftHead;
  let dummy = new ListNode();
  dummy.next = head;
  // 游标指针，用于遍历，定位区间位置
  let p = dummy;
  // 先找到区间起始位置的前驱结点，缓存
  for (let i = 0; i < m - 1; i++) {
    p = p.next;
  }
  // 缓存前驱结点到 leftHead
  leftHead = p;
  // 初始化反转区间需要的指针 prev 和 cur
  // start 是反转区间的第一个结点
  let start = leftHead.next;
  // prev 指向 start
  prev = start;
  // cur 是 start 的下一个结点
  cur = start.next;
  // 开始反转
  for (let i = m; i < n; i++) {
    // 缓存 next 结点
    let next = cur.next;
    // 反转
    cur.next = prev;
    // 缓存 cur 结点，prev 和 cur 指针前进一步
    prev = cur;
    cur = next;
  }
  // 反转完成后，区间的第一个结点是 prev，最后一个节点是 cur
  // 开始拼接节点
  leftHead.next = prev;
  start.next = cur;
  // 返回头结点
  return dummy.next;
}
```

### 5.环形链表

给定一个链表，判断链表中是否有环。

示例：

输入：3->2->0->4, 4->2 输出：true

```js
// flag 解法
function hasCycle(head) {
  while (head) {
    if (head.flag) return true;
    head.flag = flag;
    head = head.next;
  }
  return false;
}

// JSON.stringify，原理：不能序列化含有循环引用的结构
function hasCycle(head) {
  try {
    JSON.stringify(head);
    return false;
  } catch (err) {
    return true;
  }
}

// 快慢指针解法
// 原理：遍历单链表，快指针一次走两步，慢指针一次走一步，如果单链表中存在环，则快慢指针终会指向同一个节点
// 否则直到快指针指向 null 时，快慢指针都不可能相遇
function hasCycle(head) {
  if (!head || !head.next) return false;
  let slow = head;
  let fast = head.next;
  while (fast !== slow) {
    if (!fast || !fast.next) return false;
    slow = slow.next;
    fast = fast.next.next;
  }
  return true;
}
```

给定一个链表，返回链表开始入环的第一个结点。 如果链表无环，则返回 null。

示例：

输入：3->2->0->4, 4->2 输出：2

```js
function detectCycle(head) {
  while (head) {
    if (head.flag) return head;
    head.flag = true;
    head = head.next;
  }
  return null;
}
```

给定一个链表，返回链表环的长度。 如果链表无环，则返回 null。

示例：

输入：3->2->0->4, 4->2 输出：3

```js
function detectCycleLen(head) {
  if (!head || !head.next) return null;
  let slow = head;
  let fast = head.next;
  let count = 0; // 相遇次数
  let len = 0; // 环长度
  let flag = false; // 是否相遇
  while (fast && fast.next) {
    // 第二次相遇后返回环长度
    if (count === 2) return len;
    if (flag) len++;
    if (fast === slow) {
      // 第一次相遇后开始计算
      flag = true;
      count++;
    }
    slow = slow.next;
    fast = fast.next.next;
  }
  return null;
}
```

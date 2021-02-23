---
title: 二叉树
order: 7
---

### 1.经典：迭代算法实现二叉树的遍历

给定一个二叉树，返回它的先序遍历，后序遍历，中序遍历序列。

示例:

输入: [1,null,2,3],

```bash
1
 \
  2
 /
3
```

先序遍历输出 [1, 2, 3]，后序遍历输出 [2, 1, 3]，中序遍历输出 [1, 3, 2]

先序遍历：

```js
// 思路：利用栈，控制出入栈顺序
function preorderTraversal(root) {
  // 定义结果数组和边界条件
  const res = [];
  if (!root) return res;
  // 初始化栈，根节点入栈
  const stack = [];
  stack.push(root);
  // 开始遍历，只要栈不为空，重复出栈，入栈操作
  while (stack.length) {
    // 缓存栈顶节点，作为当前节点
    const cur = stack.pop();
    // 先序遍历的顺序是：根 -> 左 -> 右
    res.push(cur.val);
    // 如果子树有右节点，入栈
    if (cur.right) stack.push(cur.right);
    // 如果子树有左节点，入栈
    if (cur.left) stack.push(cur.left);
    // 继续遍历子树
  }
  return res;
}
```

后序遍历：

```js
function postorderTraversal(root) {
  const res = [];
  if (!root) return res;
  const stack = [];
  stack.push(root);
  while (stack.length) {
    const cur = stack.pop();
    rs.unshift(cur.val);
    if (cur.left) stack.push(cur.left);
    if (cur.right) stack.push(cur.right);
  }
  return res;
}
```

中序遍历：

```js
function inorderTraversal(root) {
  const res = [];
  const stack = [];
  let cur = root;
  // 边界条件有两个，stack 很好理解，栈不为空代表遍历未完成，cur 的原因我们后面解释
  while (cur || stack.length) {
    // 一路向左，缓存所有的左节点
    while (cur) {
      stack.push(cur);
      cur = cur.left;
    }
    // 取出栈顶节点(左节点)
    cur = stack.pop();
    // 中序遍历的顺序为：左 -> 根 -> 右，这里只需要将当前结点推入结果数组然后尝试读取其右节点即可，原因如下
    // 此时的栈顶节点有两种情况，如果栈顶节点是叶子节点，尝试读取其右节点会返回 null，此时下一次遍历内层循环会跳过，接着就回溯到了该节点的父节点，刚好满足 `左 -> 根` 的顺序
    // 如果栈顶节点不是叶子节点，尝试读取其右节点会取到一个存在的节点，此时又会一路向左，重复刚刚这个或回溯，或一路向左的过程，如果这个右节点没有子的左节点，那么跳出内层循环后，紧接着被推入结果数组的就是右节点本身，符合 `根 -> 右` 的顺序
    // 这也是为什么外层循环需要将 cur 作为边界条件的原因
    res.push(cur.val);
    cur = cur.right;
  }
  return res;
}
```

### 2.二叉树的层序遍历

给你一个二叉树，请你返回其按层序遍历得到的节点值。（即逐层地，从左到右访问所有节点）。

```js
function levelOrder(root) {
  const res = [];
  if (!root) return res;
  // 一想到层序遍历就是 BFS，就是队列
  const queue = [];
  // 初始化队列
  queue.push(root);
  // 冲冲冲
  while (queue.length) {
    // level 用来缓存层级的节点
    const level = [];
    const len = queue.length;
    // 循环遍历当前层级的所有节点
    for (let i = 0; i < len; i++) {
      // 首先取出队头节点
      const top = queue.shift();
      level.push(top.val);
      if (top.left) queue.push(top.left);
      if (top.right) queue.push(top.right);
    }
    res.push(level);
  }
  return res;
}
```

### 3.翻转二叉树

翻转一棵二叉树。

示例：

输入：

```bash
     4
   /   \
  2     7
 / \   / \
1   3 6   9
```

输出：

```bash
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

```js
// 思路：每一颗子树都需要交换其左右节点，所以我们以递归的方式，遍历树中的每一个节点，并将每一个节点的左右子节点进行交换即可
function invertTree(root) {
  if (!root) return root;
  let left = invertTree(root.left);
  let right = invertTree(root.right);
  root.left = right;
  root.right = left;
  return root;
}
```

### 4.二叉搜索树(BST)的增删查

```js
function seachBST(root, n) {
  if (!root) return;
  if (root.val === n) return root;
  if (root.val < n) root.right = seachBST(root.right, n);
  if (root.val > n) root.left = seachBST(root.left, n);
}

function insertIntoBST(root, n) {
  if (!root) {
    root = new TreeNode(n);
    return root;
  }
  if (root.val < n) root.right = insertIntoBST(root.right, n);
  if (root.val > n) root.left = insertIntoBST(root.left, n);
}

function deleteNode(root, key) {
  // 要删除得先找到这个节点，如果找不到直接返回
  if (!root) return;
  if (root.val < key) root.right = deleteNode(root.right, key);
  if (root.val > key) root.left = deleteNode(root.left, key);
  if (root.val === key) {
    // 找到到节点后，准备开始删，有几种情况
    // 1.当前节点是叶子节点，直接删
    // 2.当前节点有左子树，删除当前节点实质上就是：将左子树里值最大的节点移动到当前节点进行覆盖
    // 3.当前节点有右子树，将右子树里值最小的节点移动到当前节点进行覆盖
    // 4.当前节点同时存在左子树和右子树，2，3 都可以，这里我们暂不考虑平衡性，默认选择 2 的方式
    if (!root.left && !root.right) {
      root = null;
    } else if (root.left) {
      // 首先找到左子树的最大值节点
      const maxLeft = findMax(root.left);
      // 覆盖当前节点的值
      root.val = maxLeft.val;
      // 删掉最大值节点
      root.left = deleteNode(root.left, maxLeft.val);
    } else {
      const minRight = findMin(root.right);
      root.val = minRight.val;
      root.right = deleteNode(root.right, minRight.val);
    }
  }

  return root;
}

function findMax(root) {
  while (root.right) {
    root = root.right;
  }
  return root;
}

function findMin(root) {
  while (root.left) {
    root = root.left;
  }
  return root;
}
```

### 5.判定二叉树

给定一个二叉树，判断其是否是一个有效的二叉搜索树。

```js
function isValidBST(root) {
  return dfs(root, -Infinity, Infinity);

  function dfs(root, min, max) {
    if (!root) return true;
    if (root.val >= max || root.val <= min) return false;
    return dfs(root.left, min, root.val) && dfs;
  }
}
```

### 6.判定平衡二叉树

定义：平衡二叉树是任意节点的左右子树的高度差的绝对值都不大于 1 的二叉搜索树。

关键点：

1. 任意结点
2. 左右子树高度差绝对值都不大于 1
3. 二叉搜索树

```js
function isBalanced(root) {
  let flag = true;
  function dfs(root) {
    if (!root || !flag) {
      return 0;
    }
    const left = dfs(root.left);
    const right = dfs(root.right);
    if (Math.abs(left - right) > 1) {
      flag = false;
      return 0;
    }
    return Math.max(left, right) + 1;
  }

  dfs(root);
  return flag;
}
```

### 7.将一颗树转化为平衡二叉树

思路：

1. 中序遍历求出有序数组
2. 逐个将二分出来的数组子序列“提”起来变成二叉搜索树

```js
function balanceBST(root) {
  const nums = [];

  function inorder(root) {
    if (!root) return;
    root = inorder(root.left);
    nums.push(root.val);
    root = inorder(root.right);
  }

  function buildAVL(low, high) {
    // 边界条件，如果 low > high 越界，说明当前索引范围对应的子树已经构造完成
    if (low > high) {
      return null;
    }
    // 取有序数组的中间值作为根节点值
    const mid = Math.floor(low + (high - low) / 2);
    // 构建根节点
    const cur = new TreeNode(nums[mid]);
    // 构建左子树
    const left = buildAVL(low, mid - 1);
    // 构建右子树
    const right = buildAVL(mid + 1, high);
    return cur;
  }

  inorder(root);
  return buildAVL(0, nums.length - 1);
}
```

### 8.遍历序列构造树

根据一棵树的前序遍历与中序遍历构造二叉树。

示例：

已知前序遍历 preorder = [3, 9, 20, 15, 7]，中序遍历 inorder = [9, 3, 15, 20, 7]，返回如下的二叉树：

```bash
  3
  / \
9  20
  /  \
  15   7
```

经典递归构造树，把握给定遍历序列的特征。

```js
function buildTree(preorder, inorder) {
  const len = preorder.length;

  function build(preL, preR, inL, inR) {
    // 越界处理
    if (preL > preR) return null;
    // 初始化目标节点
    const root = new TreeNode();
    // 根节点的值就是前序遍历的第一个值
    root.val = preorder[preL];
    // 定位到根节点在中序遍历中的位置
    const k = inorder.indexOf(root.val);
    // 计算左子树的节点数
    const numLeft = k - inL;
    // 构造左子树
    root.left = build(preL + 1, preL + numLeft, inL, k - 1);
    // 构造右子树
    root.right = build(preL + numLeft + 1, preR, k + 1, inR);
    return root;
  }

  build(0, len - 1, 0, len - 1);
}
```

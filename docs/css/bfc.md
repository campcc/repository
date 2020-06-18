---
title: BFC
order: 2
---

# BFC

`BFC`（Block Formatting Contexts），[块级格式化上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)，是 `CSS` 中一个比较晦涩难懂的概念。

下面我们尝试以通俗易懂的语言彻底地理解它。

## 盒模型

`CSS` 盒模型描述了通过 `文档树中的元素` 以及相应的 `视觉格式化模型` 所生成的矩形盒子。

简单来说，盒模型定义了一个 `矩形盒子`，当我们需要对文档进行布局时，浏览器的渲染引擎就会根据盒模型，将所有元素表示为一个个矩形的盒子，盒子的外观由 `CSS` 决定。

一个标准的盒子由四个部分组成，由内向外分别为：`内容`，`内边距`，`边框`，`外边距`：

![boxmodel-_3_.png](https://i.loli.net/2020/03/09/ZwoPDJeMr8XRkBi.png)

**标准的盒模型中，内容区域的大小可以明确地通过 `width`，`min-width`，`max-width`，`height`，`min-height`，`max-height` 控制**，也就是说，通过 `CSS` 设置的**元素宽高只是包含内容区域**。你可能听说过 `怪异盒模型`，这种盒模型最早在 `IE` 浏览器中出现，也叫 `IE盒模型`，`box-sizing` 属性值为 `border-box` 时，元素会呈现`怪异盒模型`，此时，**元素的宽高包含了内容，内边距和边框**。

## 视觉格式化模型

`CSS` 视觉格式化模型描述了**盒子是怎样生成的**，简单来说，它定义了**盒子生成的计算规则**，通过规则将文档元素转换为一个个盒子。

每一个盒子的布局由`尺寸`，`类型`，`定位`，`盒子的子元素或兄弟元素`，`视口的尺寸和位置`等因素决定。

视觉格式化模型的计算，取决于一个矩形的边界，这个矩形边界，就是 `包含块`( containing block ) ，比如：

```html
<table>
  <tr>
    <td></td>
  </tr>
</table>
```

上述代码片段中，`table` 和 `tr` 都是包含块，`table` 是 `tr` 的包含块，同时 `tr` 又是 `td` 的包含块。

需要注意的是，**盒子不受包含块的限制，当盒子的布局跑到包含块的外面时，就是我们说的溢出（overflow）**。

视觉格式化模型定义了盒（Box）的生成，其中的盒主要包括了`块级盒`,`行内盒` 和 `匿名盒`。

### 块级元素

`CSS` 属性值 `display` 为 `block`，`list-item`，`table` 的元素。

### 块级盒

块级盒具有以下特性：

- `CSS` 属性值 `display` 为 `block`，`list-item`，`table` 时，它就是块级元素
- 视觉上，块级盒呈现为竖直排列的块
- 每个块级盒都会参与 `BFC` 的创建
- 每个块级元素都会至少生成一个块级盒，称为主块级盒；一些元素可能会生成额外的块级盒，比如 `<li>`，用来存放项目符号

### 行内级元素

`CSS` 属性值 `display` 为 `inline`，`inline-block`，`inline-table` 的元素。

### 行内盒

行内盒具有以下特性：

- `CSS` 属性值 `display` 为 `inline`，`inline-block`，`inline-table` 时，它就是行内级元素
- 视觉上，行内盒与其他行内级元素排列为多行
- 所有的可替换元素（`display` 值为 `inline`，如 `<img>`， `<iframe>`，`<video>`，`<embed>` 等）生成的盒都是行内盒，它们会参与
  `IFC（行内格式化上下文）` 的创建
- 所有的非可替换行内元素（`display` 值为 `inline-block` 或 `inline-table`）生成的盒称为原子行内级盒，不参与 `IFC` 创建

### 匿名盒

匿名盒指不能被 `CSS` 选择器选中的盒子，比如：

```html
<div>
  匿名盒1
  <p>块盒</p>
  匿名盒2
</div>
```

上述代码片段中，`div` 元素和 `p` 元素都会生成一个块级盒，`p` 元素的前后会生成两个匿名盒。

匿名盒所有可继承的 `CSS` 属性值都为 `inherit`，所有不可继承的 `CSS` 属性值都为 `initial`。

## 定位方案

`CSS` 页面布局技术允许我们拾取网页中的元素，并且控制它们相对正常布局流（普通流）、周边元素、父容器或者主视口/窗口的位置。技术布局从宏观上来说是受定位方案影响，定位方案包括`普通流`（Normal Flow，也叫常规流，正常布局流），`浮动`（Float），`定位技术`（Position）。

### 普通流

浏览器默认的 `HTML` 布局方式，此时浏览器不对页面做任何布局控制，

当 `position` 为 `static` 或 `relative`，并且 `float` 为 `none` 时会触发普通流，普通流有以下特性：

- 普通流中，所有的盒一个接一个排列
- `BFC` 中，盒子会**竖着**排列
- `IFC` 中，盒子会**横着**排列
- 静态定位中（`position` 为 `static`），盒的位置就是普通流里布局的位置
- 相对定位中（`position` 为 `relative`），盒的偏移位置由 `top`，`right`，`bottom`，`left` 定义，
  **即使有偏移，仍然保留原有的位置，其它普通流不能占用这个位置**

### 浮动

- 浮动定位中，盒称为浮动盒（Floating Box）
- 盒位于当前行的开头或结尾
- 普通流会环绕在浮动盒周围，除非设置 `clear` 属性

### 定位技术

定位技术允许我们将一个元素从它在页面的原始位置准确地移动到另外一个位置，有四种：`静态定位`，`相对定位`，`绝对定位`，`固定定位`。

#### 静态定位

默认的定位方式（`position` 为 `static`），此时元素处于`普通流`中。

#### 相对定位

相对定位通常用来对布局进行微调，`position` 为 `relative` 时，元素使用相对定位，此时可以通过 `top`，`right`，`bottom`，`left` 属性对元素的位置进行微调，设置其**相对于自身的偏移量**。

#### 绝对定位

绝对定位方案中，**盒会从普通流中移除**，不会影响其他普通流的布局。绝对定位具有以下特点：

- 元素的属性 `position` 为 `absolute` 或 `fixed` 时，它是绝对定位元素
- 它的定位相对于它的包含块，可以通过 `top`，`right`，`bottom`，`left` 属性对元素的位置进行微调，设置其**相对于包含块的偏移量**
- `position` 为 `absolute` 的元素，其定位将相对于最近的一个 `relative`、`fixed` 或 `absolute` 的父元素，如果没有则相对于 `body`

#### 固定定位

与绝对定位方案类似，唯一的区别在于，它的包含块是**浏览器视窗**。

## 块级格式化上下文

通过对 `CSS` 盒模型，定位，布局等知识的了解，我们知道 `BFC` 这个概念其实来自于`视觉格式化模型`，

它是页面 `CSS` 视觉渲染的一部分，**用于决定块级盒的布局及浮动相互影响范围的一个区域**。

### BFC 的创建

以下元素会创建 `BFC`：

- 根元素（`<html>`）
- 浮动元素（`float` 不为 `none`）
- 绝对定位元素（`position` 为 `absolute` 或 `fixed`）
- 表格的标题和单元格（`display` 为 `table-caption`，`table-cell`）
- 匿名表格单元格元素（`display` 为 `table` 或 `inline-table`）
- 行内块元素（`display` 为 `inline-block`）
- `overflow` 的值不为 `visible` 的元素
- 弹性元素（`display` 为 `flex` 或 `inline-flex` 的元素的直接子元素）
- 网格元素（`display` 为 `grid` 或 `inline-grid` 的元素的直接子元素）

以上是 `CSS2.1` 规范定义的 `BFC` 触发方式，在最新的 `CSS3` 规范中，弹性元素和网格元素会创建 `F(Flex)FC` 和 `G(Grid)FC`。

### BFC 的范围

> A block formatting context contains everything inside of the element creating it, that is not also inside a descendant element that creates a new block formatting context.

直译过来就是，`BFC` 包含创建它的元素的所有子元素，但是不包括创建了新的 `BFC` 的子元素的内部元素。

简单来说，子元素如果又创建了一个新的 `BFC`，那么它里面的内容就不属于上一个 `BFC` 了，这体现了 `BFC` **隔离** 的思想，我们还是以 `table` 为例：

```html
<table>
  <tr>
    <td></td>
  </tr>
</table>
```

假设 `table` 元素创建的 `BFC` 我们记为 `BFC_table`，`tr` 元素创建的 `BFC` 记为 `BFC_tr`，根据规则，两个 `BFC` 的范围分别为：

- `BFC_tr`：`td` 元素
- `BFC_table`：只有 `tr` 元素，不包括 `tr` 里的 `td` 元素

也就是所说，**一个元素不能同时存在于两个 BFC 中**。

### BFC 的特性

`BFC` 除了会创建一个隔离的空间外，还具有以下特性，`附 CodePen 示例链接地址，可结合示例进行理解`：

- `BFC` 内部的块级盒会在垂直方向上一个接一个排列 [①](https://codepen.io/lycheelee/pen/BaNYLNO?editors=1100)
- 同一个 `BFC` 下的相邻块级元素可能发生外边距折叠，创建新的 `BFC` 可以避免的外边距折叠 [②](https://codepen.io/lycheelee/pen/mdJXrwK?editors=1100)
- 每个元素的外边距盒（margin box）的左边与包含块边框盒（border box）的左边相接触（从右向左的格式化，则相反），即使存在浮动也是如此 [③](https://codepen.io/lycheelee/pen/JjdpbGZ?editors=1100)
- 浮动盒的区域不会和 `BFC` 重叠 [④](https://codepen.io/lycheelee/pen/mdJXaXK?editors=1100)
- 计算 `BFC` 的高度时，浮动元素也会参与计算 [⑤](https://codepen.io/lycheelee/pen/wvayENb?editors=1100)

### BFC 的应用

#### 自适应多栏布局

利用 `特性③` 和 `特性④`，中间栏创建 `BFC`，左右栏宽度固定后浮动。由于盒子的 margin box 的左边和包含块 border box 的左边相接触，同时浮动盒的区域不会和 `BFC` 重叠，所以中间栏的宽度会自适应，[示例](https://codepen.io/lycheelee/pen/XWbEjNJ?editors=1100)。

#### 防止外边距折叠

利用 `特性②`，创建新的 `BFC` ，让相邻的块级盒位于不同 `BFC` 下可以防止外边距折叠，[示例](https://codepen.io/lycheelee/pen/eYNMdjJ?editors=1100)。

#### 清除浮动

利用 `特性⑤`，`BFC` 内部的浮动元素也会参与高度计算，可以清除 `BFC` 内部的浮动，[示例](https://codepen.io/lycheelee/pen/ZEGxpgO?editors=1100)。

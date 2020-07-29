---
title: 元信息类标签
order: 2
---

# 元信息类标签

## head

head 标签本身并不携带任何信息，它主要是作为盛放其它语义类标签的容器使用。head 标签规定了自身必须是 html 标签中的第一个标签，它的内容必须包含一个 title，并且最多只能包含一个 base。如果文档作为 iframe，或者有其他方式指定了文档标题时，可以允许不包含 title 标签。

## title

## base

## meta

meta 标签是一组键值对，它是一种通用的元信息表示标签。在 head 中可以出现任意多个 meta 标签。一般的 meta 标签由 name 和 content 两个属性来定义。name 表示元信息的名，content 则用于表示元信息的值

```html
<meta name="" content="" />
```

### charset

```html
<!-- charset 型 meta 标签非常关键，它描述了 HTML 文档自身的编码形式。因此，我建议这个标签放在 head 的第一个 -->
<meta charset="UTF-8" />
```

### http-equiv

### viewport

### 预定义 meta

- author: 页面作者。
- description：页面描述，这个属性可能被用于搜索引擎或者其它场合。
- generator: 生成页面所使用的工具，主要用于可视化编辑器，如果是手写 HTML 的网页，不需要加这个 meta。
- keywords: 页面关键字，对于 SEO 场景非常关键。
- referrer: 跳转策略，是一种安全考量。
- theme-color: 页面风格颜色，实际并不会影响页面，但是浏览器可能据此调整页面之外的 UI（如窗口边框或者 tab 的颜色）。

## meta tag

```html
<!-- 默认使用最新浏览器 -->
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />

<!-- 不被网页(加速)转码 -->
<meta http-equiv="Cache-Control" content="no-siteapp" />

<!-- 搜索引擎抓取 -->
<meta name="robots" content="index,follow" />

<!-- 删除苹果默认的工具栏和菜单栏 -->
<meta name="renderer" content="webkit" />
<meta
  name="viewport"
  content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no, minimal-ui"
/>
<meta name="apple-mobile-web-app-capable" content="yes" />

<!-- 设置苹果工具栏颜色 -->
<meta
  name="apple-mobile-web-app-status-bar-style"
  content="black-translucent"
/>
```

## 写在最后

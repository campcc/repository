---
title: 语义类标签
order: 1
nav:
  title: HTML
  order: 8
---

# 语义类标签

典型的“入门容易，精通困难”的一部分知识

## Web 语义化

Web 语义化广义上讲，就是让机器可以读懂内容。

![image.png](https://i.loli.net/2020/09/10/T9aQfv7FxC24ZPi.png)

(图片说明：内容的语义表达能力和 AI 的智能程度决定了机器分析处理 Web 内容能力的高低)

于开发者而言，语义化期望我们能够做到 “词必达意”，简单来说就是 **用正确的语义标签来描述内容和结构**。

## header & hgroup

之所以把它俩放到一起，是因为 header 和 hgroup 都具有页面，文章的 “标题” 的语义，

header 更多地用于表示网页或文章的页眉，通常包含一组介绍性的或是辅助导航的实用元素。

这可能包含标题元素、Logo、搜索框、作者名称等，比如最常见的用法，

- 作为页面的标题

```html
<body>
  <header>
    <h1>Page Title</h1>
    <img alt="logo" src="xxx.png" />
  </header>
</body>
```

- 作为文章的标题

```html
<article>
  <header>
    <h1>Article Title</h1>
    <p>...</p>
  </header>
</article>
```

hgroup 标签正如其名，表示一组标题元素，它的子元素为 h1 ~ h6 的组合。

hgroup 更多用来描述文档章节所属的多级别的目录，与 header 类似，可以作为页面，文章，section 的标题，

```html
<article>
  <hgroup>
    <h1>Article Title</h1>
    <h2>Sub Title</h2>
  </hgroup>
</article>
```

## footer

footer 表示最近一个章节内容或者根节点元素的页脚，通常包含该章节作者、版权数据或者与文档相关的链接等信息。

```html
<article>
  <footer>
    <p>©copyright 1996-2020 campcc</p>
    <p>author: steven丶lee</p>
  </footer>
</article>
```

## aside

aside 表示可以独立拆分出来的，与文章主体不那么相关的部分，通常表现为侧边栏或者标注框，

需要注意的一点是，aside 和 header 中都可能出现导航（nav 标签），二者的区别是，

header 中的导航多数是到文章自己的目录，而 aside 中的导航多数是到关联页面或者是整站地图。

```html
<aside>
  <nav>...</nav>
  <p>The Rough-skinned Newt defends itself with a deadly neurotoxin.</p>
</aside>
```

## section

你可以把 section 标签理解为：“一个有语义的 div”，文档中一个独立的部分。

一个 section 一般会包含一个标题，利用 section 我们可以对文档内容做一定的结构性划分。

需要注意的是，section 会弱化 h1 ~ h6 的语义，section 的嵌套会使得其中的 h1 ~ h6 下降一级。

## main

一般作为 body 的直接子元素，表示文档或应用的主体部分，

```html
<body>
  <main role="main">
    <p>
      Geckos are a group of usually small, usually nocturnal lizards. They are
      found on every continent except Australia.
    </p>
    <p>
      Many species of gecko have adhesive toe pads which enable them to climb
      walls and even windows.
    </p>
  </main>
</body>
```

## article

article 是一种特别的结构，它表示具有一定独立性质的文章。

所以，article 和 body 具有相似的结构，同时，一个 HTML 页面中，可能有多个 article 存在。

一个典型的场景是多篇新闻展示在同一个新闻专题页面中，这种类似报纸的多文章结构适合用 article 来组织,

```html
<body>
  <!-- article 1 -->
  <article>
    <header>xxx</header>
    <section>xxx</section>
    <section>xxx</section>
    <section>...</section>
  </article>
  <!-- article 2 -->
  <article>...</article>
  <!-- article 3 -->
  <article>...</article>
</body>
```

## nav

nav 为当前文档或其他文档中提供导航链接，常见的表现为，菜单，目录和索引。比如下面的面包屑导航，

```html
<nav class="crumbs">
  <ol>
    <li class="crumb"><a href="bikes">Bikes</a></li>
    <li class="crumb"><a href="bikes/bmx">BMX</a></li>
    <li class="crumb">Jump Bike 3000</li>
  </ol>
</nav>
```

## p

p 表示文本的一个段落，p 具有垂直的空白隔离或首行缩进，

需要注意的是，应该使用 CSS margin 属性去改变段落之间的间隙，而不是在段落之间插入空的段落元素或者 `<br>` 元素。

## ul & ol & li

ul 和 ol 分别表示无序列表和有序列表，li 作为列表的条目元素，在 ul 和 ol 里有不同的表现，

无序列表里，列表条目通常用点排列显示；在有序列表里，列表条目通常在左边显示按升序排列的计数，例如数字或者字母。

示例，ol 通常渲染为一个带编号的列表，

```html
<ol>
  <li>Mix flour, baking powder, sugar, and salt.</li>
  <li>In another bowl, mix eggs, milk, and oil.</li>
  <li>Stir both mixtures together.</li>
  <li>Fill muffin tray 3/4 full.</li>
  <li>Bake for 20 minutes.</li>
</ol>
```

## em & i

em 表示需要用户着重阅读的内容，合理地使用 em 标签能够避免文字产生歧义。

em 和 i 标签在默认情况下的视觉效果是一样，但是它们具有完全不同的语义，

`<em>` 标签表示其内容的着重强调，而 `<i>` 标签表示从正常散文中区分出的文本，例如外来词，虚构人物的思想等。

PS：作品的标题，例如书籍或电影的名字，应该使用 `<cite>`。

## strong

strong 表示文本十分重要，一般用粗体显示。

很多文章常常会拿 em 和 strong 做对比，其实它俩可谓天差地别，并没有任何混淆的可能。

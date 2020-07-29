---
title: 语义类标签
order: 1
nav:
  title: HTML
  order: 2
---

# 语义类标签

典型的“入门容易，精通困难”的一部分知识

## header & footer

最后 footer 中包含 address，address 明确地只关联到 article 和 body。

## section

介绍 `section` 之前，我们先了解下什么是 `hgroup` 标签

```html
<hgroup>
  <h1>World Wide Web</h1>
  <h2>From Wikipedia, the free encyclopedia</h2>
</hgroup>
```

从 HTML 5 开始，我们有了 section 标签，这个标签可不仅仅是一个“有语义的 div”，它会改变 h1-h6 的语义。section 的嵌套会使得其中的 h1-h6 下降一级，因此，在 HTML5 以后，我们只需要 section 和 h1 就足以形成文档的树形结构

## article

article 是一种特别的结构，它表示具有一定独立性质的文章。所以，article 和 body 具有相似的结构，同时，一个 HTML 页面中，可能有多个 article 存在。一个典型的场景是多篇新闻展示在同一个新闻专题页面中，这种类似报纸的多文章结构适合用 article 来组织。

### abbr

### hr

hr 表示故事走向的转变或者话题的转变

### p

### strong & em

### blockquote, q & cite

如果是直接引用的内容，那么，我们还应该加上 blockquote 或者 q 标签

### time

### figure & figcaption

figure 用于表示与主文章相关的图像、照片等流内容

这种插入文章中的内容，不仅限图片，代码、表格等，只要是具有一定自包含性（类似独立句子）的内容，都可以用 figure。这里面，我们用 figcaption 表示内容的标题，当然，也可以没有标题。

### dfn

你需要在你要定义的词前后放上 dfn 标签，dfn 标签是用来包裹被定义的名词

### nav

### ol & ul

### pre, samp, code

```html
<pre>
  <samp>
GET /home.html HTTP/1.1
Host: www.example.org
  </samp>
</pre>
```

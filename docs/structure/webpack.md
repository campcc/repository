---
title: 解析规则
order: 1
nav:
  title: Webpack
  order: 4
---

# 解析规则

前端模块化中，我们经常使用相对路径和第三方类库名的方式来引入模块：

```js
import foo from './foo.js'; // 相对路径
import bar from '@/src/bar.js'; // 相对路径，alias也是相对路径
import react from 'react'; // 第三方类库名称
```

当然，还可以通过绝对路径引入模块，但是这种方式不推荐也不常用，我们暂不讨论。

webpack 中有一个专门的模块 [enhanced-resolve](https://github.com/webpack/enhanced-resolve/) 来处理依赖模块路径的解析，从这个模块名称我们大致可以猜到是基于 Node.js 的，没错，其实就是 Node.js 模块解析的增强版，支持更多的自定义配置。

webpack 的模块解析规则分为三种，**相对路径**，**绝对路径**和**模块名**。

## 绝对路径

我们先从最简单但是不常用的绝对路径说起，绝对路径通常是从盘符开始，像下面这样：

```bash
C:\hello-world\index.js
```

webpack 处理绝对路径的解析规则就是直接查找对应路径下的模块。

## 相对路径

相对路径的解析规则略微复杂，大致分为下面几个步骤：

1. 首先查找当前模块的路径下是否有匹配的文件或文件夹
2. 如果文件名匹配直接加载模块
3. 如果是文件夹，查找文件夹下面的 package.json 文件
4. 如果有 package.json 文件，查找 main 字段指定的文件
5. 如果没有 package.json 文件或者没有 main 字段，查找 index.js 文件

## 模块名

如果是模块名称，webpack 会逐次查找当前目录下，父级目录及以上的目录下的 node_modules 文件夹，如果找到 node_modules 文件夹，会去 node_modules 文件夹中查找指定的模块。

---
title: resolve
order: 2
---

# resolve

webpack 配置中，模块路径解析的相关配置都在 `resolve` 字段中，`resolve` 到底有什么魔法?

## alias

你可能经常在一些代码中看到如下的片段：

```js
import { color } from '@/constants';
```

这里模块路径中的 `@` 就是别名，webpack 支持通过 **alias** 属性配置我们需要的别名，假设我们有个 `src` 模块经常用，可以通过配置模块别名来使模块的引入变得简单：

```js
module.exports = {
  //...
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
};
```

上面的配置表示模糊匹配，只要路径中有 `@` 都会被替换掉，比如：

```js
import App from '@/index.js'; // 相当于 import App from '[项目绝对路径]/src/index.js'
import { reduce } from '@/utils/array.js'; // 相当于 import { reduce } from '[项目绝对路径]/src/utils/array.js'
```

如果想要精确匹配，可以在别名末尾加上 `$`：

```js
alias: {
  '@$': path.resolve(__dirname, 'src') // 只会匹配 import '@'，不会匹配 import '@/xxx'
}
```

## extensions

webpack 支持通过 extensions 配置路径解析模块的扩展名，配置后引入模块可以不带扩展，默认为：

```js
extensions: ['.wasm', '.mjs', '.js', '.json'];
```

上述配置表示，路径解析时，如果路径不包含扩展名 webpack 会尝试给路径添加 `extensions` 中的扩展名，然后再进行依赖模块的查找。比如，引入 `src` 目录下的 `index.js` 文件就可以写成：

```js
import App from 'src/index';
```

需要注意的是，如果配置了 extensions 会覆盖 webpack 原有的默认配置。假设 extension 配置是：

```js
extentsions: ['.css'];
```

这时再通过不带扩展名的方式引入 `js` 文件，就会报错：

```js
import App from 'src/index'; // Module not found，因为 extension 中找不到扩展名对应的模块
```

想要解决这个问题，可以将扩展名重新添加到 extensions 数组，

```js
extentsions: ['.js', '.css'];
```

## modules

modules 指定了 webpack 解析模块时需要搜索的目录，默认为 `node_modules`：

```js
modules: [`node_modules`];
```

上述的配置表示，webpack 在解析模块名时，以引入 `react` 为例：

```js
import React from 'react';
```

webpack 会逐级向上搜索 `node_modules` 目录，直到找到名称为 `react` 的模块。

## mainFiles

mainFiles 指定了解析目录时使用的文件名，默认为 `index`。

```js
mainFiles: ['index'];
```

上面的配置表示，如果 webpack 路径解析遇到目录时，默认查找目录下的 `index.js` 文件。

## mainFields

mainFields 指定了当路径解析，引用的是一个目录或模块的时候，应该使用 `package.json` 中的哪个字段指定的文件，默认的配置是：

```js
mainFields: ['browser', 'module', 'main'], // target 为 web 或 webworker 时

mainFields: ['module', 'main'], // target 为其他时，比如(node)
```

以 [D3](https://github.com/d3/d3/blob/master/package.json) 的 `package.json` 为例：

```json
{
  "main": "dist/d3.node.js",
  "module": "index.js"
}
```

上述的配置表示，当我们从 `npm` 包导入 `D3` 模块时，默认查找的是 `main` 字段指定的文件：

```js
import * as D3 from 'd3';
```

按照默认的配置，如果有 `browser` 字段，会优先解析 `browser` 字段指定的文件，其次是 `module`, 再其次是 `main`。而由 `webpack` 打包的 `Node.js` 应用程序默认会从 `module` 字段中解析文件。

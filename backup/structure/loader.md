---
title: Loader
order: 3
---

# Loader

Loader（模块转换器），本质就是一个函数，它只完成一个功能：**对接收到的内容（source）进行转换，返回转换后的结果**。

什么是模块（module）？

Webpack 中，除了 js 范畴内的 es module，commonjs，AMD 等，css 的 @import，url(...)，图片，字体等都被视为模块。

由于 Webpack 只认识 JavaScript，所以 Loader 就相当于翻译官，通过 Loader，我们可以对其他资源进行转译的预处理。

## 使用方法

每个 Loader 必须确保职责单一，就像流水线的工人一样，

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
};
```

多个 Loader 时，调用顺序为从右向左。

## 实现准则

- 【Simple】，Loader 只做单一任务
- 【Chaining】，遵循链式调用原则
- 【Stateless】，即函数式里的 Pure Function，Loader 应该是无副作用的

## 简单 Loader

接下来，我们使用 [loader-utils]() 实现一个简单的 Loader，替换 console.log，去除换行符然后在文件结尾加上一行自定义的内容，

```js
/**
 * @File: 实现一个简单 Loader, 替换 console.log, 去除换行符，在文件结尾增加一行自定义内容
 */
'use strict';

const loaderUtils = require('loader-utils');

module.exports = function replace(source) {
  // 获取 Loader 配置项
  const options = loaderUtils.getOptions(this);

  console.log('loader配置项', options);

  const result = source
    .replace(/console.log\(.*\);?/g, '')
    .replace(/\n/g, '')
    .concat(`console.log("${options.messages || 'No Options'}")`);

  return result;
};
```

## 异步 Loader

```js
/**
 * @File: 实现一个简单 Loader, 替换 console.log, 去除换行符，在文件结尾增加一行自定义内容
 */
'use strict';

module.exports = function async(source) {
  let count = 1;

  // 调用 this.async() 可以告诉 Webpack，这是一个异步的 Loader，需要等待 callback 回调后再执行下一个 Loader
  // this.async 返回一个异步回调，调用这个返回的异步回调即表示异步 Loader 处理结束
  const callback = this.async();

  const timer = setInterval(
    () => console.log(`Time has passed ${count} s`),
    1000,
  );

  // 异步操作
  setTimeout(() => {
    clearInterval(timer);
    callback(null, source);
  }, 3000);
};
```

---
title: Plugin
order: 4
---

# Plugin

Webpack 插件基于事件流框架 [Tapable](https://github.com/webpack/tapable)，插件可以扩展 Webpack 的功能。

我们知道在 Webpack 运行的生命周期中会广播出许多事件，Plugin 允许我们监听这些事件，在合适的时机配合 Webpack 提供的 API 以改变输出结果。

## 原理

基于 Tapable 在 Webpack 编译的整个生命周期的特点节点执行特定的功能。

## Tapable & Compiler & Compilation

Tapable 是一个基于发布订阅的事件流工具类，Webpack 中的 Compiler 和 Compilation 对象都继承至 Tapable。

Compiler 负责编译，是 Webpack 在编译初始化阶段创建的一个全局单例，包含完整的配置信息，loaders，plugins 以及各种工具方法。

Compilation 代表一次 Webpack 构建和生成编译资源的过程，在 watch 模式下，每一次文件改变触发的重新编译都会生成一个新的 Compilation 对象，Compilation 包含了当前编译的模块 module，编译生成的资源，变化的文件，依赖的状态等。

## 手写 Plugin

### 实现思路

- 一个命名的 JavaScript 函数或 类
- 原型对象 prototype 上定义一个 apply 方法，供 Webpack 调用，并在调用时注入 Compiler 对象
- apply 函数中，通过 Compiler 对象挂载 Webpack 的事件钩子，在钩子函数中拿到当前编译的 Compilation 对象
- 处理 Webpack 内部实例的特定数据
- 功能完成后调用 Webpack 提供的回调

### 基本模型

```js
const PLUGIN_NAME = 'Plugin';

class Plugin {
  constructor(option) {
    this.option = option; // 通过构造函数获得插件的配置项
  }

  // 在原型对象上定义一个 apply 函数供 Webpack 调用
  apply(compiler) {
    // 注册 Webpack 事件监听函数
    compiler.hooks.emit.tapAsync(PLUGIN_NAME, (compilation, asyncCallback) => {
      // 操作或改变 Compilation 内部数据
      console.log(compilation);

      console.log('Compile completed, ready to output');

      // 如果是异步钩子，结束后需要执行异步回调
      asyncCallback();
    });
  }
}

module.exports = Plugin;
```

### 实现一个简单的 Plugin

基于基本模型，我们实现一个简单的插件，功能是在 dist 目录自动生成 README 文件，

```js
/**
 * @File: 实现一个简单 Loader, 替换 console.log, 去除换行符，在文件结尾增加一行自定义内容
 */
'use strict';

const PLUGIN_NAME = 'ReadmePlugin';

class ReadmePlugin {
  constructor(option = {}) {
    this.option = option;
  }

  apply(compiler) {
    compiler.hooks.emit.tapAsync(PLUGIN_NAME, (compilation, asyncCallback) => {
      const { title } = this.options;
      compilation.assets['README.md'] = {
        source: () => `# ${title || 'Default Title'}`, // README 文件的内容
        size: () => 30, // 指定文件大小
      };
      asyncCallback();
    });
  }
}

module.exports = ReadmePlugin;
```

compiler.hooks 上挂载了不同时期触发的 Webpack 事件函数（类似于 React 的生命周期），可以在编译的各个阶段执行其他逻辑或者改变输出结果。

完整的事件列表可以看这里，[compiler-hooks](https://webpack.js.org/api/compiler-hooks/)。

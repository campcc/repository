---
title: 构建工具调优
order: 2
---

## 从构建工具看 HTTP 优化

从输入 URL 到显示页面这个过程中，涉及到网络层面的，有三个主要过程：

- DNS 解析
- TCP 连接
- HTTP 请求/响应

对于 DNS 解析和 TCP 连接两个步骤(如果忘了，可以回顾下我们在计算机基础中的章节 [TCP](https://campcc.github.io/repository/basics/osi/tcp)，[DNS](https://campcc.github.io/repository/basics/osi/dns))，前端可以做的努力非常有限。相比之下，HTTP 连接这一层面的优化才是我们网络优化的核心。

我们直奔主题，网络层面 HTTP 优化有两个大的方向：

1. 减少请求次数
2. 减少单次请求所花费的时间

这两个优化点最先想到的就是对应我们日常开发中最常见的 **构建工具**，因为它所做的事情本质就是--**资源的压缩和合并**。

所以接下来我们从时下最主流的 Webpack 说起，从性能调优切入一窥究竟。

## 构建提速

**不要让 loader 做太多事情**

构建提速的一个优化点就是，不要让 loader 做太多的事情，以 babel-loader 为例，它无疑是强大的，但也是慢的，最常见的优化方式就是，用 include 和 exclude 来帮助我们避免不必要的转译，比如 webpack 官方在介绍 babel-loader 时给出的示例：

```js
module: {
  rules: [
    {
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env'],
        },
      },
    },
  ];
}
```

此外，开启构建缓存，将转译结果缓存至文件系统，也可以提高 babel-loader 的效率，

```js
loader: 'babel-loader?cacheDirectory=true';
```

## 第三方库处理

第三方库以 node_modules 为代表，它们庞大得可怕，却又不可或缺。

### 作为外部依赖的 external

external 相信大家做过组件库开发的都不陌生，它可以将依赖作为外部依赖打包，从而减少库的体积。

### 提取依赖的 DllPlugin

DllPlugin 是基于 Windows 动态链接库（dll）的思想，会把第三方库单独打包到一个文件中，而这个文件就是一个单纯的依赖库，它有什么特别之处呢？

**这个依赖库不会跟着你的业务代码一起被重新打包，只有当依赖自身发生版本变化时才会重新打包。**

DllPlugin 处理文件的姿势一般分两步，

1. 基于 dll 配置文件，打包 dll 库
2. 基于 webpack 配置文件，打包业务代码

以简单的 React 项目为例，dll 配置文件,

```js
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    // 依赖的库数组
    vendor: [
      'prop-types',
      'babel-polyfill',
      'react',
      'react-dom',
      'react-router-dom',
    ],
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].js',
    library: '[name]_[hash]',
  },
  plugins: [
    new webpack.DllPlugin({
      // DllPlugin的name属性需要和libary保持一致
      name: '[name]_[hash]',
      path: path.join(__dirname, 'dist', '[name]-manifest.json'),
      // context需要和webpack.config.js保持一致
      context: __dirname,
    }),
  ],
};
```

运行上述配置文件，dist 文件夹会为我们生成,

```bash
vendor-manifest.json // 描述每个第三方库对应的具体路径
vendor.js // 第三方库打包的结果
```

接着，我们只需要在 webpack 配置中引入 dll 描述文件即可完成基于 dll 的 webpack 构建过程优化，

```js
const path = require('path');
const webpack = require('webpack');
module.exports = {
  mode: 'production',
  // 编译入口
  entry: {
    main: './src/index.js',
  },
  // 目标文件
  output: {
    path: path.join(__dirname, 'dist/'),
    filename: '[name].js',
  },
  // dll相关配置
  plugins: [
    new webpack.DllReferencePlugin({
      context: __dirname,
      // manifest就是我们第一步中打包出来的json文件
      manifest: require('./dist/vendor-manifest.json'),
    }),
  ],
};
```

## 多线程构建

webpack 是单线程的，这意味着就算此刻存在多个任务，也只能排队一个接一个地等待处理。

为了充分利用 CPU 在多核并发方面的优势，我们可以用 Happypack 将 loader 由单进程转为多进程。

下面是一个简单的例子，

```js
const HappyPack = require('happypack')
// 手动创建进程池
const happyThreadPool =  HappyPack.ThreadPool({ size: os.cpus().length })

module.exports = {
  module: {
    rules: [
      ...
      {
        test: /\.js$/,
        // 问号后面的查询参数指定了处理这类文件的HappyPack实例的名字
        loader: 'happypack/loader?id=happyBabel',
        ...
      },
    ],
  },
  plugins: [
    ...
    new HappyPack({
      // 这个HappyPack的“名字”就叫做happyBabel，和楼上的查询参数遥相呼应
      id: 'happyBabel',
      // 指定进程池
      threadPool: happyThreadPool,
      loaders: ['babel-loader?cacheDirectory']
    })
  ],
}
```

## Tree Shaking

Tree-Shaking 的初衷本质是为了构建体积压缩，我们知道，对于构建结果，可以用 [webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer) 将文件结构可视化，找出导致体积过大的原因。

找到原因后要做的无非就是两件事，

1. 拆分资源，这个可以交给 DllPlugin 去做
2. 删除冗余代码

其中删除冗余代码的一个典型应用，就是 Tree-Shaking。那么，它到底是个什么东西？

从 webpack2 开始，webpack 原生支持了 ES6 的模块系统，并基于此推出了 Tree-Shaking。

我们知道 ES6 的模块系统最大的一个特点就是静态分析，基于 import/export 语法，可以在编译的过程中就获悉哪些模块并没有真正被使用，而这些没用的代码，在最后打包的时候会被去除。

没错，这就是 Tree-Shaking。它更适用处理模块级别的冗余代码。至于粒度更细的冗余代码的去除，往往会被整合进 JS 或 CSS 的压缩或分离过程中。

以 UglifyJsPlugin 为例，在早期的 webpack(@3-) 版本中，你可能会看到以下配置，

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
module.exports = {
  plugins: [
    new UglifyJsPlugin({
      // 允许并发
      parallel: true,
      // 开启缓存
      cache: true,
      compress: {
        // 删除所有的console语句
        drop_console: true,
        // 把使用多次的静态值自动定义为变量
        reduce_vars: true,
      },
      output: {
        // 不保留注释
        comment: false,
        // 使输出的代码尽可能紧凑
        beautify: false,
      },
    }),
  ],
};
```

但是我们知道 webpack4 现在已经默认使用 uglifyjs-webpack-plugin 对代码做压缩了。

在 webpack4 中，现在我们是通过配置 `optimization.minimize` 与 `optimization.minimizer` 来自定义压缩相关的操作的。

其实学习工具的使用并不复杂，相反，只有了解了背后的原理，历史(为什么会演变成这样)，才能做到举一反三，因为工具都存在用法迭代的问题，但核心的动机，原理不会变。

## 按需加载

什么是按需加载？

我们先来看一个场景，

假设我们现在用 React 构建了一个单页应用，用 React-Router 来控制路由，十个路由对应了十个页面，每一个页面都很复杂，这个时候如果我们将整个项目打成一个包，用户访问我们的系统时，会出现什么情况？

有很大机率会卡死，对不对？更好的做法是什么？

是不是应该先给用户展示一个页面(大概率可能是首页)，然后其他页面等请求到了再去加载。当然这是一个很极端的场景，但是它很好的诠释了什么是按需加载，以及为什么我们需要做按需加载。

注意：多页应用针对不同页面本来就是按需加载的，这里我们讨论的是 SPA 单页搭配路由的场景。

下面我们来看如何通过 webpack 来做组件的按需加载，

首先为了开启按需加载，我们需要在 webpack 配置文件中指定 chunkFilename，

```js
output: {
	path: path.join(__dirname, '/../dist'),
	filename: 'app.js',
	publicPath: defaultSettings.publicPath,
	// 指定 chunkFilename
	chunkFilename: '[name].[chunkhash:5].chunk.js',
},
```

然后修改路由处的代码进行简单的配合，

```js
const getComponent => (location, cb) {
  require.ensure([], (require) => {
    cb(null, require('../pages/ComponentA').default)
  }, 'a')
},
...
<Route path="/bug" getComponent={getComponent}>
```

这里核心的方法就是 `require.ensure`,

```js
require.ensure(dependencies, callback, chunkName);
```

这是一个异步的方法，webpack 在打包时，ComponentA 会被单独打成一个文件，只有在我们跳转 `a` 这个路由的时候，这个异步方法的回调才会生效，才会真正地去获取 ComponentA 内容，从而实现按需加载。

类似的，按需加载的粒度，还可以继续细化，细化到更小的组件、细化到某个功能点，都是 ok 的。

这是 React-Router 3 的用法，在 React-Router 4 中，我们用 `Bundle-Loader`，也就是 Code-Splitting 替换掉了上述做法。甚至，我们还可以借助一些现有的库，比如 `react-loadable` 来简化这个过程。但是，就像我们在前面介绍 Tree-Shaking 时所说，工具是会迭代的，我们要抓出的是核心的东西而不是去纠结工具的用法。

其实去看 Bundle Loader 的源码，你会发现，它本质上还是通过 `require.ensure` 来实现的，所以，所谓按需加载，根本上就是在正确的时机去触发相应的回调。

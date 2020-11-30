---
title: 培训
order: 1
nav:
  title: 培训
  order: 9
---

# 技术培训

石钢工艺质量管理系统前端包含两个项目，Web 端和 App 端，下面主要针对 Web 端做一些简单的介绍。

## 技术架构

Web 端基于 React 全家桶（React + Redux + Redux Middleware + React Router），ES6/7，CSS3

### 构建工具

**Webpack + Creat React App**

![image.png](https://i.loli.net/2020/11/30/C5XrsVERgWkND7y.png)![image.png](https://i.loli.net/2020/11/30/CJUjhTnAEa49Gwe.png)

脚手架基于 React 官方推荐脚手架工具 CRA，在 CRA 基础上，我们修改并集成了部分自定义 Webpack 配置，主要包括，

- 蚂蚁组件库的主题配色支持
- 自定义 Eslint 配置，Prettier 配置支持
- 博智前端 Webpack 构建相关配置集成
- ES6/7 语法支持
- dayjs 时间库支持
- ... ...

### 开发语言

**React 16.11.0 + ECMAScript 6/7**

#### React

![image.png](https://i.loli.net/2020/11/30/Dl7ut5YTyhZjx2C.png)

React 是一个声明式，组件化，高性能的用于构建用户界面的 JavaScript 库。

我们认为作为一个 React 开发者，你至少需要具备以下条件，

- 了解/熟悉 JavaScript，ES5
- 对 MVVM 有一定了解
- 熟悉 JSX 模板语法
- 熟悉 React 组件的生命周期，了解渲染流程
- 熟悉 React 组件间简单的通信（父子，子父，兄弟）
- 熟悉 React 事件原理，常用的绑定规则
- 熟练运用 React 本地状态
- ... ...

#### ES6

![image.png](https://i.loli.net/2020/11/30/6hlCMpt8OF5fvrQ.png)

石钢项目默认支持 ES6/7/8 语法，你至少需要了解以下的 ES6/7 特性，

- 类，继承
- 解构
- 模板字符串
- 箭头函数
- Promise
- async/await 异步语法
- 数组，对象等常用数据结构的扩展

### UI 框架

**Ant Design + 博智云创自研组件库 bici-transformers**

![image.png](https://i.loli.net/2020/11/30/rBXxR75vi3utemD.png)

#### Ant Design

Ant Design 是基于蚂蚁设计体系的 React UI 组件库，主要用于研发企业级中后台产品。

石钢项目使用的 Ant Design 版本为 3.24.2(3.x 的最新版本)。

#### bici-transformers

除了蚂蚁金服的基础 UI 库外，石钢项目还使用了基于博智云创前端设计规范的自研组件库 bici-transformers，包括，

- 基础请求库
- 支持复杂数据展示查询的复杂表格
- 实时通信方案的 WebSocket 连接库
- ... ...

### 路由方案

**React Router**

React Router 是一组导航组件的集合，可与 React 应用进行声明式的组合。石钢项目的前端路由基于 React Router 5.1.2 版本，我们在页面导航的基础上，支持路由嵌套，二级，三级路由动态渲染，路由面包屑，权限路由等特性。

### 状态管理

**本地状态 + Redux**

![image.png](https://i.loli.net/2020/11/30/md2LJouHj96aKp7.png)

Redux 是 JavaScript 状态容器，提供可预测化的状态管理。

石钢项目中，我们对于数据状态采用了本地状态搭配 Redux 的方案。其中 Redux 主要用于存储一些全局状态（如消息，通知，用户信息，登录凭证等），以及做时光旅行。对于模块间，组件间的状态建议维护在本地。我们通过页面维度进行模块拆分，对于同一个模块而言，一般涉及到的状态嵌套层次不会超过 2 ~ 3 层，所以优先考虑使用 React 本地状态进行数据组织。如果对于特殊较为复杂的模块，才考虑通过 Redux 进行维护。

我们认为开发前至少需要具备对于 Redux 以下的特性的掌握，

- 了解纯函数，Reducer，Action 等基本概念
- 熟悉 Redux 状态流
- 熟悉单一 Store 的数据流

### 中间件

Redux-thunk，Redux-logger，Redux-persist

- Redux-thunk，支持返回函数的 Action Creator
- Redux-logger，本地开发日志打印
- Redux-persist，状态持久化

### Module CSS

CSS 模块化的方案有很多，石钢项目中我们使用的是 Module CSS，同时以页面维度进行 CSS 模块拆分。你需要熟练 CSS2.1/CSS3 的常用语法。

在模块 CSS 基础上，推荐结合属性 CSS（本项目默认支持）以提高样式编码效率。

### 拖拽方案

react-dnd + interactjs

### 图表库

BizCharts + Echarts + AntV G6

- Bizcharts，阿里通用图表组件库，致力于打造企业中后台高效、专业、便捷的数据可视化解决方案，基于 G2 与 G2Plot，基础图表库
- Echarts，一个兼容性较强的使用 JavaScript 实现的开源可视化库，本项目中主要用于支撑历史曲线等大数据量展示
- AntV G6，蚂蚁可视化架构下，一个简单，完备的图可视化引擎，工艺质量流程图的基础图表库

### 工具库

Lodash，Moment（dayjs），Axios

## 目录结构

![image.png](https://i.loli.net/2020/11/30/c7A2DUnwKjqkWYp.png)

## 开发流程

石钢项目使用 Git 作为版本管理工具，多人协同上我们推荐使用 Git Flow 工作流，支持热更新，自动构建和基于 DaoCloud 的持续集成。

### 依赖环境

- Node.js 11.6.0+
- npm 5.0+

### 推荐使用 nrm 和 nvm

推荐使用 [nrm](https://github.com/Pana/nrm) 管理 npm 源，支持命令一键快捷查看，添加和切换源

```bash
# 查看所有源
nrm ls

# 一键切换源
npm use taobao
```

推荐使用 [nvm](https://github.com/nvm-sh/nvm) 管理 Node 版本

```bash
# 查看可用的 Node 版本
nvm ls

# 一键切换 Node 版本
nvm use 11.6.0
```

### 推荐使用 npm 或 yarn 安装

推荐使用 npm 或 yarn 的方式进行开发，不仅可在开发环境轻松调试，也可放心地在生产环境打包部署使用，享受整个生态圈和工具链带来的诸多好处。

```bash
$ npm install bicid --save
```

```bash
$ yarn add bicid
```

如果你的网络环境不佳，推荐使用 [cnpm](https://github.com/cnpm/cnpm)。

## 构建发布

云端环境我们已经集成了 DaoCloud，这里主要针对非云端环境的本地构建发布流程做一定说明。

本地构建发布的流程较为简单，首先生成静态包，

1. 修改配置文件 `src/config/index.js`，将环境变量设置为需要构建发布的环境（release, staging, production）

```js
const APP_NODE_DEV = 'dev'; // dev，staging，pro 分别指代 release, staging, production 三个环境
```

2. 执行打包命令，构建静态包，

```bash
# 构建
yarn build
# 或者
npm run build
```

3. 验证本地包，可以使用 `serve` 静态先部署一个版本

```bash
# 本地静态部署，验证
serve build
```

4. 上传静态包到服务器，完成发布

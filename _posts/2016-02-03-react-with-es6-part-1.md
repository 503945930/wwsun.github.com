---
layout: post
title: 使用ES6编写React应用（1）：简介及环境配置
category: knowledge
---

本文将会介绍如何使用ES6来编写React应用，作为本系列的第一篇文章，本文将主要讲诉相关的开发环境配置。

<!--more-->

## Statement

- **作者：** [景庄](http://wwsun.github.com)，Web开发者，主要关注JavaScript、Node.js、React、Docker等。

本文所使用的各项技术均为最新版本，具体的版本信息参考如下：

- Babel v6
- Node v4
- Koa v1
- React v0.14 (`react` and `react-dom`)

## 项目结构

本文所使用的项目结构如下：

    + build     客户端代码的构建结果目录
    + config    配置信息目录
    + lib       服务端库文件
        - render.js 渲染脚本
    + src       客户端源代码存放目录
    + test      测试文件目录
    + views     视图文件目录
    - index.js  服务器脚本

## 准备开发环境

### 构建一个简单的Koa服务器

为了能够让我们所构建的应用正常的运行和调试，首先需要做的是构建一个基本的Web容器，幸运的是，
我们可以基于Node.js快速的构建一个简单的并且满足需求的Web服务器。为了简单起见，我们选择使用Koa进行构建。

1. 使用`npm init --yes`快速的生成项目的初始文件。
2. 安装服务器端的相关依赖，如下：
    - koa
    - koa-logger
    - koa-route
    - koa-static
    - co-views
    - swig
3. 由于我们也想用ES6开发服务端代码，因此我们打算使用Babel动态编译代码，安装相关依赖：
    - babel-cli
    - babel-preset-es2015-node5
4. 在`package.json`中增加`babel`和`scripts`配置项：
        "babel": {
            "presets": [
            "es2015-node5"
            ]
        },
        "scripts": {
            "start": "babel-node index.js"
        }

服务器端代码的相关依赖和配置信息都已经完成了，现在只需要用ES6来开发一个简单的koa服务器即可：

```javascript
// index.js
'use strict';

import path from 'path';
import koa from 'koa';
import logger from 'koa-logger';
import serve from 'koa-static';
import route from 'koa-route';

import render from './lib/render';

var app = koa();

app.use(logger());
app.use(route.get('/', home));

function *home() {
  this.body = yield render('home', {});
}

app.use(serve(path.join(__dirname, 'build')));

app.listen(3000);
console.log('listening on port 3000');
```

由于服务器需要渲染基本的视图文件，因此我们可以借助`co-views`和`swig`编写一个简单的服务器端渲染脚本：

```javascript
// lib/render.js
'use strict';

import path from 'path';
import views from 'co-views';

export default views(path.join(__dirname, '../views/'), {
  map: {html: 'swig'}
});
```

创建视图文件`views/home.html`：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>React.js with ES6</title>
</head>
<body>

<div id="react-app"></div>

<!--<script src="bundle.js"></script>-->
<script src="http://localhost:8080/bundle.js"></script>
</body>
</html>
```

### 配置Webpack

由于我们需要使用ES6来编写React应用，因此我们需要将React代码编译为普通的JS代码，幸运的是，
我们可以借助Webpack和Babel来完成这项任务。大致的步骤如下：

1. 安装Webpack，包括如下依赖：
    - webpack
    - webpack-dev-server
2. 安装Babel相关依赖，用于编译React代码：
    - babel-loader
    - babel-preset-es2015
    - bebel-preset-react
3. 创建Webpack配置文件：`config/webpack.config.js`
4. 在`package.json`文件中添加编译React代码相关的`scripts`脚本：
        "scripts": {
            "start": "babel-node index.js",
            "build": "webpack --config config/webpack.config.js",
            "watch": "webpack-dev-server --config config/webpack.config.js --hot --inline --progress"
        },

Webpack的配置文件如下：

```javascript
// config/webpack.config.js
var path = require('path');
var webpack = require('webpack');
var node_modules = path.resolve(__dirname, '../node_modules');

var dir_src = path.resolve(__dirname, '../src');
var dir_build = path.resolve(__dirname, '../build');

module.exports = {
  entry: path.resolve(dir_src, 'main.jsx'),
  output: {
    path: dir_build, // for standalone building
    filename: 'bundle.js'
  },
  devServer: {
    contentBase: dir_build
  },
  module: {
    loaders: [
      {test: /src(\\|\/).+\.jsx?$/, exclude: /node_modules/, loader: 'babel', query: {presets: ['es2015', 'react']}}
    ]
  },
  plugins: [
    // Avoid publishing files when compilation fails
    new webpack.NoErrorsPlugin()
  ],
  stats: {
    colors: true // Nice colored output
  },
  // Create Sourcemaps for the bundle
  devtool: 'source-map'
};
```

## React应用

1. 安装React相关依赖：
    - react
    - react-dom
2. 编写React应用

React应用的入口文件：
```javascript
// src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom';
import Root from './components/root.jsx';

let attachElement = document.getElementById('react-app');

ReactDOM.render(<Root phrase="ES6"/>, attachElement);
```

React应用组件树的根文件：
```javascript
// src/components/root.jsx
import React from 'react';

class Root extends React.Component {
  render() {
    return <h1>Hello from {this.props.phrase}!</h1>;
  }
}

export default Root;
```

## 测试及启动

1. 使用`npm run watch`启动`webpack-dev-server`
2. 使用`npm start`启动服务器
3. 打开浏览器访问`http://localhost:3000`查看结果

## 参考文件

1. [React and ES6, Part 1](http://egorsmirnov.me/2015/05/22/react-and-es6-part1.html)
1. [React and Webpack Cookbook](https://fakefish.github.io/react-webpack-cookbook/index.html)
1. [React - Getting Started](https://facebook.github.io/react/docs/getting-started.html)
1. [Setting Up Babel 6](http://babeljs.io/blog/2015/10/31/setting-up-babel-6/)
1. E-book: [Exploring ES6](https://leanpub.com/exploring-es6/)
1. [JSX in Depth](https://facebook.github.io/react/docs/jsx-in-depth.html)
1. [Babel Handbook](https://github.com/thejameskyle/babel-handbook/blob/master/translations/zh-Hans/user-handbook.md)
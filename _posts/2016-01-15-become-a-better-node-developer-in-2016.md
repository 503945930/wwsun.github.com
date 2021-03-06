---
layout: post
title: 如何在2016年成为一个更好的Node.js开发者
category: knowledge
---

本文主要讨论一些进行Node.js开发的最佳实践和建议，这些建议不仅仅适合开发者，
还适合那些管理与维护Node.js基础架构的工作人员。遵循本文提供的这些建议，
能够让你更好的进行日常的开发工作。

<!--more-->

## Statement

- **原文地址：**https://blog.risingstack.com/how-to-become-a-better-node-js-developer-in-2016/
- **译者：**[景庄](http://wwsun.github.com)，Web开发者，主要关注JavaScript、Node.js、React、Docker等。

## 使用ES2015

在2015年的夏天，ES2015的最终草案（即ES6）正式发布了。该版本为JavaScript语言增加了大量的新的语言特性，主要包括：

- 箭头函数
- 模版字符串
- rest operator（不定参数）, argument spreading
- 生成器
- promises
- maps, sets
- symbols

以及很多其他特性。一个更加完整的新特性的列表你可以从[Kyle Simpson](https://twitter.com/getify)的[ES6 and Beyond](https://github.com/getify/You-Dont-Know-JS/tree/master/es6%20%26%20beyond)中进行了解。
并且它们中的绝大部分特性已经被加入到了Node.js v4中。

在客户端，你也可以借助Babel来使用ES6的所有新特性，Babel是一个JavaScript转译器。目前在服务器端，
我们只倾向于使用那些被加入到最新的稳定版本的特性，这样无需转译代码，这可以避免出现那些令我们头疼的潜在问题。

对于Node.js中的ES6的更多信息，你可以访问官方站点：[https://nodejs.org/en/docs/es6/](https://nodejs.org/en/docs/es6/)

## 回调约定 - 同时支持Promise

在去年，我们可能会推荐你为你的模块暴露错误优先的回调接口。但是随着生成器函数的正式标准化，并且**异步函数**也即将到来，
因此我们现在建议你在编写模块的接口时应该暴露**支持Promise**的的错误优先的回调函数。

为什么需要这样？首先回调接口是为了提供向后兼容性，为了能够在未来能够获得更好的兼容性，需要同时提供Promise支持。

你可以参考下面的例子来进一步的理解具体应该如何进行编程。在这个例子中`readPackage`函数读取了`package.json`文件，
并同时通过Promise和回调接口返回了它的内容。

```javascript
const fs = require('fs');

function readPackage (callback) {
  // as of now we do not have default values in Node.js
  callback = callback || function () {}
  return new Promise((resolve, reject) => {
    fs.readFile('./package.json', (err, data) => {
      if (err) {
        reject(err);
        return callback(err);
      }
      resolve(data);
      return callback(null, data);
    })
  })  
}

module.exports.readPackage = readPackage;
```

## 异步模式

在Node.js中，很长一段时间你只有两种方法来管理异步流：回调或者流（Stream）。对于回调函数而言，
你可以使用类似于[async](https://www.npmjs.com/package/async)这类库，
对于流而言，有[through](https://www.npmjs.com/package/through)、[bl](https://www.npmjs.com/package/bl)、[highland](http://highlandjs.org/)等库可以选择。

但是随着Promise、生成器、异步函数等被逐渐引入进标准的ECMAScript，JS中的流程控制也得到了极大的改善。

> 关于异步JavaScript的发展历史，你可以参考[异步JavaScript的发展历程](http://wwsun.github.io/posts/evolution-of-javascript-async.html)这篇博文。

## 错误处理

错误处理在应用开发过程中起着至关重要的作用：确定应用崩溃的时间，或者仅仅是打印错误信息，确保应用继续运行都是有一定难度的。

为了能够更简单的说明这个问题，我们决定将其分为两种：程序员错误（programmer errors）和运算错误（operational errors）。     

程序员错误就是我们所说的bug，由于你不知道程序运行的确切状态因此当出现错误时你最好立刻停止应用的运行（crash the process）。

另一方面，运算错误是由于系统或者远程服务本身所导致的问题。例如：请求超时和内存不足等。基于错误发生的特点，你可以对症下药，
然后重试，例如文件丢失，你可以去创建相应的文件。

### 在回调中进行错误处理

如果一个错误发生在异步操作的过程中，错误对象应该作为异步函数的第一个参数进行传递。你必须始终要检查该错误对象并进行错误处理。

在前面的有关**回调约定**的例子里面已经展示了如何在回调函数中进行错误的优先处理。

### 在Promise中进行错误处理

如果是下面的代码片段会发生什么情况？

```javascript
Promise.resolve(() => 'John')
  .then(() => {
    throw new Error('ops');
  })
  .catch((ex) => {
    console.log(ex);
  })
  .then(() => {
    throw new Error('ups');
    console.log(Doe');
  })
```

1. 在第3行会抛出一个异常。
2. catch会处理它，并且在stdout中打印出：`[Error: ops]`
3. 执行继续，并且在第9行会抛出一个新的错误
4. 没有了

的确没有什么了 - 最后一个被抛出的错误将会是静默的。你需要注意，你应该始终以一个catch语句作为promise链的最后一环。
这会为你解决很多头疼的问题。像下面这样：

```javascript
Promise.resolve(() => 'John')
  .then(() => {
    throw new Error('ops');
  })
  .catch((ex) => {
    console.log(ex);
  })
  .then(() => {
    throw new Error('ups');
    console.log(Doe');
  })
  .catch((ex) => {
    console.log(ex);
  });
```

现在会输出如下内容：

    [Error: ops]
    [Error: ops]

## 使用JavaScript标准风格

在过去几年中，我们会使用JSHint、JSCS、ESLint等非常有用的代码质量工具来尽可能的自动化检查我们的代码。

最近，当谈到代码风格的时候，我们使用[feross](https://github.com/feross)的[JavaScript标准风格](https://github.com/feross/standard)。

![js standard code style](https://cdn.rawgit.com/feross/standard/master/badge.svg)

原因是它非常的简单：无需任何配置文件，只需要将其放到项目中。主要包括如下一些规则：

- 使用2个空格作为缩进
- 字符串使用单引号 - 除了为了避免转义
- 不要包括没有被使用的变量
- 没有分号
- 永远不要以 （ 或者 [ 作为一行的开始
- 关键字后加空格 `if (condition) { ... }`
- 函数名后加空格 `function name (args) { ... }`
- 始终使用`===`代替`==`，但是可以使用`obj == null`来检查`null || undefined`。
- 始终要处理Node.js的`err`函数参数
- 始终要为浏览器全局变量增加`window`前缀，除了`document`和`navigator`
- 尽可能避免使用类似于`open`、`length`、`evet`、`name`等走位浏览器全局变量。

当然，如果你的 编辑器只支持ESLint的话，这里有一个ESLint的规则库用于使用标准风格，即[eslint-plugin-standard](https://github.com/xjamundx/eslint-plugin-standard)。
安装了这个插件后，你的`.eslintrc`文件可以是下面这样的：

    {
        "plugins": [
            "standard"
        ],
    }

## 12-Factor应用（The Twelve-Factor Application）

如今，软件通常会作为一种服务来交付，它们被称为网络应用程序，或软件即服务（SaaS）。
[12-Factor](http://12factor.net/zh_cn/)应用宣言描述了进行Web应用开发的最佳实践：

1. [基准代码](http://12factor.net/zh_cn/codebase)：一份基准代码，多份部署
2. [依赖](http://12factor.net/zh_cn/dependencies)：显示声明依赖
3. [配置](http://12factor.net/zh_cn/config)：在环境中存储配置
4. [后端服务](http://12factor.net/zh_cn/backing-services)：把后端服务当作附加资源
5. [构建、发布、运行](http://12factor.net/zh_cn/build-release-run)：严格分离构建和运行
6. [进程](http://12factor.net/zh_cn/processes)：以一个或多个无状态进程运行应用
7. [端口绑定](http://12factor.net/zh_cn/port-binding)：通过端口绑定提供服务
8. [并发](http://12factor.net/zh_cn/concurrency)：通过进程模型进行扩展
9. [易处理](http://12factor.net/zh_cn/disposability)：快速启动和优雅终止可最大化健壮性
10. [开发环境与线上环境等价](http://12factor.net/zh_cn/dev-prod-parity)：尽可能的保持开发、预发布、线上环境相同
11. [日志](http://12factor.net/zh_cn/logs)：把日志当作事件流
12. [管理进程](http://12factor.net/zh_cn/admin-processes)：后端管理任务当作一次性进程运行

这套理论适用于任意语言和后端服务（数据库、消息队列、缓存等）开发的应用程序。

## 开始新的项目

始终通过`npm init`命令来开始一个新项目。这可以为你的项目创建一个初始的`package.json`。

如果你想跳过初始的提问并直接使用默认的配置，只需要运行`npm init --yes`即可。

## 文件夹结构

Node项目的文件夹组织没有固定的格式，但是如果遵循某种惯例可让其他人更容易与你一起开发。以下是建议的文件夹结构：

    - .gitignore    从Git库中忽略的文件清单
    - .npmignore    不包括在npm注册库中的文件清单
    - LICENSE       模块的授权文件
    - README.md     以Markdown格式编写的模块README文件
    - bin           保存模块可执行文件的文件夹
    - doc           保存模块文档的文件夹
    - examples      保存如何使用模块的实际示例的文件夹
    - lib           保存模块代码的文件夹
    - man           保存模块的任何手册页的文件夹
    - package.json  描述模块的JSON文件
    - src           保存源文件的文件夹，经常用于CoffeeScript文件
    - test          保存模块测试的文件夹

如果不想将文件放入npm注册库，就将它们加入到.npmignore文件中。如果模块没有`.mpmignore`文件但有`.gitignore`文件，
则npm使用`.gitignore`文件的内容来忽略注册库中的内容。

## 监控你的应用

当发生某个故障或是故障即将发生时，及时的通知你，能够为你挽回损失。

为了进行应用的监控，你可以使用类似的SaaS产品或是开源软件。在开源软件方面，主要包括：[Zabbix](http://www.zabbix.com/),
[Collected](https://collectd.org/), [ElasticSearch](https://www.elastic.co/products/elasticsearch)和[Logstash](https://www.elastic.co/products/logstash)。

如果你不想要自己进行部署，可以考虑使用线上的服务，你可以尝试使用[Trace](http://trace.risingstack.com/?utm_source=how-to-become-a-better-node-js-developer-in-2016&utm_medium=rsblog&utm_campaign=tracebeta&_ga=1.25646138.685893790.1443447346)，
它是我们公司开发的Node.js和微服务监控解决方法。

![trace img](https://risingstack-blog.s3.amazonaws.com/2015/Dec/trace_transaction-1451512486757.png)

## 使用构建系统

尽可能的自动化一切东西。没有什么比让开发来做应该让grunt做的事情更无聊和令人恼火的了，这不仅浪费时间，而且没有意义。

现如今JvavaScript的这类工具已经非常的丰富了，包括Grunt, Gulp, 和Webpack，你知道几个就行。

在RisingStack，绝大部分的前端开发新项目都是使用Webpack来进行自动化构建，其他类型的则使用gulp实现自动化任务。
对于新手而言，Webpack可能会花费大量的时间去理解，所以我强烈建议你去阅读一下[Webpack Cookbook](http://christianalfoni.github.io/react-webpack-cookbook/)。

## 使用最新的长期支持（LTS）的Node版本

为了能够更好的获取稳定性和新特性，我们建议你使用最新的Node的LTS（长期支持）版本，它们是使用偶数发布编号的版本。
当然，你也可以自由的使用最新的实验版本，即称为稳定发布版本的使用奇数发布编号的。

如果你需要为多个项目工作，并且使用了不同的Node.js版本，建议你最好使用一个Node版本管理器——[nvm](https://github.com/creationix/nvm)。

更多信息你可以参考Node.js官方网站的发布信息：

[What You Should Know about Node.js v5 and More](https://nodejs.org/en/blog/community/node-v5/)

## 每周更新你的项目依赖

养成每周更新一次你的项目依赖的习惯。这方面，你可以使用`npm outdated`或者是[ncu](https://www.npmjs.com/package/npm-check-updates)包。

以ncu为例，首先全局安装ncu：

    npm install -g npm-check-updates
    
在项目根目录执行`ncu`命令：

    $ ncu
    
    express           4.12.x  →   4.13.x
    multer            ^0.1.8  →   ^1.0.1
    react-bootstrap  ^0.22.6  →  ^0.24.0
    react-a11y        ^0.1.1  →   ^0.2.6
    webpack          ~1.9.10  →  ~1.10.5
    
    Run with -u to upgrade your package.json
    
更新项目依赖

    $ ncu -u
    
    express           4.12.x  →   4.13.x
    
    package.json upgraded

## 选择合适的数据库

当我们谈到Node.js和数据库的时候，可能你想到的第一个技术是MongoDB。当然这并没有什么错，但是你不应该直接就去使用它。
在这么做之前你需要问你自己和你的团队几个问题。包括下面几个：

- 应用会有结构化数据吗？
- 应用会进行交易处理吗？
- 数据需要存放多长时间？

可能你需要的仅仅是Redis，或者是如果你有结构化数据，那么你要用的可能是PostgrelSQL。
如果你需要在Node.js中使用SQL的话，你可以看看[knex](http://knexjs.org/)。

## 使用语义版本控制（Semantic Versioning）

> 语义版本控制是一种为了兼容性空啊率的使用三段式版本号的正式约定，即：`major.minor.patch`，分别为主版本，次版本，补丁。

如果是一个不会向后兼容（backward-compatible）的API变化使用主版本号。当添加新的特性且API变化是向后兼容的时候使用次版本号。
如果只是对Bug进行修复可以使用包版本号。

幸运的是，你可以使用[semantic-release](https://github.com/semantic-release/semantic-release)这个模块自动化你的JavaScript的模块发布。

## 坚持阅读

在JavaScript和Node.js世界，坚持保持对最新的新闻和技术进展的关注是件具有挑战的事情。
为了能够让这件事变得简单，确保你订阅了如下几个媒体：

- [Node.js新闻通信周刊](http://nodeweekly.com/)
- [微服务新闻通信周刊](https://microserviceweekly.com/)
- [Changelog周刊 —— 开源新闻](https://changelog.com/)
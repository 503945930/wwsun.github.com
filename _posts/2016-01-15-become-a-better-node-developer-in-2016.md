---
layout: post
title: 如何在2016年成为一个更好的Node.js开发者
category: knowledge
---

本文提供的建议和最佳事件不仅仅适合开发者，还适合用于管理和维护Node.js基础架构，
本文将会指导你更高的进行日常的开发工作，以及一些其他的建议。

<!--more-->

## Statement

> 原文地址：https://blog.risingstack.com/how-to-become-a-better-node-js-developer-in-2016/

## 使用ES2015

在2015年的夏天ES2015的最终草案（即ES6）正式发布了。在该版本中为JavaScript语言增加了大量的新的语言特性，主要包括：

- 箭头函数
- 模版字符串
- rest operator（不定参数）, argument spreading
- 生成器
- promises
- maps, sets
- symbols

以及很多其他特性。一个更加完整的新特性的列表你可以从[Kyle Simpson](https://twitter.com/getify)的[ES6 and Beyond](https://github.com/getify/You-Dont-Know-JS/tree/master/es6%20%26%20beyond)中进行了解。
它们中的绝大部分特性被加入到了Node.js v4中。

在客户端，你也可以借助Babel来使用ES6的所有新特性，Babel是一个JavaScript转译器。，目前为止，
在服务器端我们只倾向于使用那些被加入到最新的稳定版本的特性，而无需编译代码，避免出现那些令我们头疼的潜在问题。

对于Node.js中的ES6的更多信息，你可以访问官方站点：[https://nodejs.org/en/docs/es6/](https://nodejs.org/en/docs/es6/)


## 回调约定 - 同时支持Promise

在过去的一年，我们推荐你为模块暴露错误优先的回调函数接口。由于生成器函数已经标准化了，并且异步函数也即将来临，
因此你的模块应该暴露同时支持Promise的错误优先的回调函数。

为什么？为了提供向后兼容性，因此回调函数接口必须要提供，为了能够更好的为未来的兼容性做打算，你同时应该提供Promise支持。

为了说明如何达到这效果，可以参考如下的代码。在这个例子中`readPackage`函数读取了`package.json`文件，
并同时通过Promise和回调接口返回了它的内容。



## 异步模式

在Node.js中，很长一段时间你只有两种方法来管理异步流：回调或者流（Stream）。对于回调函数而言，
你可以使用类似于[async](https://www.npmjs.com/package/async)这类库，
对于流而言，有[through](https://www.npmjs.com/package/through)、[bl](https://www.npmjs.com/package/bl)、[highland](http://highlandjs.org/)。

但是随着Promise、生成器、异步函数等被逐渐引入进标准的ECMAScript，JS中的流程控制也得到了极大的改变。

> 关于异步JavaScript的发展历史，你可以参考[异步JavaScript的发展历程](http://wwsun.github.io/posts/evolution-of-javascript-async.html)这篇博文。

## 错误处理

错误处理在应用开发过程中起着至关重要的作用：确定应用崩溃的时间，或者仅仅是打印错误信息，确保应用继续运行都是有一定难度的。

为了能够更简单的说明这个问题，我们决定将其分为两种：程序员错误（programmer errors）和运算错误（operational errors）。     

程序员错误就是我们所说的bug，由于你不知道程序运行的确切状态因此当出现错误时你最好立刻停止应用的运行（crash the process）。

另一方面，运算错误是由于系统或者远程服务本身所导致的问题。例如：请求超时和内存不足等。基于错误发生的特点，你可以对症下药，
然后重试，例如文件丢失，你可以去创建相应的文件。

### 在回调中进行错误处理

如果一个错误发生在异步操作的过程中，错误对象应该作为异步函数的第一个参数进行传递。你必须始终要检查该错误对象并进行错误处理。

下面的代码判断显示了进行错误优先的回调函数处理的例子：



### 在Promise中进行错误处理

如果是下面的代码片段会发生什么情况？



1. 在第3行会抛出一个异常。
2. catch会处理它，并且在stdout中打印出：`[Error: ops]`
3. 执行继续，并且在第9行会抛出一个新的错误
4. 没有了

的确没有什么了 - 最后一个被抛出的错误将会是静默的。你需要注意，你应该始终以一个catch语句作为promise链的最后一环。
这会为你解决很多头疼的问题。像下面这样：



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

## The Twelve-Factor Application

## 开始新的项目


## 监控你的应用


## 使用构建系统


## 使用最新的长期支持（LTS）的Node版本


## 每周更新你的项目依赖


## 选择合适的数据库


## 使用语义版本号


## 阅读
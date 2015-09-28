---
layout: post
title: Node.js常见面试题
category: technique
---

如果你希望找一份有关Node.js的工作，但又不知道从哪里入手考察自己对Node.js的掌握程度。
本文就提供了这样的一份Node.js面试题列表，通过考察Node.js编程中的一些主要细节，
来帮助你评估你对于Node.js开发的掌握程度。

<!--more-->

在进入正文之前，需要提前声明两点：

1. 这些问题只是Node.js知识体系的一个局部，并不能完全考察被面试者的实际开发能力。
2. 对现实世界开发中遇到的问题，需要的是随机应变与团队合作，所以你可以尝试结对编程。

## Node.js面试题列表

- 什么是错误优先的回调函数？
- 你如何避免回调地狱？
- 你如何用Node来监听80端口？
- 什么是事件循环？
- 什么工具可以用来保证一致的风格？
- 运算错误与程序员错误的区别？
- 为什么npm是有用的？
- 什么是stub？举个使用场景？
- 什么是测试金字塔？当我们谈到HTTP API时，我们如何实施它？
- 你最喜欢的HTTP框架，并说明原因？

现在，我们依次来解答这些问题吧。

### 什么是错误优先的回调函数？

错误优先的回调函数用于传递错误和数据。第一个参数始终应该是一个错误对象，
用于检查程序是否发生了错误。其余的参数用于传递数据。例如：

	fs.readFile(filePath, function(err, data) {  
		if (err) {
			//handle the error
		}
		// use the data object
	});
	
**解析：**这个题目的主要作用在于检查被面试者对于Node中异步操作的一些基本知识的掌握。

### 如何避免回调地狱

你可以有如下几个方法：

- 模块化：将回调函数分割为独立的函数
- 使用Promises
- 使用`yield`来计算生成器或Promise

**解析：**这个问题有很多种答案，取决你使用的场景，例如ES6, ES7，或者一些控制流库。

### 在Node中你如何监听80端口

这题有陷阱！在类Unix系统中你不应该尝试监听80端口，因为这需要超级用户权限，
因此不建议让你的应用监听这个端口。

目前，如果你想让你的应用一定要监听80端口，可以这么做：让你的Node应用监听大于1024的端口，
然后在它前面在使用一层方向代理（例如[nginx](http://nginx.org/)）。

**解释：**这个问题用于检查被面试者是否有实际运行Node应用的经验。

### 什么是事件循环

Node只运行在一个单一线程上，至少从Node.js开发者的角度是这样的。在底层，
Node是通过[libuv](https://github.com/libuv/libuv)来实现多线程的。

Libuv库负责Node API的执行。它将不同的任务分配给不同的线程，形成一个事件循环，
以异步的方式将任务的执行结果返回给V8引擎。可以简单用下面这张图来表示。

![Event Loop](/img/posts/150928-event-loop.PNG)
（图片来源于网络）

每一个I/O都需要一个回调函数——一旦执行完便推到事件循环上用于执行。
如果你需要更多详细的解释，可以参考[这个视频](https://www.youtube.com/embed/8aGhZQkoFbQ)。
你也可以参考[这篇文章](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)。

**解释：**这用于检查Node.js的底层知识，例如什么是libuv，它的作用是什么。

### 哪些工具可以用来确保一致性的风格

你可以有如下的工具：

- [JSLint](http://jslint.com/)
- [JSHint](http://jshint.com/)
- [ESLint](http://eslint.org/)
- [JSCS](http://jscs.info/) - 推荐

在团队开发中，这些工具对于编写代码非常的有帮助，能够帮助强制执行给定的风格指南，
并且通过静态分析捕获常见的错误。

**解析：**用于检查被面试者是否有大型项目开发经验。

### 运算错误与程序员错误的区别

运算错误并不是bug，这是和系统相关的问题，例如请求超时或者硬件故障。而程序员错误就是所谓的bug。

**解析：**这个题目和Node关系并不大，用于考察面试者的基础知识。

### 为什么npm包管理器有帮助

> This command locks down the versions of a package's dependencies so
that you can control exactly which versions of each dependency will
be used when your package is installed. - npmjs.com

在你开发Node应用时npm会非常的有用，它可以帮你确定你的依赖的具体的版本号。

**解析：**它能考察面试者使用npm命令的基础知识和Node.js开发的实际经验。

### 什么是Stub？举个使用场景

Stub是用于模拟一个组件/模块的一个函数或程序。在测试用例中，Stub可以为函数调用提供封装的答案。
当然，你还可以在断言中指明Stub是如何被调用的。

例如在一个读取文件的场景中，当你不想读取一个真正的文件时：

	var fs = require('fs');
	
	var readFileStub = sinon.stub(fs, 'readFile', function (path, cb) {  
		return cb(null, 'filecontent');
	});
	
	expect(readFileStub).to.be.called;  
	readFileStub.restore(); 
	
**解析：**用于测试被面试者是否有测试的经验。如果被面试者知道什么是Stub，
那么可以继续问他是如何做单元测试的。

### 什么是测试金字塔？当我们在谈论HTTP API时如何实施它？

[测试金字塔](http://zyzhang.github.io/blog/2013/04/28/test-pyramid/)指的是：
当我们在编写测试用例时，底层的单元测试应该远比上层的端到端测试要多。

![Test Pyramid](/img/posts/150928-test-pyramid.jpeg)

当我们谈到HTTP API时，我们可能会涉及到：

- 有很多针对模型的底层单元测试
- 但你需要测试模型间如何交互时，需要减少集成测试

**解析：**本文主要考察被面试者的在测试方面的经验。

### 你最喜欢的HTTP框架以及原因

这题没有唯一的答案。本题主要考察被面试者对于他所使用的Node框架的理解程度，
考察他是否能够给出选择该框架的理由，优缺点等。

## Statement

> 原文地址：https://blog.risingstack.com/node-js-interview-questions/

## References

1. http://zyzhang.github.io/blog/2013/04/28/test-pyramid/
2. http://www.ruanyifeng.com/blog/2014/10/event-loop.html
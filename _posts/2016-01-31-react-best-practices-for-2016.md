---
layout: post
title: 2016年编写React应用的若干建议
category: knowledge
---

在2015年，伴随着React.js大量新特性的发布和React开发者大会的召开，React火遍了全球。
如果你想查看2015年React的各项重要里程碑，你可以参考[React in 2015](https://blog.risingstack.com/react-in-2015/)这篇文章。
对于2016年而言最有趣的一个问题是：我们应该如何编写一个应用，并且推荐使用的库是什么？

<!--more-->

## Statement

- **原文地址：** https://blog.risingstack.com/react-js-best-practices-for-2016/
- **译者：** [景庄](http://wwsun.github.com)，Web开发者，主要关注JavaScript、Node.js、React、Docker等。

> 本文所谓的最佳实践仅仅是一些个人建议，对于不同的开发者而言，由于其个人开发经历有长短之分，本文的内容仅一家之言，仅供参考。

如果你还是刚刚入门React.js，你需要首先阅读[React.js Tutorial](https://blog.risingstack.com/the-react-way-getting-started-tutorial/)，
或者是Pete Hunt的[React howto](https://github.com/petehunt/react-howto)。

## 处理数据

在React.js应用中处理数据超级简单，但是同时也是有一定的挑战的。
这是由于开发者构建渲染树时可以通过多种方式传递属性给React组件所导致的。然后，在你应该更新视图时并不需要经常明显的。

在2015年已经有了多个不同版本的Flux库发布了，并且继续会有更加功能完善和适应性的解决方案。让我们先来看看现状如何：

### Flux


### 使用Redux

> Redux是一个面向JavaScript应用的可预测状态容器

如果你觉得你需要Flux，并且需要一个类似的解决方案，你可以查看一下[redux](https://github.com/rackt/redux)和
Dan Abramov的[redux入门课程](https://egghead.io/series/getting-started-with-redux)，
通过它们你可以快速的入门开发技能。

> Redux拥抱了Flux的思想，但是它借鉴了Elm从而避免了复杂性。

### 保持你的状态扁平

API经常会返回嵌套的资源。对Flux或者基于Redux的架构而言很难处理这些数据。我们建议尽可能的扁平化这些资源，
可以使用诸如[normalizr](https://github.com/gaearon/normalizr)这样的库，原则就是尽可能的扁平你的状态。

提示：

```javascript
const data = normalizr(response, arrayOf(schema.user));
state = _.merge(state, data.entities);
```

我们使用[isomorphic-fetch](https://www.npmjs.com/package/isomorphic-fetch)来与我们的API进行通信。

### 使用不可变状态（immutable states）

> 可共享的可变状态是一切罪恶的根源 - Peter Hunt, React.js Conf 2015

所谓[不可变对象](https://en.wikipedia.org/wiki/Immutable_object)指的是那些对象一旦被创建了就不能被修改了的对象。

使用不可变对象可以为避免很多莫名其妙的问题，并且通过它们的引用级别的相等性检查来改善应用的渲染性能。
就像在`shouldComponentUpdate`中的那样：

```javascript
shouldComponentUpdate(nextPorps) {
  // instead of object deep comparsion
  return this.props.immutableFoo !== nextProps.immutableFoo;
}
```

**如何在JavaScript中获取不变性？**

像下面的例子那样执行的编写代码是很困难的，因为你需要经常使用[deep-freeze-node](https://www.npmjs.com/package/deep-freeze-node)这样的工具来执行单元测试
（在变化之前冻结对象，并在之后对结果进行验证）。

```javascript
return {
  ...state,
  foo
}

return arr1.concat(arr2);
```

相信我，这些是非常显而易见的例子。

一个不那么复杂的方式但是也先得不那么自然的方法是使用[Immutable.js](https://facebook.github.io/immutable-js/)。

```javascript
import { fromJS } from 'immutable';

const state = fromJS( {bar: 'biz'});
const newState = foo.set('bar', 'baz');
```

Immutable.js非常的快，并且其背后的思想也非常的漂亮。
我推荐你看一下[Lee Byron](https://twitter.com/leeb)的有关[Immutable Data和React](https://www.youtube.com/watch?v=I7IdS-PbEgI)的讲座视频，
即使你并不想要使用它。通过这个视频你能更进一步的理解它背后是如何工作的。

### Observables和Reactive解决方案

如果你并不喜欢Flux/Redux，或者仅仅是想要变得更加的Reactive，也不需要失望。
还有其他的方法可以处理你的数据。下面的清单简单的罗列了你可能会想用到的解决方案：

- [cycle.js](http://cycle.js.org/)  - "A functional and reactive JavaScript framework for cleaner code"
- [rx-flux](https://github.com/fdecampredon/rx-flux) - "The Flux architecture with RxJS"
- [redux-rx](https://github.com/acdlite/redux-rx) - RxJS utilities for Redux."
- [mobservable](https://mweststrate.github.io/mobservable/) - "Observable data. Reactive functions. Simple code."

## 路由

几乎每一个客户端应用都包括一些路由。如果你在浏览器中使用React.js，当你需要使用到路由的时候你需要选择一个合适的库。

我们的选择之一是由杰出的[rackt](https://github.com/rackt)社区提供的[react-router](https://github.com/rackt/react-router)，
Rackt总是能够为React.js爱好者提供高质量的资源。

如果你需要集成进`react-router`，你需要首先阅读它的[文档](https://github.com/rackt/react-router/tree/master/docs)，但是这里有一个更重要的事：
如果你使用了Flux/Redux，我们推荐你保持你的路由的状态尽可能的和你的store/全局状态保持同步。

同步的路由状体能够通过Flux/Redux的行为（actions）帮助你控制路由的行为，以及在你的组件中读取路由状态和参数。

Redux用户可以简单的使用[redux-simple-router](https://github.com/rackt/redux-simple-router)这个库来完成任务。

### 代码分割，懒加载

只有一小部分的webpack用户知道如何将应用程序的代码的构建包结果分割为多个JavaScript代码块：

```javascript
require.ensure([], () => {  
  const Profile = require('./Profile.js')
  this.setState({
    currentComponent: Profile
  })
})

```

这对大型应用而言非常的有用，因为对于用户浏览器而言无需在每次部署后再仅仅是一个Profile页面时下载一个完成的代码包了
（避免了很少被使用到的代码被重复下载）。

虽然更多的代码块会导致更多的HTTP请求——但是如果通[过HTTP/2多路复用](https://http2.github.io/faq/#why-is-http2-multiplexed)技术就不是一个问题了。

通过组合[块哈希](https://christianalfoni.github.io/react-webpack-cookbook/Optimizing-caching.html)（chunk hashing）技术你也可以在代码发生变化后优化缓存命中率。

react-router的下一个版本将会进一步的改进代码分割部分的功能。

对于react-router的未来发展目标，可以参考[Ryan Florence](https://twitter.com/ryanflorence)的[Welcome to Future of Web Application Delivery](https://medium.com/@ryanflorence/welcome-to-future-of-web-application-delivery-9750b7564d9f#.vuf3e1nqi)。

## 组件

很多人都曾抱怨过JSX。首先，你需要知道它在React中是可选的。

到最后，它可以通过Babel被编译为JavaScript。你可以编写JavaScript来替代JSX，但是对于处理HTML的时候使用JSX会被JavaScript更自然。
尤其是因为甚至只有少部分的人可以仍然理解和修改需要的部分。

> JSX是一个类似于XML的JavaScript语法扩展。你可以在React中使用一个简单的JSX语法转换器。——[JSX in depth](https://facebook.github.io/react/docs/jsx-in-depth.html)。

如果你需要阅读更多的关于JSX的内容，
你可以查看[JSX Looks Like An Abomination - But it’s Good for You](https://medium.com/javascript-scene/jsx-looks-like-an-abomination-1c1ec351a918#.ca28nvee6)这篇文章。

### 使用类

React能够很好的与ES2015的类工作：

```javascript
class HelloMessage extends React.Component {  
  render() {
    return <div>Hello {this.props.name}</div>
  }
}
```

我们更喜欢在mixins之上的高阶组件，这样可以让`createClass`先得更加像是一个语法问题，而不是一个技术问题。
我们相信如果使用`createClass`来替代`React.Component`并没有什么错，反之亦然。

### PropType


### 高阶组件


## 测试

### 组件测试


### Redux测试


### 使用npm


### 打包尺寸



### 组件级别热重新加载


### 使用ES2015


### Linters


## GraphQL和Relay


## Takeaway from these React.js Best Practices
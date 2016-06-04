---
layout: post
title: 使用ES6编写React应用（5）：工程体系和组件编码
category: knowledge
---

React是一个能够实现简单优雅的组件化方案的库。借助React能够快速的构建组件化前端应用，并且将关注的重点集中在视图层。
但是对于构建React应用而言，会涉及到大量的附加工具的使用，例如webpack, babel, eslint, livereload, koa等等，
为了简化这些流程，本文将带你快速的构建React工程体系和编码规范。

<!--more-->

## Statement

- **作者：** [景庄](http://wwsun.github.com)，Web开发者，主要关注JavaScript、Node.js、React、Docker等。

## 源代码

阅读本系列文章时可以参考[Github仓库](https://github.com/wwsun/react-es6-tutorial)中的代码。

## 前提

如果你对React还不够了解，你需要阅读[官方文档](https://facebook.github.io/react/docs/getting-started.html)，或者你可以回到本系列的前四篇文章去了解相关内容。

- [Part 1: 简介及环境配置](http://wwsun.github.io/posts/react-with-es6-part-1.html)
- [Part 2: 组件的属性和状态](http://wwsun.github.io/posts/react-with-es6-part-2.html)
- [Part 3: 组件中方法绑定](http://wwsun.github.io/posts/react-with-es6-part-3.html)
- [Part 4: 高阶组件](http://wwsun.github.io/posts/react-with-es6-part-4.html)

对于开发React应用而言，我们推荐使用ES6语法，甚至基于Babel，你可以提前使用到`stage-0`特性，
这导致在正式的编码之前你需要花费大量的时间配置开发环境，包括Babel, Webpack, Livereload等。

为了简化这些配置过程，在本文中，我们推荐使用[seu](https://github.com/wwsun/seu)来辅助你完成相关的工作，`seu`是一个轻量级的react应用开发工具集，
能够让你免于配置复杂的开发环境，而是直接进入应用的编码阶段，目前`seu`包括了脚手架生成、
项目构建、动态watch、代码Lint等功能。在默认情况下，你可以直接使用es6和`stage-0`特性进行编码，
并且支持`scss`和`less`。

## 项目开发及工程体系

在正式开始之前，我们需要全局安装`seu`，安装过程非常简单，只需要通过`npm`全局安装即可。
确保你的Node版本至少是`v4.4`。

```bash
$ npm install seu -g
$ seu --version
0.3.2
```

然后是创建项目的文件夹，然后进入该文件夹：

```bash
$ mkdir react-demo && cd react
```

进入项目文件夹后，你可以借助`seu`来快速生成项目的脚手架：

```bash
$ seu init
```

执行该命令会自动的项目中创建初始项目结构，并安装默认的依赖项，你可以通过检查`package.json`来查看，
你可以更新所生成的文件中的相关内容。

安装项目依赖：

```bash
$ npm install
```

最后，使用如下命令启动测试服务器：

```bash
$ seu server
```

打开浏览器访问`http://localhost:3000/demo`即可看到结果了

## 开发及编码规范

### 使用ES6和stage-0特性编写React代码

在初始化的项目中，默认你可以使用[ES6]()和[stage-0]()特性来编写React代码，是的，你无需进行任何配置，
你便可以开始编码了。
具体可以参考这篇[文章](https://csspod.com/refactoring-react-components-to-es2015-classes/)。
这里，我们先来看下大致的编码格式：

```jsx
import React, { Component, PropTypes } from 'react';

class MyComp extends Component {
  
  // props 的类型
  static propTypes = {
    someProp: PropTypes.string,
  }
  
  // porps 的默认值
  static defaultProps = {
    somePorp: '',
  }
  
  // 构造器
  constructor(props, context) {
    super(props, context);
    
    // 在这里设置初始出台
    this.state = {
      someState: props.someProp,
    }
  }
  
  // 在jsx中进行事件处理需要绑定 this 
  handleClick() {
    console.log(this);
  }
  
  render() {
    // 你需要为事件处理函数绑定 this
    // 这里我们使用的是绑定操作符 ::
    return (
      <div onClick={::this.handleClick}> Hello world </div>
    )
  }
}
```

以下是几点具体的说明：

### `class`和`extends`

首先，使用`class`和`extends`来代替原本的`createClass`工厂函数。

### 静态属性：`propTypes`和`defaultProps`

其次，由于ES2015定义的类只能定义方法，不能定义属性（可能在未来改进），
因此`propTypes`和`defaultProps`需要提取为类本身的属性（静态属性）。有两种方式实现：

```jsx

// 直接添加类属性
MyComp.propTypes = {
  someProp: React.PropTypes.string,
}

// 使用stage-0的静态属性特性
// 这两种方式达到的效果是相同的
class MyComp extends Component {
  
  static propTypes = {
    somePorp: React.PropTypes.string,
  }
}

```

### 初始化状态 `state`

然后，将`getInitialState`转移。在构造函数中使用`this.state = { }`来设置初始状态。注意，
如果要设置初始状态，`constructor`**必须先调用**`super`方法，然后再进行初始状态设置、`this`绑定等操作。
原因会在后面补充解释。

### Context

接下来（可选的），提取`Context`相关的属性（方法），`Context`的作用简而言之就是让你能够直接在组件树上传递数据，
而无需逐层向下传递属性。你可以通过[官方文档](https://facebook.github.io/react/docs/context.html)获取更详细的内容。
这里举一个简单的例子：

```jsx
class Foo extends React.Component {  
  
  // 注意，如果想要获得 context 上的数据，必须要定义 contextTypes
  // 否则，this.context 是一个空对象
  static contextTypes = {
    tag: React.PropTypes.string,
    className: React.PropTypes.string,
  }
  
  // 如果在子组件中需要获取 context, 需要在构造器中传入 context 参数
  constructor(props, context) {
    super(props, context);
    this.state = {
      tag: context.tag,
      
      // 从context中获取className
      className: this.context.className,
    };
  }

  render() {
    var Tag = this.state.tag;
    return <Tag className={this.state.className} />;
  }
}

// 外层组件
class Outer extends React.Component {  
  
  // 这里，我们为Outer组件（称为context provider）添加了 childContextTypes 和 getChildContext
  // React会自动将信息向下传递，仍何子树上的组件通过定义 contextTypes 都可以获取到这些信息 
  static childContextTypes = {
    tag: React.PropTypes.string,
    className: React.PropTypes.string,
  }
  
  getChildContext() {
    return {
      tag: 'span',
      className: 'foo',
    };
  }

  render() {
    return <Foo />;
  }
}
```

上面的例子来源于React官方[测试用例](https://github.com/facebook/react/blob/757756f682ab11961479e3c10adb0444271fdb9e/src/isomorphic/modern/class/__tests__/ReactES6Class-test.js)。
如果在组件中定义了`contextTypes`，
[有些生命周期方法](https://facebook.github.io/react/docs/context.html#referencing-context-in-lifecycle-methods)需要接收一些额外context对象作为参数。

### 无状态组件

对无状态组件而言，你可以这么写代码：

```javascript
// 同样，如果需要使用 context
// 需要定义 context 作为函数的形参
const Button = (props, context) => {
  return (
    <button style={{background: context.color}}>
      {props.children}
    </button>
  );
}

// 同样，需要指定 contextTypes/
Button.contextTypes = {color: React.PropTypes.string};

export default Button;
```

### 上下文绑定

最后，是绑定`this`。因为使用ES6方式编写的组件的实例不会自动绑定`this`，需要手动绑定。
有几种方式，你可以参考[这篇文章](http://wwsun.github.io/posts/react-with-es6-part-3.html)。
在之前的例子中，我们使用了`stage-0`特性的绑定操作符`::`，它只是`bind()`方法的语法糖。

### 补充：ES6继承机制说明

这里，我们再补充下对继承机制的说明：

ES5的继承方式：实质是先创造子类的实例对象`this`，然后再将父类的方法添加到`this`上面（`Parent.apply(this)`）。
ES6的继承机制：先创造父类的实例对象`this`（所以必须先调用`super`方法），然后再用子类的构造函数修改`this`。

如果子类没有显式定义`constructor`方法，这个方法会被默认添加。此外，在子类的构造函数中，只有调用了`super`之后，
才可以使用`this`关键字，否则会报错。这是因为子类实例的构建，是基于对父类实例加工，只有 `super` 方法才能返回父类实例。
详细内容可以参考软老师的[文章](http://es6.ruanyifeng.com/#docs/class)。

## 无状态组件与高阶组件

### 编写无状态组件

在上面，我们编写了一个[无状态组件](https://medium.com/@joshblack/stateless-components-in-react-0-14-f9798f8b992d#.w1xcb1k46)`Button`，
它是一个纯函数。React在'v0.14'时引入了无状态组件，这种组件主要有两个特点：

1. 没有state
2. 所有相同的 props 一定对应相同的标记输出，也就是所谓的[纯函数](http://www.zcfy.cc/article/251)

为啥要写无状态组件，官方是这么说的：

> This pattern is designed to encourage the creation of these simple components that should comprise large portions of your apps.
In the future, we’ll also be able to make performance optimizations specific to these components by avoiding unnecessary checks and memory allocations.
-- Ben Alpert

并且通过这种方式编写的React代码，能够像写JavaScript那样，更加的直观和简单。
Redux官方的[todo](https://github.com/reactjs/redux/tree/master/examples/todos/components)是一个很好的示例，值得参考。

### Mixin和高阶组件

使用ES6方式编写的React组件，[默认不支持Mixin](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750#.iv610fbsm)，
React官方推荐使用高阶组件的方式。[在绝大部分时候](https://github.com/brigand/react-mixin#aside-do-i-need-mixins)，
你无需使用Mixin，[高阶组件](http://wwsun.github.io/posts/react-with-es6-part-4.html)会更简单和清晰。

当然，如果你确实要用的话，我们可以借助[react-mixin](https://github.com/brigand/react-mixin)这个npm模块，
结合`stage-0`的[装饰器](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841#.pmmvh3sgw)特性，
我们可以这么写：

```javascript
import reactMixin from 'react-mixin';

let mixin = {
  // something to do
}

@reactMixin.decorate(mixin);
class Foo() {
  // todo
}
```

## 使用CSS预处理器

在初始化的项目中，默认你可以使用SASS来编写CSS样式规则。为什么我们推荐你使用CSS预处理器呢？其实原因很简单：

- CSS语法不够强大，无法嵌套，导致大量的书写重复
- 没有变量，没有函数，没有模块

基于这些原因，有理由使用CSS预处理器来进行CSS的编写，使其更像一种编程开发工作。预处理器无外乎主流的Less, Sass, Stylus等。
本文使用的是SASS，不过使用的SCSS版本，因为更加的直观，容易理解。组件的样式导入也非常简单：

```jsx
import 'index.scss';
```

原则上，每个组件都有自己的样式文件。如果你想学习sass，你可以访问[这个地址](http://www.w3cplus.com/sassguide/)。

## 代码Lint和编码风格

为什么要使用Lint工具，引用一段话：

> Either as a gentle reminder in my editor of choice while working, and keeping an eye on me with every save.
Or as as part of my build process by notifying me of possible errors and bad practices that could cause unexpected results or crashes.
-- Evan Schultz

我们选择[ESLint](http://eslint.org/docs/user-guide/configuring)作为Lint工具，它是由Nicholas C. Zakas大神创建的，
它的宗旨是“everything is pluggable, every rule is standalone”。ESLint具有可扩展，每条规则独立，不内置编码风格等特点，
更重要的是它支持JSX语法的检测，更多内容你可以参考[这篇文章](https://csspod.com/getting-started-with-eslint/)。
ESLint的详细使用规范可以参考[官方文档](http://eslint.org/docs/user-guide/configuring)。

在编码风格方面，我们的建议是遵循Airbnb的[JavaScript风格规则](https://github.com/airbnb/javascript)，
以及[React/JSX风格规范](https://github.com/airbnb/javascript/tree/master/react)。
无需很多的配置，你便可以快速的开始编写复合业界规范的代码。

### 配置ESLint

如果你只在命令行使用ESLint，那么你无需做任何配置，因为`seu`已经内置了整套Lint方案，
你只需要在项目的根目录中，在命令行执行`seu lint`命令即可。你可以跳过本节内容。

如果你使用的是类似于`WebStrom`这样的IDE工具的话，你想让WebStrom来辅助Lint，
你需要进行一些[配置工作]()。

为了能够快速开始，我们使用的是来自Airbnb的[eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb)，
默认的规则包括ECMAScript 6+和react。为此，你需要安装如下开发依赖：

- `eslint`
- `eslint-plugin-import`
- `eslint-plugin-react`
- `eslint-plugin-jsx-a11y`

```bash
$ npm install --save-dev eslint-config-airbnb eslint-plugin-import eslint-plugin-react eslint-plugin-jsx-a11y eslint
```

ESLint的支持[指定解析器](http://eslint.org/docs/user-guide/configuring#specifying-parser)，
默认解析器为[Espree](https://github.com/eslint/espree)，你也可以使用其他的解析器，例如`esprima`或`babel-eslint`。
这里我们选择的是[babel-eslint](https://www.npmjs.com/package/babel-eslint)，因为它能干lint所有有效的Babel代码。

安装`babel-eslint`，作为lint规则的解析器：

```bash
$ npm install --save-dev babel-eslint
```

然后在项目的根目录创建`.eslintrc`文件（支持JSON和YAML两种语法），我们需要在该文件中加上`"extends: "airbnb"`，
具体内容如下：

```json
{
  "root": true,
  "parser": "babel-eslint",
  "extends": "airbnb",
  "rules": {
    "constructor-super": "error"
  }
}
```

在`package.json`的`scripts`脚本中添加`lint`脚本。

```
"lint": "eslint --ext .jsx,.js src",
```

此外你还可以通过`precommit`钩子来检测你的每次`git commit`，
注意这里只对Linux/OS X系统有效，因为用到了`awk`命令。

```
"precommit": "eslint `git diff HEAD --name-only --diff-filter=ACMRTU | awk '/\\.(js|jsx)$/' | awk -v path=$(git rev-parse --show-toplevel)/ '{print path$1}' ORS=' '`"
```

## 测试

对于React社区而言，发展了很多优秀的库。因此，在开发过程中，我们建议尽可能的编写测试，并努力的提高测试的覆盖率。

### 组件测试

这里推荐使用AirBnb所开发的[enzyme](https://github.com/airbnb/enzyme)，可以用于组件测试。直接看官方的代码：

```jsx
import React from 'react';
import { shallow } from 'enzyme';
import sinon from 'sinon';

describe('<MyComponent />', () => {

  it('renders three <Foo /> components', () => {
    const wrapper = shallow(<MyComponent />);
    expect(wrapper.find(Foo)).to.have.length(3);
  });

  it('renders an `.icon-star`', () => {
    const wrapper = shallow(<MyComponent />);
    expect(wrapper.find('.icon-star')).to.have.length(1);
  });

  it('renders children when passed in', () => {
    const wrapper = shallow(
      <MyComponent>
        <div className="unique" />
      </MyComponent>
    );
    expect(wrapper.contains(<div className="unique" />)).to.equal(true);
  });

  it('simulates click events', () => {
    const onButtonClick = sinon.spy();
    const wrapper = shallow(
      <Foo onButtonClick={onButtonClick} />
    );
    wrapper.find('button').simulate('click');
    expect(onButtonClick.calledOnce).to.equal(true);
  });

});
```

详细内容可以参考Enzyme的[官方文档](http://airbnb.io/enzyme/index.html)。

## 其他主题

还有很多主题本文没有涉及到，包括Redux，Router，Immutable State等内容。在后续，
对这些主题会进行分享。

## 小结

本文主要介绍如何利用seu快速的上手基于ES6的React的开发，并介绍使用ESLint来规范化代码的编写，
以及实用Enzyme来编写React的组件测试。
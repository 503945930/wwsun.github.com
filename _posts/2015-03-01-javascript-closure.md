---
layout: post
title: 详解JavaScript的函数闭包
category: technique
---

在JavaScript总，实现外部作用域访问内部作用域中变量的方法叫做闭包（Closure）。
也可以认为闭包是指函数有自由独立的变量（闭包是一种特殊的对象，其包含函数和创建该函数的环境）。
换句话说，定义在闭包中的函数可以“记忆”它创建时候的环境。而创建闭包的最常见的方式，是在函数内部创建另一个函数。

<!--more-->

> A closure is a function plus the connection to the variables of its surrounding scopes.

## 词法作用域

处于种种原因，有时候我们需要得到函数内部的局部变量。通常情况下，这是无法做到的，只有通过变通方法才能实现。
我们需要在函数内部再定义一个函数。比如有下面这个[例子（在JSFIDDLE查看）](http://jsfiddle.net/xAFs9/3/)：

	function init() {
	    var name = "Mozilla"; // name is a local variable created by init
	    
		// 内部函数，它是一个闭包
		function displayName() {
	        alert (name); // displayName() uses variable declared in the parent function    
	    }
	    displayName();    
	}
	init();

上面的代码中，`displayName`是`init`的内函数，可以在`displayName`中访问`init`的局部变量，但反过来却不可以。
也就是`displayName`内部的局部变量对`init`是不可见的。这就是Javascript语言特有的"链式作用域"结构（chain scope），
子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。

简单的说，内函数`displayName()`之所以能访问外函数中的`name`变量是因为JavaScript中的词法作用域的作用：
在 JavaScript 中，变量的作用域是由它在源代码中所处位置决定的（词法），并且嵌套的函数可以访问到其外层作用域中声明的变量。

## 闭包

既然`displayName`可以访问`init`的局部变量，那么只要把`displayName`返回，
我们不就可以在`init`的外部读取到它的内部变量了吗！

现在考虑下面这个[例子](http://jsfiddle.net/wwsun/tyyqjn4y/)：

	function makeFunc() {
		var name = "Mozilla";
		
		// 内函数可以访问外部函数的局部变量
		function displayName() {
			alert(name);
		}
		return displayName; // 返回内部函数
	}
	
	var myFunc = makeFunc(); // myFunc成为了闭包
	myFunc();

代码的运行结果并没有变，所不同的是，内函数`displayName()`在执行之前被外函数返回了。

这段代码看起来别扭却能正常运行。通常，函数中的局部变量仅在函数的执行期间可用。
一旦 `makeFunc()` 执行过后，我们会很合理的认为 `name` 变量将不再可用。虽然代码运行的没问题，但实际并非如此。

对这一问题的解释就是，`myFunc`变成成一个**闭包**了。闭包是一种特殊的对象，它由两部分构成：
函数，以及创建该函数的环境。环境由闭包创建时在作用域中的任何局部变量组成。
在我们的例子中，`myFunc` 是一个闭包，由 `displayName` 函数和闭包创建时存在的 "Mozilla" 字符串形成。

你可以简单的理解为：闭包就是能够读取其他函数内部变量的函数。而闭包通常是“定义在函数内部的函数”。
我们可以认为，闭包充当了函数内部和函数外部连接起来的桥梁。

### 闭包用途

1. 读取函数内部的变量
1. 让这些变量的值始终保存在内存中

注意，因为闭包会使得函数中变量都保存在内存中，因此不能滥用闭包，否则会造成内存溢出。

### 作用域链

之所以能访问这个变量，是因为内部函数的作用域链中包含了外函数`makeFunc()`的作用域。
要搞清楚其细节，就有必要了解作用域链（Scope Chain）的相关知识：
当某个函数第一次被调用时，会创建一个执行环境及相应的作用域链，
并把作用域链赋值给一个特殊的内部属性（即[scope]）。然后，
使用`this`、`arguments`和其他命名参数的值来初始化函数的活动对象。
但在作用域链中，外部函数的活动对象始终处于第二位，
外部函数的外部函数的活动对象处于第三位...直至作为作用域链终点的全局执行环境。

下面是一个[更有趣的例子](http://jsfiddle.net/wwsun/kzmLLrg3/)，一个`makeAddr`函数：

	function makeAddr(x) {
	    return function(y) {
	        return x+y;    
	    };
	}
	
	// add5和add10都是闭包
	var add5 = makeAddr(5);
	var add10 = makeAddr(10);
	
	alert(add5(2)); // 7
	alert(add10(2)); // 12

在这个例子中，我们定义了函数`makeAddr(x)`, 其接收唯一的参数x，并且返回了一个新的函数。
被返回的函数也接收唯一的参数y，并返回x和y的和。

本质上，`makeAddr`可以认为是一个函数工厂——它创建了一个函数族，可以通过指定参数用来增加一个指定值。
在上面的例子中，我们使用这个函数工厂创建了两个新的函数，分别为add5和add10，一个用来增加5，另一个用来增加10。

`add5`和`add10`都是闭包。它们共享相同的函数体定义，但是存放了不同的环境。
在add5的环境中，x的值为5, 而在add10的环境中，x的值为10。

## 闭包实战

上面我们已经讲完了闭包的理论部分，下面我们试着从实战角度看看闭包的具体应用。
闭包允许将函数与其所操作的某些数据（环境）关连起来。这显然类似于面向对象编程。
在面对象编程中，对象允许我们将某些数据（对象的属性）与一个或者多个方法相关联。

因而，一般说来，可以使用只有一个方法的对象的地方，都可以使用闭包。

在 Web 中，您可能想这样做的情形非常普遍。大部分我们所写的 Web JavaScript 代码都是事件驱动的 — 
定义某种行为，然后将其添加到用户触发的事件之上（比如点击或者按键）。我们的代码通常添加为回调：
响应事件而执行的函数。

[这个例子](http://jsfiddle.net/vnkuZ/)是这样的：假设我们想通过增加一些按钮来调整页面字体的大小。
一种方法是通过指定body元素在像素单位下的font-size，然后设置页面中其他元素使用`em`作为单位。

CSS代码如下：

	body {
	  font-family: Helvetica, Arial, sans-serif;
	  font-size: 12px;
	}
	
	h1 {
	  font-size: 1.5em;
	}
	h2 {
	  font-size: 1.2em;
	}

我们通过点击按钮来改变body元素的字体大小，因为页面其他元素使用了相对单位`em`，
因此字体大小会相应的被调整。JavaScript代码如下：

	function makeSizer(size) {
	  return function() {
	    document.body.style.fontSize = size + 'px';
	  };
	}
	
	var size12 = makeSizer(12);
	var size14 = makeSizer(14);
	var size16 = makeSizer(16);
	
函数`size12`, `size14`, `size16`分别用来调整body文本的大小为12，14， 16像素。
然后，我们可以将其与按钮元素绑定，如下：

	document.getElementById('size-12').onclick = size12;
	document.getElementById('size-14').onclick = size14;
	document.getElementById('size-16').onclick = size16;

html代码如下：

    <p>Some paragraph text</p>
    <h1>some heading 1 text</h1>
    <h2>some heading 2 text</h2>

    <a href="#" id="size-12">12</a>
    <a href="#" id="size-14">14</a>
    <a href="#" id="size-16">16</a>

## 使用闭包来仿造私有方法

在Java中你可以声明私有方法，私有方法表示只能在当前类中使调用该方法。
但是JavaScript并没有提供一个直接的方法去声明私有方法，但是可以使用闭包来仿造私有方法。
私有方法的作用不仅仅在于可以有效的限制代码的访问，
也为管理你的全局命名空间提供了一种强有力的方式，使得无关的方法不被公开的暴露出来。

下面来看如何定义可以访问私有函数和变量的公共函数，我们使用闭包来达到这一目的，
这也被称为[模块模式](https://www.google.com.hk/search?q=javascript+module+pattern&gws_rd=cr)：

	var counter = (function() {
		var privateCounter = 0;
		function changeBy(val) {
			privateCounter += val;
		}
		
		return {
			increment: function() {
				changeBy(1);
			},
			decrement: function() {
				changeBy(-1);
			},
			value: function() {
				return privateCounter;
			}
		};
	})();
	
	alert(counter.value()); /* Alerts 0 */
	counter.increment();
	counter.increment();
	alert(counter.value()); /* Alerts 2 */
	counter.decrement();
	alert(counter.value()); /* Alerts 1 */

这里有好多细节。在以往的示例中，每个闭包都有它自己的环境；而这次我们只创建了一个环境，
为三个函数所共享，分别是：`counter.increment`, `counter.decrement`, `counter.value`。

被共享的环境是在一个匿名函数的函数体内被创建的，这将会在其被定义后立即执行（立即执行函数 IIFE）。
这个共享环境包含两个私有项：变量`privateCounter`和函数`changeBy`，
外界无法直接访问匿名函数内部的这两个私有项。取而代之的是，
只能通过访问三个公开的接口函数，也就是被匿名函数返回的三个函数。

这三个公共函数是共享同一个环境的闭包，而它们之所以都可以访问变量`privateCounter`和函数`changeBy`，
是因为JavaScript词法作用域的作用。

您应该注意到了，我们定义了一个匿名函数用于创建计数器，然后直接调用该函数，并将返回值赋给 `Counter` 变量。
也可以将这个函数保存到另一个变量中，以便创建多个计数器。

	var counter1 = makeCounter();
	var counter2 = makeCounter();
	alert(counter1.value()); /* Alerts 0 */
	counter1.increment();
	counter1.increment();
	alert(counter1.value()); /* Alerts 2 */
	counter1.decrement();
	alert(counter1.value()); /* Alerts 1 */
	alert(counter2.value()); /* Alerts 0 */

可以发现，这两个计数器是彼此独立的，它们的环境在函数`makeCounter()`在调用期间是彼此不同的。
闭包变量`privateCounter`在每一次包含不同的实例。

利用这种方式使用闭包可以向OOP编程一样提供非常多的好处，尤其是数据隐藏和封装。

### 立即执行函数 IIFE

通过IIFE这种方式我们可以构造块作用域，通常的模式为：

	(function () {
		var tmp = ...; // 这里的tmp就是个局部变量
		
			
	}());
	
IIFE是一种函数表达式，它在被定义后立即被调用。而在函数内部定义的变量自然是局部变量。

## 在循环中创建闭包：一个常常会犯的错误

在ES6引入`let`关键字之前，闭包的一个很常见的问题发生在循环中创建闭包。
考虑下面[这个例子](http://jsfiddle.net/v7gjv/)：

HTML代码如下：

	<p id="help">Helpful notes will appear here</p>
	<p>E-mail: <input type="text" id="email" name="email"></p>
	<p>Name: <input type="text" id="name" name="name"></p>
	<p>Age: <input type="text" id="age" name="age"></p>

JavaScript代码如下：

	function showHelp(help) {
	  document.getElementById('help').innerHTML = help;
	}
	
	function setupHelp() {
	  var helpText = [
	      {'id': 'email', 'help': 'Your e-mail address'},
	      {'id': 'name', 'help': 'Your full name'},
	      {'id': 'age', 'help': 'Your age (you must be over 16)'}
	    ];
	
	  for (var i = 0; i < helpText.length; i++) {
	    var item = helpText[i];
	    document.getElementById(item.id).onfocus = function() {
	      showHelp(item.help);
	    }
	  }
	}
	
	setupHelp();

在`helpText`数组中我们定义了三个提示信息，每个都使用ID与html文档中的输入域相关联。
通过循环来迭代这些提示信息，将其分别与三个输入框的`onfocus`事件绑定，以在用户聚焦在不同的输入框时提示不同的信息。

如果你试着运行这些代码，会发现其并不会正常工作，无论你的焦点在哪个输入框中，提示都是关于年龄的那条信息。

该问题的原因在于赋给 `onfocus` 是闭包（showHelp）中的匿名函数而不是闭包对象；
在闭包（showHelp）中一共创建了三个匿名函数，但是它们都共享同一个环境（item）。
在 onfocus 的回调被执行时，循环早已经完成，且此时 item 变量（由所有三个闭包所共享）已经指向了 helpText 列表中的最后一项。

> 闭包只能取得包含函数中任何变量的最后一个值！闭包保存的是整个变量对象，而不是某个特殊的变量。

解决这个问题的一种方案是使`onfocus`指向一个新的闭包对象。

	function showHelp(help) {
	  document.getElementById('help').innerHTML = help;
	}
	
	function makeHelpCallback(help) {
	    return function() {
	        showHelp(help);   
	    }
	}
	
	function setupHelp() {
	  var helpText = [
	      {'id': 'email', 'help': 'Your e-mail address'},
	      {'id': 'name', 'help': 'Your full name'},
	      {'id': 'age', 'help': 'Your age (you must be over 16)'}
	    ];
	
	  for (var i = 0; i < helpText.length; i++) {
	    var item = helpText[i];
	    document.getElementById(item.id).onfocus = makeHelpCallback(item.help);
	  }
	}
	
	setupHelp();


这段代码可以如我们所期望的那样工作。所有的回调不再共享同一个环境， 
`makeHelpCallback` 函数为每一个回调创建一个新的环境
。在这些环境中，`help` 指向 `helpText` 数组中对应的字符串。
[完整代码](http://jsfiddle.net/wwsun/grp341z3/)。

## 性能问题

由于闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存，
过多使用闭包会使内存被占用过多，因此需要慎重使用闭包。。

例如，在创建新的对象或者类时，方法通常应该关联于对象的原型，而不是定义到对象的构造器中。
原因是这将导致每次构造器被调用，方法都会被重新赋值一次（也就是说，为每一个对象的创建）。

可以考虑一下下面的这个例子：

	function MyObject(name, message) {
	  this.name = name.toString();
	  this.message = message.toString();
	  this.getName = function() {
	    return this.name;
	  };
	
	  this.getMessage = function() {
	    return this.message;
	  };
	}

这里的代码中并没有得到任何有关闭包的好处，因此可以重构为：

	function MyObject(name, message) {
	  this.name = name.toString();
	  this.message = message.toString();
	}
	MyObject.prototype = {
	  getName: function() {
	    return this.name;
	  },
	  getMessage: function() {
	    return this.message;
	  }
	};

但是，重新定义原型显然也不是什么好方法，并且也是不被推荐的。
因此，更好的做法是将方法追加到已有原型上：

	function MyObject(name, message) {
	  this.name = name.toString();
	  this.message = message.toString();
	}
	MyObject.prototype.getName = function() {
	  return this.name;
	};
	MyObject.prototype.getMessage = function() {
	  return this.message;
	};

在前面的例子中，被继承的原型可以被所有对象所贡献，并且方法定义也无需在每个对象创建时重新书写。
你可以参考JavaScirpt的[对象模型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Details_of_the_Object_Model)了解更多的内容。

###References

1. http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html
1. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures
1. JavaScript高级程序设计，第3版，第7章
1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Closures
1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Details_of_the_Object_Model
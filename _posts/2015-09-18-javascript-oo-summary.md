---
layout: post
title: JavaScript面向对象编程实践
category: technique
---

面向对象编程是用抽象方式创建基于现实世界模型的一种编程模式。主要包括模块化、多态、和封装几种技术。
JavaScript 的核心是支持面向对象的，同时它也提供了强大灵活的面向对象编程（OOP）语言能力。
本文简要介绍了使用`ECMAScript 5`进行面向对象编程的基本知识，以及实现对象创建和对象继承的常用方法。

<!--more-->

## 原型

我们创建的每个对象都有一个prototype属性，这个属性是一个指针，指向一个对象，
而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。也就是说，
prototype就是通过调用构造而创建的那个对象实例的原型对象。
使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。

基于原型的编程不是面向对象编程中体现的风格，且行为重用（在基于类的语言中也称为继承）
是通过装饰它作为原型的现有对象的过程实现的。这种模式也被称为弱类化，原型化，或基于实例的编程。

## 类

JavaScript是一种基于原型的语言，在ES6之前，它没有类的声明语句。通常做法是，使用函数声明作为类的声明。
定义一个类和定义一个函数一样简单。例如我们要定义一个新类`Person`。

	function Person() { } // 注意区别于函数定义，类名需要大写

## 创建对象（类的实例）

在JavaScript中最常用的创建对象的方式为组合使用构造函数模式和原型模式。使用构造函数定义实例属性，
而原型模式用于定义方法和共享的属性。结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，
最大限度的节省了内存。此外，你还可以向构造函数传递参数。

在实例化时构造器被带调用。构造器是对象中的一个方法。在JavaScript中函数就可以作为构造器使用，
因此不需要特别定义一个构造器方法。构造器常用于给对象的属性赋值或者为调用函数做准备。

下面我们来完善`Person`类的定义，首先我们使用构造函数定义实例属性如下：

	function Person(name, age, job){
		this.name = name;
		this.age = age;
		this.job = job;
	}


上面我们为`Person`定义了三个属性：`name`,`age`,`job`。

为了能够进一步完善Person对象，可以使用原型模式为其添加实例方法`sayName()`：

	Person.prototype = {
		constructor: Person,
		sayName : function () {
			console.log(this.name);
		}
	};

`Person.prototype`是一个可以被`Person`的所有实例共享的对象。它是一个名叫原型链（Prototype Chain）的查询链的一部分：
当你试图访问一个`Person`没有定义的属性时，解释器会首先检查这个`Person.prototype`来判断是否存在这样一个属性。
所以，任何分配给`Person.prototype`的东西对通过`this`对象构造的实例都是可用的。

方法与属性很相似， 不同的是：一个是函数，另一个可以被定义为函数。 调用方法很像存取一个属性。
为定义一个方法, 需要将一个函数赋值给类的 prototype 属性; 这个赋值给函数的名称就是用来给对象在外部调用它使用的。

上面我们完成了对`Person`对象的基本定义，其包括三个实例属性和一个实例方法。我们可以使用`new obj`创建`obj`的新实例。
我们来创建`Person`类的两个实例：

	var person1 = new Person("Weiwei SUN", 25, "Software Engineer");
	var person2 = new Person("Lily", 21, "student");
	
	console.log(person1.sayName()); // Weiwei SUN
	console.log(person2.sayName()); // Lily
	console.log(person1.sayName === person2.sayName); //true

注意上面的输出结果，`person1.sayName` 和 `person2.sayName` 引用了相同的函数。在调用函数的过程中，
`this`的值取决于我们怎么样调用函数。 在通常情况下，我们通过一个表达式person1.sayHello()来调用函数：
即从一个对象的属性中得到所调用的函数。此时this被设置为我们取得函数的对象（即person1）。
这就是为什么`person1.sayName()`输出为`Weiwei SUN`，而`person2.sayName()`输出为`Lily`的原因。 

## 继承

创建一个或多个类的特定版本类的方式被称为继承（JavaScript只支持单继承）。在JavaScript中，
继承通过赋予子类一个父类的实例并定制化子类来实现。在ES5中，我们通常用`Object.create`方法来实现继承。

	// 定义Student的构造器
	function Student(name, subject) {
		// 调用父类构造器，确保`this`在调用过程中设置正确
		Person.call(this, name);
		this.subject = subject;
	}
	
	// 实现继承：继承自Person.prototype
	Student.prototype = Object.create(Person.prototype);
	Student.prototype.constructor = Student;
	
	// 为Student加入新的实例方法
	Student.prototype.sayHello = function () {
		console.log("Hello, I'm " + this.name + ". I'm studying " + this.subject + ".");
	};
	
	// 测试用例
	var s1 = new Student('Lulu', 'Math');
	s1.sayName(); // Lulu
	s1.sayHello(); // Hello, I'm Lulu. I'm studying Math.
	
	console.log(s1 instanceof Student); // true
	console.log(s1 instanceof Person); // true

## 封装

在上一个例子中，Student类虽然不需要知道`Person`类的`sayName()`方法是如何实现的，
但是仍然可以使用这个方法；`Student`类不需要明确地定义这个方法，除非我们想改变它。 
这就叫做封装，对于所有继承自父类的方法，只需要在子类中定义那些你想改变的即可。

## 抽象

抽象是允许模拟所要解决的问题中的通用部分的一种机制。这可以通过继承（具体化）或组合来实现。
JavaScript通过继承实现具体化，通过让类的实例是其他对象的属性值来实现组合。

JavaScript Function 类继承自Object类（这是典型的具体化） 。
`Function.prototype`的属性是一个Object实例（这是典型的组合）。

## References

1. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript
2. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create
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

## 类和对象

JavaScript是一种基于原型的语言，在ES6之前我们在JavaScript创建对象的方法都是创建一个构造函数，
然后为函数的原型对象添加方法。

### 混合使用构造函数和原型

使用构造函数定义实例属性，而原型模式用于定义方法和共享的属性。结果，每个实例都会有自己的一份实例属性的副本，
但同时又共享着对方法的引用，最大限度的节省了内存。此外，你还可以向构造函数传递参数。

在实例化时构造器被带调用。构造器是对象中的一个方法。在JavaScript中函数就可以作为构造器使用，
因此不需要特别定义一个构造器方法。构造器常用于给对象的属性赋值或者为调用函数做准备。

下面我们为`Animal`类定义了一个属性`name`。

```javascript
function Animal() {
  this.name = name;
}

Animal.prototype.getName = function () {
  return this.name;
}
```

也就是说，在JavaScript中，类的声明就是函数的声明，定义一个类也就是定一个函数。
不过，我们类函数通常首字母为大写。例如上面我们定义的`Animal`。

而通过ES6定义一个类则要简单的多，使用`class`关键字即可：

```javascript
class Animal {
  constructor () {
    this.name = name;
  }
  
  getName () {
    return this.name;
  }
```

为了能够进一步完善`Animal`类，我们使用了原型模式为其添加实例方法`getName()`。

`Animal.prototype`是一个可以被`Animal`的所有实例共享的对象。它是一个名叫原型链（Prototype Chain）的查询链的一部分：
当你试图访问一个`Animal`没有定义的属性时，解释器会首先检查这个`Animal.prototype`来判断是否存在这样一个属性。
所以，任何分配给`Animal.prototype`的东西对通过`this`对象构造的实例都是可用的。

方法与属性很相似， 不同的是：一个是函数，另一个可以被定义为函数。 调用方法很像存取一个属性。
为定义一个方法, 需要将一个函数赋值给类的`prototype`属性; 这个赋值给函数的名称就是用来给对象在外部调用它使用的。

## 继承

在JavaScript中，对于子类的解决方案是，创建一个新的构造函数，并且设置其原型为其父类的原型。
调用父类的构造函数，并将`this`设置为其上下文对象。注意，在JavaScript中只支持单继承。
在JavaScript中，继承通过赋予子类一个父类的实例并定制化子类来实现。在ES5中，
我们通常用`Object.create`方法来实现继承。

```javascript
// 定义Dog构造器
function Dog(name) {
  Animal.call(this, name); // 调用父类构造器，重新设置绑定`this`值
}

// 实现继承
Dog.prototype = Object.create(Animal.prototype);

// 为子类添加新的方法
Dog.prototype.speak = function () {
  return "woof";
};

var dog = new Dog("Scamp");
console.log(dog.getName() + ' says ' + dog.speak());
```    

使用ES6实现类的继承则要简单的多，上面的代码可以改写如下：

```javascript
class Dog extends Animal {
  constructor(name) {
    super(name);
  }
  
  speak() {
    return "woof";
  }
}

var dog = new Dog("Scamp");
console.log(dog.getName() + ' says ' + dog.speak());
```

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
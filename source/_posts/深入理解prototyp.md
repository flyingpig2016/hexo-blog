---
title: 深入理解prototyp
toc: true
comments: true
date: 2017-07-10 13:40:38
tags: javascript高级程序设计
categories:
---
对JavaScript中原型模式的理解
-------
### 一：什么是原型对象？有什么优点？**
&emsp;&emsp; 简单的来说，无论何时，我们创建的每一个函数都有一个prototype属性，这个属性是一个指针，指向一个对象，这个对象包含了通过调用该构造函数所创建的对象共享的属性和方法。其实我们平常的叫法就是指：**prototype就是通过该构造函数创建的某个实例的原型对象**，但是其实prototype是每个构造函数的属性而已，只能说万物皆对象罢了。
&emsp;&emsp; 原型对象的优点是：所有的对象实例都可以共享它包含的属性和方法。这一点可以在构造函数里就可以看出来，因为构造函数在函数里面就定义了对象的实例信息，而原型对象可以在任何地方定义属性和方法。例如：
```
function Person(){}
Person.prototype.name = 'bangbang';
Person.prototype.age = 18;
Person.prototype.job = 'programmer';
Person.prototype.dream = function(){
	console.log('Change yourself');
}

var person1 = new Person();
person1.dream();	//Change yourself

var person2 = new Person();
person2.dream();
//判断两个实例继承的方法和属性是否全等
console.log(person1.dream === person2.dream);
console.log(person1.age === person2.age);
```
由以上的代码可以看出它比构造函数方法的好处就是，person1和person2访问的都是同一个dream()函数，即他们的属性和方法都是共享的。
<!--more-->
### 二：深入理解原型对象和prototype？**
&emsp;&emsp;由上面可知，我们创建了一个函数，就会根据ECMAscript特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个constructor(构造函数)属性，这个属性是一个指向prototype属性所在函数的指针。就像前面的例子，Person.prototype.constructor指向Person，而通过这个构造函数，我们还可以继续为原型对象添加其他属性和方法。
&emsp;&emsp;下面，我们就开始从原理上说明上面代码运行机制的：
1. 创建一个自定义的构造函数后，它的原型对象默认只会有constructor属性，其他属性和方法都是从Object继承而来的；
2.  当调用构造函数创建一个新实例后，该实例的内部将包含一个指针，指向构造函数的原型对象。ECMA5中称这个指针叫[[prototype]]。虽然在脚本中没有标准的方式访问[[prototype]]，但Firefox、Safari和Chrome在每个对象上都支持一个属性_proto_;而在其他浏览器中，这个属性则是完全不可见的。不过要明确的真正一点就是，这个连接存在于 **实例** 和 **构造函数的原型对象** 之间，而不是存在于实例和构造函数之间，用前面使用Person构造函数和Person.prototype创建实例的代码为例，各个对象之间的关系如下图所示：
![这里写图片描述](http://img.blog.csdn.net/20170514170752175?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmx5aW5ncGlnMjAxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
&emsp;&emsp;上图展示了Person构造函数、Person的原型以及Person现有的两个实例之间的关系，再次，Person.prototype指向了原型对象，而Person.prototype.constructor又指回了Person。原型对象中除了包含constructor属性之外，还包括后来添加的其他属性。Person的每个实例——person1和person2都包含一个内部属性，该属性指向了Person.prototype；换句话说，他们与构造函数没有直接关系。
&emsp;&emsp;注意：虽然这两个实例都不包含属性和方法，但我们却可以调用name、age、job、dream()方法和属性，这是通过查找对象属性的过程来实现的。
&emsp;&emsp;虽然我们不能访问到[[prototype]]，但可以通过isPrototypeOf()方法来确定对象之间是否存在这种关系。因为这两个实例都有一个指向Person.prototype的指针，因此，都返回true如下所示：
```
console.log(Person.prototype.isPrototypeOf(person1));//true
console.log(Person.prototype.isPrototypeOf(person2));//true
```
ECMA5增加了一个方法，叫Object.getPrototypeOf(),这个方法返回[[Prototype]]的值,也就是括号里面对象的原型，例如：
```
console.log(Object.getPrototypeOf(person1) == Person.prototype; // true
console.log(Object.getPrototypeOf(person1).name); //'bangbang'
```
用hasOwnProperty()方法可以检测一个属性是存在于实例中还是原型中。还是上面的例子：

```
console.log(person1.hasOwnProperty('name'); //false
person1.name = 'qiqi';
console.log(person1.hasOwnProperty('name'); //true —— 来自实例
console.log(person1.name);  //'qiqi' ——来自实例
delete person1.name;
console.log(person1.name); //'bangbang' —— 来自原型
console.log(person1.hasOwnProperty('name'); //true ——来自原型
```
当我们为person1添加name属性的时候，如下图图所示：
![这里写图片描述](http://img.blog.csdn.net/20170514170806065?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmx5aW5ncGlnMjAxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
### 三：组合使用构造函数模式和原型模式优点最多：**
&emsp;&emsp;以上我讲述了用构造函数生成对象的原理以及生成的对象和原型对象之间的关系，原理了解了这么多，我么当然需要知道怎么在合适的时候用他们。创建自定义类型的最常见方式就是组合使用构造函数模式和原型模式。构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。每个实例都会有自己的一份实例属性的副本，但同时又共享着对方发的引用，最大限度的节省了内存。如下例所示：
```
		function Person(name,age,job){
			this.name = name;
			this.age = age;
			this.job = job;
			this.love = ['qiqi','lili'];
		}
		Person.prototype = {
			constructor : Person,
			dream : function(){
				console.log(this.love[0]);
			}
		}
		var person1 = new Person('bangbang',18,'programmer');
		var person2 = new Person('xiaoya',18,'teacher');
		console.log(person1.love);
		console.log(person2.love)
		console.log(person1.love === person2.love);  //false
		//给person1的love属性添加元素
		person1.love.push('niuniu');
		console.log(person1.love)  //["qiqi", "lili", "niuniu"]
		console.log(person2.love)  //["qiqi", "lili"]
		console.log(person1.love === person2.love);  //false
		console.log(person1.dream === person2.dream);//true
```
以上这种模式是ECMAscript中使用最广泛的，认同度最高的，优点最多的创建自定义类型的方法，也就是创建JavaScript中类的方法。

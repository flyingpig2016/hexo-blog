---
title: js工厂模式、构造函数以及他们创建对象的优缺点
toc: true
comments: true
date: 2017-07-10 13:44:07
tags: javascript高级程序设计
categories:
---
### 工厂模式和构造函数的区别
> 学习JavaScript有一段时间了，但是随着工作的进展，我发现，JavaScript绝对不是写一下简单效果那么肤浅，是时候完全，透彻地理解他的精髓了。

好了，让俺老猪带你装逼带你飞吧！
### 一：为什么使用工厂模式和构造函数？**
&emsp;&emsp;虽然Object构造函数或者字面量(var person={})可以创建单个对象，但是，他有一个明显的缺点，那就是：重复造轮子，产生大量的重复性代码，为了解决这个问题，我们就开始使用工厂模式和构造函数模式。
### 二：什么是工厂模式？它有什么优缺点？**
&emsp;&emsp;1.顾名思义，工厂模式就是像工厂一样来创建对象。但这样的解释似乎有点欠妥，高大上一点，工厂模式其实是软件领域中一种广为人知的一种设计模式，这种模式抽象了创建具体对象的过程。开发人员发明了一种函数，用函数来大量创建对象的方法；
<!--more-->
2.工厂模式例子如下：
```
function createPerson(name,age,say){
	var obj1 = new Object();
	obj1.name = name;
	obj.age = age;
	obj.say = say;
	obj.should = function(){
		alert(this.say);
	}
	return obj;
}
var person1 = createPerson('bangbang',18,'脚踏实地才能壮志凌云！');
var person2 = createPerson('yanniu',18,'我相信你可以！');
```
&emsp;&emsp;但是工厂模式可以创建多个相似的对象，但是却没有解决对象的识别问题(不知道这个对象的类型)。相信这句话还需要探究，不要急，接下来看那看构造函数模式，你就会理解。
### 三：什么是构造函数模式，它和工厂模式比较有哪些好处？它自身又有那些缺点？**
&emsp;&emsp;1.实例如下：
```
function CreatePerson(name, age, say){
	this.name = name;
	this.age = age;
	this.say = say;
	this.should = function(){
		console(this.say);
	}
}
var person1 = new CreatePerson('bangbang',18,'修身，齐家，天下很太平');
var person2 = new CreatePerson('yanniu',18,'吃地沟油的命，操中南海的心');
```
&emsp;&emsp;这个例子中，CreatePerson()函数取代了createPerson()函数；睁大你的两个招子，第二个CreatePerson()明显是开头大写，这是构造函数的惯例，为了区分和普通函数的区别，因为构造函数也是函数，只不过可以创建对象而已。和工厂模式不同之处看最后一点，不要着急，先自己感悟一下，先说出来就没有味道了，嘿嘿！
2.要创建CreatePerson()的新实例，必须使用new，即和使用var person1  = new Object()差不多，实际上会经历一些步骤：

  * 创建一个新的对象；
  * 将构造函数作用域赋给新的对象（即this指向新对象）；
  * 执行构造函数里面的代码；
  * 返回新的对象。
  但是创建的person1和person2都有一个constructor(构造函数)属性，该属性指向CreatePerson,代码如下：
```
console.log(person1.constructor == CreatePerson());//true
console.log(person2.constructor == CreatePerson());//true
```
&emsp;&emsp;不过用instanceof操作符来检测对象的类型还是更可靠地，因为instanceof是用来专门比较一个对象是否为某个构造函数的实例。
```
console.log(person1 instanceof Object); 	 //true;
console.log(person1 instanceof CreatePerson);//true;
console.log(person2 instanceof Object);      //true;
console.log(person2 instanceof CreatePerson);//true;
```
&emsp;&emsp;创建自定义的构造函数意味着将来可以将他的实例标识为一种特定的类型，这就是构造函数比工厂模式强的地方，因为它只道自己从哪里来，通过谁产生的，在回顾上面（一）我所写的，你就会很明白了。
### 四:构造函数和普通函数的区别在哪？**
&emsp;&emsp;1.构造函数和普通函数唯一的区别是他们的调用方式不同。
&emsp;&emsp;2.任何函数只要通过new来调用，那它就可以作为构造函数，而任何函数，如果不通过new操作符来调用，他就是普通的函数。
&emsp;&emsp;例子如下：
```
//当做构造函数来使用
var person1 = new CreatePerson('bangbang',18,'修身，齐家，天下很太平');
person1.should(); //'修身，齐家，天下很太平'

//构造函数当做普通函数使用
CreatePerson('bangbang',18,'修身，齐家，天下很太平')
window.should();//'修身，齐家，天下很太平'
//在另一个对象的作用域中调用（这个需要以后的继承探讨）
var  person3 = new Object():
CreatePerson.call(person3,'qiqi',18,'我是一个护士');//person3就有了CreatePerson类的所有的属性和should方法
```
### 五：构造函数有哪些不足之处？**
&emsp;&emsp;1.构造函数最明显的缺点就是，每个方法都要在每个实例上重新创建一遍。在前面的例子中，person1和person2都有一个should的方法，但是两个方法不是同一个Function的实例，因此每定义一个函数(也就是实例化一个对象),从逻辑上构造函数还可以这样定义：
```
function CreatePerson(name,age,say){
	this.name = name;
	this.age = age;
	this.say = say;
	this.should = new Function('console.log(this.say)');
}
```
&emsp;&emsp;由上面可以得出这种方式创建函数实例化出的对象都包含一个不同的Function实例，说的更通俗点：这种方式创建函数，会导致不同的作用域链和标识符解析，但是创建Function的新实例的机制仍然是相同的。所以，不同实例上的同名函数是不相等的，证明如下：
```
console.log(person1.should == person2.should);//false
```
2.因此以上的写法可以这样简化：
```
function CreatePerson(name,age,say){
	this.name = name;
	this.age = age;
	this.say = say;
	this.should = should;
}
function should(){
	console.log(this.say);
}
var person1 = new CreatePerson('bangbang',18,'乐观');
var person2 = new CreatePerson('yanniu',18,'切记盲目乐观');
```
&emsp;&emsp;我们把函数放在构造函数的外面作为全局函数，因此person1和person2对象就共享了在全局作用域中定义的同一个say函数。但是问题又来了：**在全局作用域中定义的函数实际上只能被某个对象调用，这让该全局作用域有点名不副实。**，更让然无法接受的是：如果对象需要定义很多方法，那么就要定义很多个全局函数，那么我们这个自定义的引用类型就丝毫没有封装性可言了。因此我们需要另一种高大上的模式，——原型模式（prototype）。


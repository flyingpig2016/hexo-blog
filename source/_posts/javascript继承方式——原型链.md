---
title: javascript继承方式——原型链
date: 2017-07-10 13:37:46
tags: javascript高级程序设计
toc : true     
comments: true
---
                 JavaScript中依靠原型链的继承方式
---------------------
### 开门见山，ECMAScript中实现继承主要是靠原型链来实现的。**
### 一：什么是原型链呢？**
&emsp;&emsp;其基本思想是利用原型，让一个引用类型继承另一个引用类型的属性和方法。根据上一节[原型对象](http://blog.csdn.net/flyingpig2016/article/details/53048394)：我们知道，每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。那么如果我们让原型对象等于另一个类型的实例，结果会怎样呢？很显然，此时的原型对象将包含一个指向另一个原型的指针，相应的，另一个原型中也包含一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系层层递进，就**构成了实例和原型的链条，这就是原型链**。以下就是原型链形成的基本模式：
<!--more-->
```
	//注意凡是this.-的，都是类的私有属性和方法，凡是-prototype.-的都是共有属性和方法

	//Father类型
	function Father(){
		this.firstName = 'xue';
		this.fatherSay = function(){
			return this.firstName;
		}
	}
	Father.prototype.hobby = 'game';
	Father.prototype.sayFirstName = function(){
		return this.firstName;
	}
	//创建Son类型
	function Son(){
		this.lastName = 'bangbang';    
		this.sonSay = function(){
			return this.lastName;
		}
	}
	//Son.prototype未被重写的时候，添加属性friend
	Son.prototype.friend = 'qiqi';
	console.log(Son.prototype)		//Object {friend: "qiqi", constructor: function}
	//Son.prototype未被重写的时候，实例化的对象son1
	var son1 = new Son();
	console.log(son1.friend);		//qiqi
	console.log(Object.getPrototypeOf(son1));	//Object {friend: "qiqi", constructor: function}

	//继承了father(),即重写了Son.prototype
	Son.prototype = new Father();
	Son.prototype.sayLastName = function(){
		return this.lastName;
	}
	//Son.prototype被重写后，实例化的对象son2
	var son2 = new Son();
	console.log(Object.getPrototypeOf(son2));//Father {firstName: "xue", fatherSay: function, sayLastName: function}
	console.log(son2.friend);		   //undifined : Son.prototype被指向了，所以son2中没有friend	
	console.log(son2.sayLastName());		   //bangbang : Son.prototype虽然被重指向了，但是sayLastName()是后来添加到prototype中的
	console.log(son2.firstName);	   //xue :证明了Son类继承了Father类的属性firstName
	console.log(son2.fatherSay());	   //xue :证明了Son类继承了Father类的方法fatherSay();
	console.log(son2.sayFirstName());  //xue ：证明了Son类继承了Father类的方法sayFirstName();
	console.log(son2.hobby);		   //game:证明了Son类继承了Father类的属性hobby;
```
&emsp;&emsp;以上代码定义了两个类,Father()和Son(),每个类型分别有一个属性和一个方法，主要区别是Son()继承了Father(),继承是通过创建Father的实例，并把它赋值给Son.prototype实现的。实现的本质就是重写原型对象，换句话说，原来存在于Father()的实例中的所有属性和方法，现在也存在于Son()中了。在确立了继承关系之后，我们给Son.prototype添加了一个新的方法，这样就在继承了Father()的属性和方法的基础上又添加了一个新的方法sayLastName()和属性friend  。上面Object.getPrototypeOf()函数用来返回实例的[[prototype]],即返回对象的原型。继承原理如下图所示：
![这里写图片描述](http://img.blog.csdn.net/20170515181542781?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmx5aW5ncGlnMjAxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
&emsp;&emsp;如上图所示，son1是Son.prototoype未被重新指向的时候实例化的对象，所以，它里面可访问的属性和方法有：friend,lastName,sonSay()；son2是Son.prototype已经被重新指向的时候实例化的对象，此时，Son类已经继承了Father类，所以，son2中可以访问的的属性和方法firstName，hobby，fatherSay()，sayFirstName(),最后我们在重指向Son.prototype后再给Son添加一个方法`Son.prototype.sayLastName = function(){return this.lastName;}`，然后son2也就具备了该方法。
&emsp;&emsp;我们开始总结原型搜索机制：当以读取模式访问一个实例属性时，首先会在实例中搜索该属性，如果没有找到该属性，就会继续搜索实例的原型。在通过原型链实现继承的情况下，搜索过程就得以沿着原型链继续向上。就拿上面的例子说，调用son2.sayFirstName()的时候，经过三个步骤：首先会搜索实例，然后搜索Son.prototype，最后一步才会找到父级原型中的sayFirstName()方法。
### 二：怎样确定原型和实例的关？**
&emsp;&emsp;我们可以通过两种方式确定原型和实例之间的关系。
&emsp;&emsp;第一种方式是使用instanceof操作符，只要用这个操作符来测试实例与原型链中出现的构造函数，结果就会返回true。
```
console.log(son2 instanceof Object);  //true
console.log(son2 instanceof Son);  //true
console.log(son1 instanceof Father);  //true
```
由于原型链的关系，我们可以说，son2是Object，Son，Father中任何一个类型的实例。
&emsp;&emsp;第二种方式是isPrototypeOf()方法，同样，只要是原型链中出现过的原型，都可以说是该原型链所派生的实例的类型。

```
console.log(Object.prototype.isPrototypeof(son2); //true;
console.log(Father.prototype.isPrototypeof(son2); // true;
console.log(Son.prototype.isPrototypeof(son2);   //ture;
```
### 三：原型链的问题？**
&emsp;&emsp;第一个问题：我们知道，包含引用类型值的原型属性可以被有实例共享，在通过原型来实现继承的时候，原型实际会变成另一个类型的实例，于是原先的实例属性也就变成了现在的原型属性了：
```
	function Father(){
		this.colors = ['red','blue','green'];
	}
	function Son(){
		this.names = 'bangbang';
	}
	//继承了Father();
	Son.prototype = new Father();
	
	var son1 = new Son();
	son1.colors.push('black');
	console.log(son1);
	
	var son2 = new Son();
	console.log(son2.colors);
```
&emsp;&emsp;当Son继承了Father之后，Son.prototype就变成了Father的一个实例，因此，它也拥有了一个它自己的colors属性，就跟创建了一个Son.prototype.colors属性一样。结果是Son的所有实例都会共享这一个colors属性，而我们对son1.colors的修改能够通过son2.colors反映出来。
原型链的第二个问题是：在创建子类型的实例时，不能向超类型的构造函数中传递参数。实际上，应该说是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。实践中，很少使用原型链。
### 四：借用构造函数继承（即：用call()和apply()方法继承）**
```
    function Father(){
        this.firstName = "xue";
        this.color = ["white","black"];
        this.sayFirstName = function(){
            console.log(this.firstName);
        }
    }
    Father.prototype.hobby = 'phone';
    Father.prototype.work = function(){
        console.log('work');
    }
    //继承
    function Son(){
        this.lastName = "bangbang";
        this.sayName = function(){
            console.log(this.lastName);
        }
        Father.call(this);
    }
    console.log(Son.prototype);
    
    var son1 = new Son();
    son1.color.push('black');
    console.log(son1.color);  //["white", "black", "black"]
    console.log(son1.hobby)   //undefiind//证明call不能继承Father的prototype中的属性
```
但是，这种借用构造函数的继承方式存在问题——方法都在构造函数中定义，因此函数复用就无从谈起了。而且，在超类型的原型(如：Father.prototype)中定义的方法，对子类型而言是不可见的，结果所有类型都只能使用构造函数模式。
**五：组合式继承（最常用的继承模式）**
&emsp;&emsp;指的是将**原型链**和**借用构造函数**的技术组合到一块。它们背后的思想是使用原型实现对**原型属性和方法**的继承，而通过借用构造函数来实现对**实例属性**的继承。这样，既通过在原型上定义方法实现了函数复用，又能保证每个实例都有它自己的属性。
```
    //创建父类
    function Father(firstName){
        this.firstName = firstName;
        this.colors = ["red",'blue','green'];
    }
    //创建子类
    Father.prototype.sayName = function(){
        console.log(this.firstName);
    }
    function Son(firstName,age){
    // 继承实例属性
        Father.call(this,firstName);
        this.age = age;
    }
    //继承原型属性和方法
    Son.prototype = new Father();
    Son.prototype.constructor = Father;
    Son.prototype.sayAge = function(){
        console.log(this.age);
    }
    var son1 = new Son("bangbang",18);
    son1.colors.push('black');
    console.log(son1.colors);   //["red", "blue", "green", "black"]
    son1.sayName(); //bangbang
    son1.sayAge();  //18

    var son2 = new Son('qiqi',17);
    console.log(son2.colors);   //["red", "blue", "green"]
    son2.sayName();             //qiqi
    son2.sayAge();              //17
```

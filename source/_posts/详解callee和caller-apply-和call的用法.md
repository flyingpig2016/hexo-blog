---
title: '详解callee和caller,apply()和call的用法'
date: 2017-07-10 13:31:05
tags: javascript高级程序设计
toc : true     
comments: true
---
### 一：函数内部的对象：arguments和this**
&emsp;&emsp;我们都知道js中arguments对象主要是保存函数的参数(如果不知道arguments，在有参数的函数中输出一下就知道了)，但是这个对象还有一个属性为callee，它是一个指针，指向拥有这个arguments对象的函数。例如阶乘函数：
```
    function factorial(num){
        if(num <= 1){
            return 1;
        }else{
            return num * factorial(num-1);
        }
    }
    console.log(factorial(5));//120
```
由于函数的执行和函数名factorial紧紧耦合在了一起，为了消除这种紧密耦合的现象，可以使用arguments.callee:
```
    function factorial2(num){
        if(num <= 1){
            return 1;
        }else{
            return num * arguments.callee(num-1);
        }
    }
    console.log(factorial2(5)); //120
```
两者的区别如下：
```
    var trueFactorial = factorial2;
    factorial = function () {
        return 0;
    }
    console.log(trueFactorial(6)); //720
    console.log(factorial(6));      //0
```
&emsp;&emsp;上面的trueFactorial实际上是在另一个位置上保存了一个函数的指针。然后又将一个简单的函数赋值给factorial变量。如果不用arguments.callee，那么，调用trueFactorial(6)的时候就返回0.但是上面的例子解除了代码和函数的耦合状态，因此trueFactorial仍然可以正常的计算阶乘。
<!--more-->
### 二：this的意义？**
&emsp;&emsp;this引用的是函数执行的环境对象（当我们在网页的全局作用域中调用函数的时候，this对象就是window对象）。也就是说，**在全局作用域中的所有变量和函数都是window对象的属性和方法**，例如：
```
    var color = 'green';  //相当于window.color = 'green'
    var obj1 = {color:"blue"};
    function saycolor(){
        console.log(this.color);
    }
    saycolor();         //green，这里的this代表window
    window.saycolor()   //green
    obj1.saycolor = saycolor;
    obj1.saycolor();    //blue
```
### 三：caller属性是什么？**
&emsp;&emsp;caller属性保存着调用当前函数的函数的引用。如果是在全局作用域中调用当前的函数，它的值就是null。例子如下：
```
    function a(){
        console.log(a.caller); //返回正在调用俺的函数
    }
    function b(){
        a();
    }
    a();    //null
    b();    //function b(){a();}
```
但是为了实现更松散的耦合，也可以通过arguments.callee.caller来达到相同的目的：
```
    function a(){
        console.log(arguments.callee.caller); //返回正在调用俺的函数b
    }
    function b(){
        a();
    }
    a();    //null
    b();    //function b(){a();}
```
不过arguments.callee在严格模式下会导致错误。
### 四：call()和apply()的应用**
&emsp;&emsp;每个函数都包含两个属性：length和prototype。其中length表示函数希望接受地参数的个数，例如：
```
    function a(){
        console.log(000);
    }
    function b(arg1){
        console.log(111);
    }
    function c(arg1,arg2){
        console.log(222)
    }
    console.log(a.length); //0
    console.log(b.length); //1
    console.log(c.length); //2
```
&emsp;&emsp;prototype属性是非常重要的，它指向一个对象，包含了所有它生成实例下的所有属性和方法，再次不再赘述，我的prototype博文中另有详细讲解。
每个函数都包含两个非继承而来的方法：apply()和call()。这两个方法的用途都是在特定的作用域中调用函数，其实就是所在函数体内this对象的值。
1.apply()接受两个参数：一个是在其中运行函数的作用域，另一个是参数数组。其中：第二个参数可以是array的实例，也可以是arguments对象。实例如下：
```
    function fun(arg1,arg2){
        return arg1 + arg2;
    }
    function callfun1(arg1,arg2){
        return fun.apply(this,arguments); //传入arguments对象
    }
    function callfun2(arg1,arg2){
        return fun.apply(this,[arg1,arg2])//传入参数数组
    }
    console.log(callfun1(3,7)); //10
    console.log(callfun2(3,2)); //5
```
2.call()方法和apply()方法的作用相同，曲别在于接收参数的方式不同。用call方法传递参数的时候必须直接传递，也就是传递给函数的参数必须一一列举出来，例子如下：

```
    function fun(arg1,arg2){
        return arg1 + arg2;
    }
    function callfun3(arg1,arg2){
        return fun.call(this,arg1,arg2);
    }
    console.log(callfun3(2,6)); //8
```
但是传递参数并非apply()和call()用的最多的地方，他们真正强大的地方时能够扩充函数赖以运行的作用域。例子如下：
```
    var age = 18;
    var person = {age:17};
    function say(){
        console.log(this.age);
    }
    say();
    say.call(this);  //18
    say.call(window) //18
    say.call(person) //17，此时运行的作用域为person，相当于把person的作用域传给了say函数里面
```
3.bind()用法。bind()方法会创建一个函数的实例，其this值会被绑定到传给bind()函数的值。例如：

```
    var age = 18;
    var person = {age:17};
    function say(){
        console.log(this.age);
    }
    var psay = say.bind(person);
    console.log(psay) //function say(){console.log(this.age);}
    psay();     //17，相当于把person作用域传给了say()构造函数里面，它和apply和call的区别就是，bind必须还要创建一个函数实例，也就是bind不能直接执行传参后的函数。
```
上面代码中person对象的作用域传到了bind()新创建的psay()函数执行环境里面，所以执行完输出this.age=17.

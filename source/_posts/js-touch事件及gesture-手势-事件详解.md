---
title: js touch事件及gesture(手势)事件详解
toc: true
comments: true
date: 2017-07-10 13:49:49
tags: javascript高级程序设计
categories:
---
### 一：touch事件分类？
1. touchstart：当手指触摸屏幕时触发。不管有多少个手指放在了屏幕上，只要再触摸一下屏幕就会触发。在此，我用小米的google浏览器做了一个实验，添加一个计数器，用来查看屏幕上手指触摸的次数，代码如下：
```
	window.onload = function(){
		var i = 0;
		document.addEventListener('touchstart',function(event){
			myDiv.innerHTML = '触屏的次数：'+ i++;
		},false)
	}
```
2.touchmove：当手指在屏幕上滑动的时候触发该是事件，在这期间可以通过event.preventDefault()来阻止滚动；
<!--more-->
3.touchend：手指从屏幕中移开的时候触发；
4.touchcancel：当系统停止跟踪触摸时触发。（不做说明）
例如我们在手机屏幕上利用前面三个事件来获取手指的位置，压力，接触面的值：效果如下：
![这里写图片描述](http://img.blog.csdn.net/20161219191933181?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmx5aW5ncGlnMjAxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
html和css代码：
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
		<meta name="viewport" content="width=device-width,height=device-height, user-scalable=no,initial-scale=1, minimum-scale=1, maximum-scale=1 ">
	<title>Document</title>
	<style>
		#myDiv{width:200px;height:200px;margin: 0 auto;}
		#myDiv2{width:40px;height: 40px;border-radius: 20px; background-color:deeppink;position: absolute;top:0;left:0;z-index:0}
	</style>
</head>
<body>
	<div id="myDivStart">Touch start(0, 0)</div>
	<div id="myDivmove">Touch move(0, 0)</div>
	<div id="myDivend">Touch end(0, 0)</div>
	<div id="myradius">Touch radius(0, 0)</div>
	<div id="force">Touch force：</div>
	<div id="rotateAngle">Touch rotationAngle：</div>
	<div id="myDiv2"></div>
</body>
</html>
```
js代码：
```
		function touchEvent(event){
		event = event || window.event;
		var mydiv2 = document.getElementById('myDiv2');
		var myDivStart = document.getElementById('myDivStart');
		var myDivmove = document.getElementById('myDivmove');
		var myDivend = document.getElementById('myDivend');
		var myradius = document.getElementById('myradius');
		var force = document.getElementById('force');
		var rotationAngle = document.getElementById('rotateAngle');
		var coordinateX = event.changedTouches[0].clientX;
		var coordinateY = event.changedTouches[0].clientY;	
		
		mydiv2.style.left = parseFloat(coordinateX)-20 + 'px';	
		mydiv2.style.top = parseFloat(coordinateY)-20 + 'px';
		myradius.innerHTML = "Touch radius：(" + event.changedTouches[0].radiusX.toFixed(2) + ' ,' + event.changedTouches[0].radiusY.toFixed(2) + ")";
		rotationAngle.innerHTML = "Touch rotationAngle：" + event.changedTouches[0].rotationAngle;
		force.innerHTML = "Touch force：" + event.changedTouches[0].force.toFixed(3);
		var intX = parseInt(coordinateX);
		var intY = parseInt(coordinateY);
		switch(event.type){
			case "touchstart" :
				myDivStart.innerHTML = "Touch started：(" + event.touches[0].clientX.toFixed(2) + ' ,' + event.touches[0].clientY.toFixed(2) + ")";
				console.log(event);
				break;
			case "touchmove" :
				event.preventDefault();//	阻止滚动
				myDivmove.innerHTML = "Touch moved：(" + intX + ' ,' + intY + ")";
				console.log(event);
				break;
			case "touchend" :
				myDivend.innerHTML = "Touch end：(" + intX + ' ,' + intY + ")";
				console.log(event);
				myradius.innerHTML = "Touch radius：(0.00,0.00)"
				force.innerHTML = "Touch force：0.00" ;
				break;
		}
	}
	document.addEventListener("touchstart",touchEvent,false);
	document.addEventListener("touchmove",touchEvent,false);
	document.addEventListener("touchend",touchEvent,false);
```
### 二：touch事件对应的event对象有哪些？**
&emsp;&emsp;根据上面的代码，可以得到，当触发touch事件的时候，会生成一个TouchEvent对象，如下图：
![这里写图片描述](http://img.blog.csdn.net/20161219140856227?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmx5aW5ncGlnMjAxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
常见的有上面划线三种属性，在做一些效果的时候，也是用以下的三个属性中的touchList对象来确定元素的所有信息。
1.touches：表示当前跟踪的触摸操作的touch对象的属性；
2.targetTouches：表示特定事件目标的Touch对象的数组。
3.changeTouches：表示从上一次触摸以来，发生了改变的touch对象的数组。
&emsp;&emsp;以上这些属性都是一个包含名为TouchList的对象，当触发点为1(即：event.touches.length ===event.targetTouches.lengh ===event.changeTouches.length)的时候，它们包含触摸点的所有信息,但是这时候虽然包含信息一样，但是执行(event.targetTouches == event.touches)返回值为false，那就是说，里面肯定有一些地方不相同，暂时还没有找到，如下图所示：
![这里写图片描述](http://img.blog.csdn.net/20161219141651698?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmx5aW5ncGlnMjAxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
下面是一些被触摸位置的一些重要属性：

- clientX和clientY属性：相对于视口的坐标；
- identifier：标识触摸的唯一id；
- pageX和pageY：相对于页面的坐标；
- screenX和screenY：相对于屏幕的坐标；
- radiusX和radiusY：能够包围用户和触摸平面的接触面的最小椭圆的水平轴(X轴)半径和垂直轴(Y轴)半径；
- rotationAngle ：这个属性和 Touch.radiusX ,  Touch.radiusY一起, 描述了用户和触摸平面的接触面.
- force：返回这个 Touch 对象对应的手指挤压触摸平面的压力大小, 从0.0(没有压力)到1.0(最大压力)的浮点数.
- target：触摸的DOM节点目标。
**注意：**在touchend事件发生时，touches属性和targetTouches属性中的touchList对象就没有任何东西了，所以，这时候要想取得触发点的信息，**就必须使用changeTouches属性。**
### 三：touch、mouse和click事件触发顺序？**
&emsp;&emsp;touch事件和click事件触发顺序：
1.touchstart事件；
2.mouseover事件；
3.mousemove事件(一次)；
4.mousedown事件；
5.mouseup事件；
6.click事件；
7.touchend事件。
兼容性：iOS版的Safari、Android版的webkit、bada版的Dolfin、OS6中的BlackBerry webkit、Opera中的Mobile10.1 和 LG专有的PHantom浏览器。但是只有IOS的Safari支持多点触摸。桌面版本的Firefox6+和Chrome也支持触摸。
### 四：iOS2.0中的手势事件：gesture事件**
&emsp;&emsp;**注意**：下面的事件只能在iOS中运行。在iOS中档两个手指操作的时候，就会产生手势，手势通常用于改变显示元素的大小，或者旋转显示的元素。手势事件分为三种：
1.gesturestart：当一个手指已经按在屏幕上，另一个手指又触摸屏幕的时候触发。类似于touchstart的作用一样；
2.gesturechange：当触摸屏幕的任何一个手指的位置发生变化的时候触发。
3.gestureend：当任何一个手指从屏幕上面移开时触发。
下面这些代码在安卓上不能用，只有在iOS上可以使用：
```
	var mydiv = document.getElementById('myDiv');
	function gesture(event){
		console.log(event)
		switch(event.type){
			case "gesturestart" : 
				mydiv.innerHTML = "gesture start (rotation=" + event.rotation + ',scale=' + event.scale + ')';
				break;
			case "gesturechange" :
				mydiv.innerHTML = "gesture change (rotation=" + event.rotation + ',scale=' + event.scale + ')';
				break;
			case "gestureend" :
				mydiv.innerHTML = "gesture end (rotation=" + event.rotation + ',scale' + event.scale + ')';
				break;
		}
	}
	document.addEventListener('gesturestart',gesture,false);
	document.addEventListener('gesturechange',gesture,false);
	document.addEventListener('gestureend',gesture,false);
```



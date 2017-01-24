---
title: JavaScript 变量声明
date: 2016-12-11 22:38:42
tags:
    - web
    - javascript
---
### 更新日志
2017-01-24 发现名字不太合适。。。改个名字。。。

发现这一块很迷啊。。
首先来看下问题~
```javascript
var foo = 1;
function bar() {
    alert(foo);
    if (!foo) {
        var foo = 10;
    }
    alert(foo);
}
bar();

```
你猜两个输出分别是什么呢~答案是undefined和10哦！答错了的话就跟我一探究竟吧！^ ^
<!--more-->
其实主要是搞明白第一个为什么是undefined，这个搞明白了后面那个执行完if语句自然就是10了。
**********************************
### 1.JavaScript的声明
1.1函数声明&变量声明
形如 `function foo() {}`和`var foo`的分别是函数声明和变量声明。
而JavaScript解释器会把这些声明，统一移到该代码所在作用域的顶部(参考 [ben cherry的JavaScript Scoping and Hoisting](http://www.adequatelygood.com/JavaScript-Scoping-and-Hoisting.html))
比如如下代码
```javascript
function foo() {
	bar();
	var x = 1;
}
```
会被解释成下面的样子
```javascript
function foo() {
	var x;
	bar();
	x = 1;
}
```
所以本文开头提到的代码会被解释成
```javascript
var foo = 1;
function bar() {
    var foo;//注意看这里
    alert(foo);//A
    if (!foo) {
        foo = 10;//注意看这里
    }
    alert(foo);//B
}
bar();

```
所以alertA处的foo仅仅被声明，还没有被赋值，就是undefined；
之后if条件为真，将foo赋值为10，所以alertB处输出10~

那么同理 想一想下面这段代码输出什么呢~
```javascript
var a = 1;
function b() {
    a = 10;
    alert(a);//A
    return;
    function a() {}
}
b();
alert(a);//B
```
答案是A:10，B:1哦~
解释一下：
跟上面变量声明一样的道理啦，这里的函数声明`function a() {}`会被解释器提到`a=10;`的前面去，
于是`a=10;`赋值的其实不是函数b外面的变量a，而是函数b内部的函数a(是不是有点晕。。其实不搞这些重名就没这么多事了。。但是多了解下原理总是好事~^ ^)
所以也不要以为return后面的就是死代码哦~这就是一个反例
未完待续。。。

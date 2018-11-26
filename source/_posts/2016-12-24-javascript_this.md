---
title: javascript之this的绑定
date: 2016-12-24 19:38:03
updated: 2016-12-24 19:38:03
tags:
    - javascript
---
### 前言
今天在[udacity](https://cn.udacity.com/)上听课~觉得老师对javascript里面的this讲得很清楚，总结一下~
<!-- more -->
### 基本原则
1.  看`点`的左边是什么，this就绑定什么。（回调函数、new有例外,见原则3 & 5）
2.  没有`点`的话，this就绑定默认值------全局
3.  回调函数大多数是全局，若用一个function包起来则满足原则一
4.  `functionName.call`可以绑定任意变量
5.  `new r.method(g,b);`的this，是一个新生成的对象
[总结的可能不大清楚，下面看看例子吧~]
### 举例
```javascript
var func=function(x,y){
  console.log(this,x,y);
}
var r={id:'r'},g={id:'g'},b={id:'b'};
r.method=func;
// -------下面是调用啦-----------------
func(g,b);     // <global>,g,b
r.method(g,b); // r,g,b
func.call(r,g,b); // r,g,b
setTimeout(func,10); // <global>,undefined,undefined
setTimeout(r.method,10); // <global>,undefined,undefined
setTimeout(r.method(g,b),10); // <global>,g,b
setTimeout(function(){r.method()},10);// r,undefined,undefined
setTimeout(function(){r.method(g,b)},10);// r,g,b
console.log(this);//<global>
new r.method(g,b);//<object>,g,b
```
### 解释
首先说一下上面提到的全局`<global>`，是一个默认存在的Window对象~
我在浏览器控制台看到它大概长这样：
`Window {speechSynthesis: SpeechSynthesis, caches: CacheStorage, localStorage: Storage, sessionStorage: Storage, webkitStorageInfo: DeprecatedStorageInfo…}`

前5行就不解释了~~下面的就按照行号来说啦

7  没有点，满足原则2。this绑定全局。
8  点左边是`r`，满足原则1。this绑定r
9  运用原则4：`function.call(<这是一个可选参数>,paramaters...)`
   而这个可选参数就是此函数this绑定的对象。
   `‘那如果这里我写func.call(g,b)是不是它的this又是全局了呢’`你可能会问
   并不是。因为它是从前往后对应参数的，意思就是你传了两个参数的话，也是第一个参数作为那个可选参数，第二个参数才作为该函数接口的参数，所以如果`func.call(g,b)`的话，输出就是`g,b,undefined了`~
10 setTimeout是回调函数，符合原则3的前半句。this绑定全局
11 依旧满足原则3前半句。此处虽然有 点 ，但回调函数特殊。**下面有解释**
12 跟11同理，列出来是想表示回调函数也可以正常传参
13 满足原则3后半句，因为点之前是r，所以this绑定r。**下面有解释**
14 跟13同理
15 即使不在方法里，this也不是undefined哦~依旧是默认的全局~
16 这个有点迷。老师说带new的话，this绑定的是一个新生成的对象，而且每这样new一个，就会新生成一个对象。//这种情况没太理解，那个新生成的对象意义到底是什么。希望了解的大神告知~谢谢！~~
### 关于11、13的原理解释
setTimeout是js库里面自带的方法，它的结构也无非就是下面这样
```javascript
setTimeout(func,time){
  wait_for_a_while(time)   //这方法是我yy的啊不要当真，就是这么个意思
  ----some codes-----
  func();  //一定会在某个地方调用传进来的方法
  ----some codes------
}
```
于是，11那种`r.method`的，r.method仅仅是作为一个方法传给了setTimeout，相当于r.method只是这个方法的名字，而不会被解读成r的method方法
就是说,在setTimeout里，它根本不知道r的存在
可以类比成下面这种情况：（java语言）
有一个方法的接口是`public void test(int x)`
有一个对象obj，有int类型的属性id，
而`test(obj.id)`这样的调用并不会把obj暴露给方法test，对于它来说x只是一个整数而已。
//啰嗦了这么多。。。发现自己表达能力不太够。。。哪里没理解或者有不同见解的欢迎提出！~

而，13那种用function包起来的~setTimeout在执行的时候就会去执行这个匿名函数的函数体~~
这个函数体里有一句r.method(),于是setTimeout就会执行r.method啊~于是就满足原则1啦~

哦对了~圣诞快乐！！！~~~~
**********************

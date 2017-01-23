---
title: javascript厉害的Array
date: 2017-01-19 11:14:03
tags:
    - javascript
---
### 1.摘要
虽然javascript数据类型少，但是功能还是挺多的~
比如Array类型！又可以当栈又可以当队列^o^
而且Array还有一个特点是：一个Array里面的每一项都可以是不同的类型~也算是一个有利有弊的特性吧~
### 2.能当栈又能当队列
为了实现这一点，javascript的Array有4个方法
分别是pop,push,shift,unshift。
<!-- more -->
作用分别是对Array尾部弹出一项、尾部添加N项、头部弹出一项、头部添加N项。（N>=1）
返回值分别是弹出项、Array的最新长度、弹出项、Array的最新长度。
于是！基于这4个操作~
pop和push组合，或shift和unshift，就可以模拟出 **栈** 的操作
pop和unshift组合，或shift和push, 就可以模拟出 **队列** 的操作
下面看看例子吧~
```javascript
var array=['aaa','bbb','ccc','ddd'];
console.log('original array: '+array);
var x=array.push('ccc');
console.log(x);     //5
console.log(array); //aaa,bbb,ccc,ddd,ccc
var y=array.push('eee','fff');
console.log(y);     //7
console.log(array); //aaa,bbb,ccc,ddd,ccc,eee,fff
var z=array.pop();
console.log(z);     //fff
console.log(array); //aaa,bbb,ccc,ddd,ccc,eee
var m=array.shift();
console.log(m);     //aaa
console.log(array); //bbb,ccc,ddd,ccc,eee
var n=array.unshift('ggg');
console.log(n);     //6
console.log(array); //ggg,bbb,ccc,ddd,ccc,eee
var p=array.unshift('hhh','iii');
console.log(p);     //8
console.log(array); //hhh,iii,ggg,bbb,ccc,ddd,ccc,eee
// 这最后一种情况要注意一下~
// 我原本以为会是 iii,hhh,...的，
// 因为想的是按照插入的顺序，先在头部插入hhh，再在头部插入iii嘛
// 结果hhh在iii的前面了。
// 再经思考，
// 觉得这大概跟'javascript的函数所有参数其实是一个数组'有关，
//（就是在实现上，函数的参数会合并起来成为一个数组，就是`arguments`,在函数内部可以通过arguments[i]依次访问传入的参数~~ 虽然严格来讲它不是数组，是类数组，但这里就不多说啦）
// 于是这里hhh,iii被看成一个数组，直接插入到原数组头部
```
[想看源码可以戳这里^ ^](https://github.com/disinuo/Demo_for_learningJS)（还包含javascript系列的其他部分的源码哦~~）~
**********
参考：
《Javascript高级程序设计》第2版--Nicholas C. Zakas 第5章

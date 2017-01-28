---
title: javascript之继承
date: 2017-01-28 07:57:03
tags:
    - javascript
---
### 继承原理
javascript里的继承，跟java、C等比起来，看起来好像很随意。。。
因为javascript里的‘类’的构成，包含两部分嘛---构造器和原型，
所以继承也包括两部分~原型继承和构造器继承，都得手动自己来~所以其实也可以单独继承构造器或者原型。
### 继承Part1--原型继承
### 继承Part2--构造器继承
javascript的继承是基于原型链哒~
原型链上越靠近顶端的原型就是它相邻的，原理顶端的原型的‘父类’或‘超类’
比如原型链
[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

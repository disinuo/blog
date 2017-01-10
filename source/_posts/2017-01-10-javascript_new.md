---
title: javascript关键字new
date: 2017-01-10 15:38:03
tags:
    - javascript
---
关键字new,相当于让解释器帮你加入两行代码
```javascript
this=Object.create(ClassName.prototype);
return this;
```
举个例子
<!-- more -->
```javascript
var Animal=function(name){
  this.name=name;
}
Animal.prototype={
  sleep:function(){
    console.log("sleeping~");
  }
}
var dog=new Animal('dogge');
console.log(dog.name);      //dogge
console.log(dog.sleep());   //sleeping
// 用关键字new，解释器会将Animal自动添加上述的两行代码
// 变成下面的样子
var Animal=function (name) {
  this=Object.create(Animal.prototype);
  this.name=name;
  return this;
}
// 如果不用`new`呢
var cat=Animal('Kitty');
console.log(cat);       //undefined。因为Animal方法没有返回值
```

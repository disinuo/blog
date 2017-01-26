---
title: javascript 原型与构造器之‘构造器’
date: 2017-01-26 23:18:03
tags:
    - javascript
---
### 构造器
其实构造器就是方法，只是当你用`new`调用它的时候它就是构造器了（new的作用参见[javascript关键字new](https://disinuo.me/2017/01/10/2017-01-10-javascript_new/)）。
所以如前面提到的
```javascript
var Person=function () {
};
```
就是Person的构造器。
构造器可以初始化一些属性，比如
```javascript
var Person=function (name,age) {
    this.name=name;
    this.age=age;
    this.hobbies=['Drawing','Music'];
};
```
构造器这样修改之后，创建Person实例的时候可以传参，且该 __实例本身__ 有属性name、age、hobbies。
下面对原型也做一点小改动
```javascript
Person.prototype={
    constructor:Person, //如果是以字面量形式赋值原型的话，constructor属性要显示定义
    sex:'Male',
    friends:['Lihua','Oliver'],
    getName:function () { //新增getName方法
        return this.name;
    }
};
var person1=new Person('Lily',18);
var person2=new Person('Amy',24);
person1.sex='Female';
console.log(person1.sex);//Female
console.log(person2.sex);//Male
person1.friends.push('Selena');
console.log(person1.friends);//Lihua,Oliver,Selena
console.log(person2.friends);//Lihua,Oliver,Selena
person1.friends=['Lihua'];
console.log(person1.friends);//Lihua
console.log(person2.friends);//Lihua,Oliver,Selena
person1.hobbies.push('Movie');
console.log(person1.hobbies);//Drawing,Music,Movie
console.log(person2.hobbies);//Drawing,Music
```
解释一下：
执行`person1.friends=['Lihua'];`后，person2的friends没有跟着变是因为上一节提到的，person1在自身上创建了新属性friends，于是没有修改原型的friends
***************
参考
[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

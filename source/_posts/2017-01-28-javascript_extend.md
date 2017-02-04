---
title: javascript之继承
date: 2017-01-28 07:57:03
updated: 2017-01-28
tags:
    - javascript
<!-- photos:
  - /image/2016-09-28-forceDelete/alert_delete.png -->
---


### 前言
说明一下，本文提到的‘类’，不是java、C里那种‘类’~
javascript里是没有类的概念的，只是为了方便表达~将一个包含构造器、原型的变量叫做‘类’。
javascript里的继承，跟java、C等比起来，看起来好像很随意。。。
因为javascript里的‘类’的构成，包含两部分嘛--------构造器和原型，
所以继承也包括两部分~原型继承和构造器继承，都得手动自己来~
所以其实也可以单独继承构造器或者原型。
<!-- more -->
### 继承原理
javascript的继承是基于原型链哒~
原型链两个相邻的原型，靠近顶端的是另一个的‘父类’或‘超类’。
比如有原型链如下
>A -> B.prototype -> Object.prototype ->null

B是A的父类~
### 举例
假设有一个`书`的‘类’，下面是它的构造器和原型（构造器和原型不知道是啥的话可以看这里[javascript 原型与构造器之‘原型’](https://disinuo.me/2017/01/26/2017-01-26-javascript_prototype/)，[javascript 原型与构造器之构造器](https://disinuo.me/2017/01/26/2017-01-26-javascript_constructor/)）
```javascript
var Book=function (name) {
    this.name=name;
};
Book.prototype={
    constructor:Book,
    getName:function () {
        return this.name;
    }
};
```
然后呢~有一个`小说`的'类'要继承`书`。然后看下面继承的两个步骤~
### 继承Part1--构造器继承
```javascript
var Fiction = function (name,author) {
    Book.call(this,name); //调用Book的构造器
    this.author=author;   //在构造器里给Fiction添加了一个“作者”属性
};
```
先解释一下`call`函数。
call是javascript里Function类型变量自带的函数，参数个数不定。
第一个参数是会作为该函数执行时的`this`。关于this可参考(javascript之this的绑定)[https://disinuo.me/2016/12/24/2016-12-24-javascript_this]
接下来的参数就都是调用call的函数本身的参数啦~
所以这里`Book.call(this,name)`是把Fiction的this传给Book，把name传给Book的构造器。
> 为什么要这么费劲的把Fiction的this传给Book呢？直接`Book(name);`不行吗？

不行哒~因为Book的构造器里是`this.name=name`，如果不传入Fiction的this，这句代码的this就是Book的this，就不会按照我们预想的给Fiction类创建name属性。所以要用[call函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)或者[apply函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)调用父类构造器~

所以可以看到~只要在构造器里调用另一个'类'的构造器，就算是在构造器上继承了~
这个子类也就拥有了父类在构造器上创建的方法、属性。
比如接下来我们创建一个实例~
```javascript
var blind_fiction=new Fiction('Harry Porter','J.K.');
console.log(blind_fiction.name);   //Harry Porter
console.log(blind_fiction.author); //J.K.
console.log(blind_fiction instanceof Book); //true
```
instanceof是用来判断一个变量是否是一个“类”的实例哒~
判断的原理是查找这个变量的原型链，只要在原型链里找到了这个“类”，就会返回true。
所以可见，虽然我们只是手动的进行了构造器继承，但js解释器会在子类的原型链里添加父类的原型~
原型链就这么串起来啦~
但这样还没有完全继承，这时候还不能通过子类的实例访问父类原型里的属性、方法，所以我们来看下一个part~
### 继承Part2--原型继承
继续上面的例子~
```javascript
Fiction.prototype=Object.create(Book.prototype);//继承Book原型
Fiction.prototype.constructor=Fiction;//显示定义Fiction的构造器
Fiction.prototype.getAuthor=function () {//给Fiction原型添加getAuthor方法
    return this.author;
}
```
Object.create方法的参数是一个原型~
明明一个类创建完，它的原型里会自动包含一个constructor属性，指向自己的构造器，那第二行为什么还要显示定义一下构造器呢？
因为第一行重新给Fiction的原型赋值了嘛~原来的原型就被覆盖了，这个新的原型就不包含constructor属性了，所以这里如果输出Fiction的构造器会输出Book的构造器（`console.log(Book.prototype.constructor);`）。
因为访问Fiction没有的属性，会继续沿着原型链向上找~于是就找到了Book的构造器~
那现在验证一下原型链的继承吧~
```javascript
console.log(blind_fiction.getName());   //Harry Porter
console.log(blind_fiction.getAuthor()); //J.K.
```
### class,extends语法糖
#### 示例
ECMAScript6 为了让熟悉java、C的开发人员用js用的更顺手，引入了一套新的关键字~
class,extends,constructor,super。下面举个例子~
```javascript
class Animal{
    constructor(name){
        this.name=name;
    }
    getName(){
        return this.name;
    }
}
class Dog extends Animal{
    constructor(name,age) {
        super(name);
        this.age=age;
    }
    getAge(){
        return this.age;
    }
}
var animal=new Animal('Baby');
console.log(animal.getName());  //Baby
console.log(animal.name);       //Baby

var dog=new Dog('dogge',8);
console.log(dog.name);      //dogge
console.log(dog.getAge());      //8
console.log(dog instanceof Animal); //true
```
#### 解释
- class里面，constructor就是对应的构造器啦~
- class里除constructor外的所有方法都是创建在原型里的。
- 关于class关键字的使用：声明和赋值。
上面例子里形如`class XXX{}`的就是声明。下面这种形如`var XXX=class{}`的是赋值。
```javascript
var Animal =class {
    constructor(name){
        this.name=name;
    }
    getName(){
        return this.name;
    }
}
var Dog =class extends Animal{
    constructor(name,age) {
        super(name);
        this.age=age;
    }
    getAge(){
        return this.age;
    }
}
```
- 这里的`声明`，跟js普通的变量声明有一点不同，变量声明有`变量提升`现象~(参考 [ben cherry的JavaScript Scoping and Hoisting](http://www.adequatelygood.com/JavaScript-Scoping-and-Hoisting.html))
而class是没有的，所以如果父类的声明在子类之后是会报错的！~~

本文涉及的源码可以[戳这里^ ^](https://github.com/disinuo/Demo_for_learningJS)（还包含javascript系列的其他部分的源码哦）
大年初一写博客的我绝对是真爱~哈哈哈
******
参考：
[MDN 继承与原型链文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
[MDN instanceof文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)

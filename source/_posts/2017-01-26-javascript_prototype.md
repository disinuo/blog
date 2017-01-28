---
title: javascript 原型与构造器之‘原型’
date: 2017-01-26 20:18:03
tags:
    - javascript
---
### js独特的原型
不同于C、Java等语言基于`类`，javascript是基于`原型`的。
原型本质也是对象。
每个对象都有原型（javascript中最基本的原型是Object.prototype）。
每个实例（当然，一定是个对象）都有一个原型，而这个原型作为一个对象，也有原型，从而形成了原型链。
原型链的顶端是null（Object.prototype的原型是null）。
<!-- more -->
### 原型示例
下面创建一个“类”Animal，以及Animal的原型。（这里“类”打了引号，因为它不是类，但又不知道怎么叫它了。。。）
```javascript
var Animal=function(){
}
Animal.prototype={
  name:'Dogge',
  sleep:function(){
    return "sleeping~";
  }
}
```
可见我们在Animal的原型上创建了一个属性`name`，一个方法`sleep`。
### 原型链的形成
接下来创建一个基于上面这个原型的实例
```javascript
var dog=new Animal();
console.log(dog.name);//Dogge
```
此时对象dog的原型链是 __dog -> Animal.prototype -> Object.prototype -> null__。
这里说明一下，通常情况下原型链的顶端两个都是 Object.prototype -> null，除非像下面这样这样创建
```javascript
var a=Object.create(null); //此时原型链是 a -> null
```
`Object.create`方法是用来创建对象的，参数是该对象基于的原型。比如可以这样
```javascript
var a = {count: 1};
// a ---> Object.prototype ---> null

var b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
```
### 实例与原型的关系
原型是唯一，并且实例间共享的。
可以理解为，通过实例可以访问到原型的属性（这里的属性包含方法，下同），是因为实例持有原型的属性的`引用`。
所以如果一个实例修改了原型的属性，基于该原型的其它实例的该属性也会被修改。
```javascript
var Person=function () {
};
Person.prototype={
    sex:'Male',
    friends:['Lihua','Oliver'],
};
var person1=new Person();
var person2=new Person();
person1.friends.push('Selena');
console.log(person1.friends);//Lihua,Oliver,Selena
console.log(person2.friends);//Lihua,Oliver,Selena
```
这里，本来是想给person1增加朋友Selena，结果person2的朋友也变了。这就很尴尬了。
将在下一篇[javascript 原型与构造器之构造器](https://disinuo.me/2017/01/26/2017-01-26-javascript_constructor/)的部分说怎么办~
在此之前，你可能也发现了
```
person1.sex='Female';
console.log(person1.sex);//Female
console.log(person2.sex);//Male
```
改person1的性别，person2的没有变啊，这是怎么回事。
这就涉及到了javascript神奇的属性访问与创建的原则。
### 属性的访问与创建机制
- 创建
跟java不一样，在java里，如果给对象一个不存在的属性`赋值`（注意！是 __赋值__），是会报错的。
而在javascript里这一点就显得很智能。。。解释器会帮你自动创建一个属性并赋值~
- 访问
那访问属性的时候呢，是先在对象本身上找，找不到就再到它的原型上去找，找不到再到原型的原型上去找，以此类推，直到找到该属性或到达原型链顶端为止（到顶端也没找到就返回undefined）。

所以上面的代码`person1.sex='Female';`，由于person1对象本身没有属性sex，sex属性是它的原型的，所以解释器就自动帮person1创建了sex属性，并赋值为Female。
所以可以看到，这句代码并没有修改原型的sex属性，因而访问person2.sex就自然还是Male啦~
感兴趣的话可以加一句`console.log(person1);`，在浏览器控制台将对象展开，可以看到这样的结构
```bash
Person
---sex: 'Female'
---__proto__:   #这个就是原型属性
------sex: 'Male'
```
所以当打印`person1.sex`的时候打印的是Female，是因为上面提到的访问机制，先找person1本身，找到了sex属性，于是就返回结果停止查找了（这个现象叫`属性屏蔽`）。
那现在再回头想一下，为什么`person1.friends.push('Selena')`的时候操作的就是原型的属性了呢，为什么解释器没有自动给person1创建一个friends属性呢。
因为，前面我强调了是`赋值`。push不叫赋值。
这里我是这样理解的（有不同见解欢迎提出~）：
对Array才能push，所以解释器一定要认得friends属性是一个Array。
而javascript自动创建属性需要你的初始化赋值，否则它创建的属性就是`undefined`。
所以`person1.friends.push('Selena')`时，是先在对象本身上找friends了的，发现是`undefined`，这就跟没有是一样的，于是才继续找原型的。
而且我还试了一下，把原型的friends属性删掉，重新执行`person1.friends.push('Selena')`，会报错
```javascript
Cannot read property 'push' of undefined
```
就是因为friends是undefined所以报错了嘛~
而如果接着上面的代码再加几行
```javascript
person1.friends=['Lihua'];
console.log(person1.friends);//Lihua
console.log(person2.friends);//Lihua,Oliver,Selena
```
看~果然 `赋值`就会自动创建属性啦~

有点长了，`构造器`以及`原型和构造器比较`的部分放在下一篇吧 ^ ^~
本文涉及的源码可以[戳这里^ ^](https://github.com/disinuo/Demo_for_learningJS)（还包含javascript系列的其他部分的源码哦）
***************
参考
[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

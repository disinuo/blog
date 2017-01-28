---
title: javascript 原型与构造器之‘构造器’
date: 2017-01-26 23:18:03
tags:
    - javascript
---
### 构造器
其实构造器就是方法，只是当你用`new`调用它的时候它就是构造器了（new的作用参见[javascript关键字new](https://disinuo.me/2017/01/10/2017-01-10-javascript_new/)）。
```javascript
var Person=function (name,age){//这就是个Person的构造器
    this.name=name;
    this.age=age;
    this.hobbies=['Drawing','Music'];
};
Person.prototype={//这是Person的原型
    constructor:Person,
    sex:'Male',
    friends:['Lihua','Oliver'],
    getName:function () {
        return this.name;
    }
};
```
原型不大清楚的可以参考这篇  [javascript 原型与构造器之‘原型’](https://disinuo.me/2017/01/26/2017-01-26-javascript_prototype/) ^ ^~
<!-- more -->
### 创建对象本身的属性
上面的构造器里形如`this.name=name`的代码，都是在给 __对象自身__ 添加属性，
于是通过该构造器创建的 __实例本身__ 也就拥有了相应的属性。
与原型的属性不同，原型的属性是实例共享的，而构造器创建的属性，是每个实例独自拥有的。
### 代码示例
下面的代码基于上面Person的构造器和原型
```javascript
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
比较两个push的不同~
(除这4行输出外不理解的话可以看下我的上一篇博客~)[javascript 原型与构造器之‘原型’](https://disinuo.me/2017/01/26/2017-01-26-javascript_prototype/)
person1.friends的push，对两个实例都产生了影响；
person1.hobbies的push仅对person1产生了影响。
因为friends是原型的属性，实例共享；而hobbies是实例的属性，每个实例都有自己独立的。
### 避免实例间属性干扰
所以~要解决，像代码中‘改变person1的friends时，无意中也把person2的friends改了’这种情况，
可以把该属性从原型中转移到构造器里~就好啦~
### 原型 vs 构造器
下面这些结论都是我看书啊听课啊学到的+自己总结的~不是那么绝对~欢迎讨论~~
1.  实例独立的属性都放在构造器里：
相当于java里面类的成员变量。
为了避免上述‘friends’的类似情况。
但是又会有疑问，如果不是数组类型呢，比如String类型的name，
上一篇不是说赋值的时候解释器会自动在实例上创建属性吗。
确实，从实现的角度看确实不会产生实例间的干扰了。但是自动创建属性的话，实例和原型就都拥有了该属性，而原型的该属性就会永远被屏蔽，占着空间却没卵用的数据，会对性能造成影响的。
2.  ‘静态属性’放在原型里：
‘静态属性’是指基于同一个原型的实例们都一样的属性。
这种属性如果在构造器里，每个实例各自有一个，反而白占了很多空间
3.  方法放在原型里：
这个跟上一条是一样的道理，就像java里面，类的方法一样，每个对象都是一样的呀~
也就没有每个实例都独自持有一份的必要啦。

本文涉及的源码可以[戳这里^ ^](https://github.com/disinuo/Demo_for_learningJS)（还包含javascript系列的其他部分的源码哦）
***************
参考
[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

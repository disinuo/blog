---
title: javascript迷之闭包
date: 2017-01-23 20:42:03
tags:
    - javascript
---
### 1.闭包的概念
闭包，简单来说就是指那些能够访问另一函数作用域内变量的函数。
比如最基础的
```javascript
var person=function(){
  var name='Shelly';
  return function(){
    console.log(name);
  }
}
var func=person();
func(); //Shelly
```
<!-- more -->
作为一个码惯了java的人，是不大能理解这里为什么函数person都执行完了，还能成功打印出'Shelly'的。
事实上，函数person返回的匿名函数，在这里就形成了一个闭包。
可以理解为，这个闭包含有它所在作用域的局部变量的备份，比如这个例子里就是person的name的备份。
就是说，在person执行结束后，依旧可以通过person返回的匿名函数，来访问person的局部变量。
### 2.常见应用
- #### For循环
```javascript
function createFunction() {
    var result=[];
    for(var i=0;i<10;i++){
        result[i]=function () {
            return i;
        }
    }
    return result;
}
var funcs=createFunction();
for(var i=0;i<funcs.length;i++){
    console.log(funcs[i]());
}
// 预想是输出0到9，结果却输出了10个10。。。
// 这是为什么呢。。。
// 可以这样理解
// 注意上述代码的4至6行的匿名函数 function () {return i;}
// 虽然循环了10次产生了10个这样的匿名函数，但是它们形成的是1个闭包
// 这1个闭包里只有1个i，所以result的每一项的函数返回的都是同一个东西~
// 这也就解释了为什么是10个相同的数字
// 那为什么这个数字是10呢
// 因为在for循环了10次之后，i最终被赋值成10而结束循环
// 所以当上述代码12行处调用函数输出i的时候，输出的就是最终的i的值啦~
// 我也是花了好长时间才明白的。。。不理解的话多找找资料~本文结尾的参考链接都是不错的资料~说不定哪句话就点醒你了呢^ ^
//
// 另外，可以将上述代码4至6行改成下面的样子以达到预想目标
result[i]=function (x) {
  return function(){
    return x;
  }
}(i);
// 这样改可以实现，是因为，
// 把原来的function(){return i;}外面
// 包了一个`立即执行`函数（就是在函数体后直接追加小括号，里面放要传的参数，该函数就会被立即执行）
// 由于函数传参是值拷贝传递的，而且是立即执行，所以i的每个值都复制传给了x
// 所以当后面分别调用func[i]输出的就是0到9啦~
```
- #### 工厂
来看一个例子~
一个生产`检查变量类型`函数的工厂
```javascript
function isType(type){ //这个就是工厂啦~
    return function(obj){
        return Object.prototype.toString.call(obj)==='[object '+type+']';
    }
}
var isString=isType('String');//检查是否是字符串的函数
var isNumber=isType('Number');//检查是否是Number的函数
console.log(isNumber(233));   //true
console.log(isNumber('hhh')); //false
console.log(isString('hhh')); //true
// 上述代码中，工厂的返回值就形成了一个闭包，所以isString、isNumber可以访问isType的局部变量type
```
- #### 封装私有变量
可以用闭包来弥补一下javascript里没有private的遗憾~~
```javascript
var Person=function(){
    var name='Lily';
    var age=20;
    return {
        getName:function(){
            return name;
        },
        addAge:function () {
            age++;
        },
        getAge:function () {
           return age;
        }
    }
}
var func=Person();
console.log(func.name);     //undefined
console.log(func.getName());//Lily
func.addAge();
console.log(func.getAge()); //21
```
### 3.总结
闭包是一个神奇又强大的东西~
现在理解的还是不够深刻
继续学习~~~
欢迎指出错误~欢迎讨论~
[想看源码可以戳这里^ ^](https://github.com/disinuo/Demo_for_learningJS)（还包含javascript系列的其他部分的源码哦）
**********
参考：
 [stackoverflow的大大~](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work)
 [MDN文档](https://developer.mozilla.org/cn/docs/Web/JavaScript/Closures)
《Javascript高级程序设计》第2版--Nicholas C. Zakas 第7章
《Javascript设计模式与开发实践》--曾探  第3章

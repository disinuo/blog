---
title: 警惕javascript的内存泄漏
date: 2017-01-18 21:38:03
tags:
    - javascript
    - 内存泄漏
---
javascript的BOM和DOM对象是基于C++的COM的，它的垃圾收集机制是<font color=#e06c75>引用计数策略</font>，
循环引用会造成内存泄漏。
### 引用计数策略思想
跟踪变量被引用的次数，次数为0的会被回收
### 举例
<!-- more -->
```javascript
function func(){
  var ele=document.getElementById('id');
  var obj=new Object();
  ele.obj=obj;
  obj.ele=ele;
}

```
DOM元素与一个javascript原生对象建立了循环引用。
即使func执行完，两个对象都被两一个对象引用着，被引用数永远不会为0，
于是两个对象也永远不会被自动回收。
就出现了内存泄漏
### 应对策略
暂时知道的办法就是。。。在结束使用对象的时候，
手动断开两对象的连接：
```javascript
ele.obj=null;
obj.ele=null;
// 是的这样就可以了！
// 这样基于引用计数策略的垃圾回收器在下次检查的时候
// 就会发现他们的引用数变成了0，就把它们回收了~~
```

**********

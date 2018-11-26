---
title: java：Array(数组)和ArrayList的区别
date: 2017-05-31 14:54:21
updated: 2017-05-31 14:54:21
tags:
    - java
    - 数据结构
---
### 前言
在java中，ArrayList其实是内部封装了一个数组来放数据的。那我就想他们两个究竟有哪些区别呢
这篇博客主要以[8 Difference Between Array And ArrayList In Java With Example](http://javahungry.blogspot.com/2015/03/difference-between-array-and-arraylist-in-java-example.html)为基准~，有的地方加了自己的理解
<!-- more -->
### 区别
#### 是否可改变size
- 数组不能改变size
- ArrayList的size是动态的
- 原理：不是说ArrayList里也是一个数组嘛，那它怎么实现的动态size呢~看下源码（省略部分细节）

```java
//下面的【elementData】就是那个放数据的数组
public boolean add(E e) {
   //这个方法的作用是 【size不够的话就扩容】
    ensureExplicitCapacity(size + 1);  
    elementData[size++] = e;
    return true;
}
private void ensureExplicitCapacity(int minCapacity) {
    //需要的最小容量比当前的实际容量大-->去扩容！
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    //扩容到原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //扩容完还是不够，那干脆你要多少我就扩到多少
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //处理溢出的情况
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    //复制elementData
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
- 所以其实每次调用ArrayList的add方法的时候它都默默的做着这些工作

#### 性能
- 自动扩容：ArrayList的自动扩容会降低性能。因为扩容时是使用了临时变量实现复制的
- 它俩的add()、get()的复杂度一样，（数组的add指的就是通过序号赋值啦）

#### 元素类型
- 数组的元素类型可以是对象，也可以是基本类型
- ArrayList的元素类型不能是基本类型
比如你 **不能** 这样初始化一个ArrayList
```java
List<int> list=new ArrayList<int>();//会报错的！
```
那你可能会问，我可以这样写呀
```java
List<Integer> list=new ArrayList<Integer>();
list.add(3);
```
看起来你是加的基本类型int，但其实JVM会把它解析成
```java
list.add(new Integer(3));
```

#### 迭代访问
- ArrayList可以创建迭代器Iterator，并且这个迭代器是Fail Fast的（这个翻译过来感觉怪怪的）。
什么是Fail Fast呢，就是：创建了某个ArrayList的迭代器，在这个迭代器迭代的过程中，如果这个ArrayList的长度被修改了(包括add、remove，使用的不是迭代器的方法)，会立刻(Fast)抛出异常(Fail)。（update一个元素不会抛异常，亲测）
一个简单例子~
```java
public List<Integer> filterMinus(List<Integer> list){
  Iterator<Integer> itr=list.iterator();
  int i=0;
  while (itr.hasNext()){
    int x=itr.next();
    System.out.println(x);
    if(x<0){
        list.remove(i);
        // itr.remove(); 把上面那行注掉用这行的，就不会抛异常
    }
    i++;
  }
}
```
- 好像有点跑偏了，对 [Fail Fast 感兴趣的话可以戳这里](http://javahungry.blogspot.com/2014/04/fail-fast-iterator-vs-fail-safe-iterator-difference-with-example-in-java.html)

#### 类型安全
- 一个数组里的元素都是同一个类型的。因为创建数组的时候要声明类型嘛~
- 一个ArrayList的元素可以是不同类型
- 因为ArrayList在声明的时候可以不规定类型，其实可以理解为声明了一个`ArrayList<Object>的list`，这样看来的话也可以`Object[] array`声明一个数组~所以其实没什么区别啦

#### 获取长度
- 数组：length属性
- ArrayList：size()方法
- 本来我之前一直记混他俩获取长度的方式。现在终于明白为什么了。
  - 数组的长度不可改变，所以是一个public final的成员变量，反正大家都不可以改，直接public就好
  - ArrayList的长度是动态变化的，类内部可读可写，类外部仅可读，所以要把它封装成private的，外界通过get方法得到，也就是size()方法
  - 自己的理解，不对的话欢迎指出！~^ ^

#### 增加元素
- 数组：直接通过序号赋值。  eg：array[0]=3.1;
- ArrayList：通过add方法。eg：list.add(0,3.1);

#### 多维
原文作者说数组支持多维，ArrayList不可以。我不赞成这一点，不过这里还是写一下。比如下面这代码妥妥跑成功了
```java
List<List> list=new ArrayList<List>();
List row1=new ArrayList();
row1.add(3);
row1.add("hello");
row1.add(-8.8);
row1.add('a');
list.add(row1);
List row2=new ArrayList();
row2.add(88);
row2.add(666);
row2.add("world");
list.add(row2);
System.out.println(list);
//  输出
// [[3, hello, -8.8, a], [88, 666, world]]
```

### 总结
- 所以其实，我理解的ArrayList就是对Array的一个封装，提供了很多我们常用的方法（添加、删除一个元素等），减少了我们的使用负担（不必须初始化时决定大小、类型，自动扩容）

----------
参考
[8 Difference Between Array And ArrayList In Java With Example](http://javahungry.blogspot.com/2015/03/difference-between-array-and-arraylist-in-java-example.html)
[Fail Fast Vs Fail Safe Iterator In Java : Java Developer Interview Questions](http://javahungry.blogspot.com/2014/04/fail-fast-iterator-vs-fail-safe-iterator-difference-with-example-in-java.html)
[Difference between Array vs ArrayList in Java](http://www.java67.com/2012/12/difference-between-array-vs-arraylist-java.html)

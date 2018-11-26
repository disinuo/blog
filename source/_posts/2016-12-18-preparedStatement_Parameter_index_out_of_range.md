---
title: preparedStatement 报错 Parameter index out of range (1 > number of parameters, which is 0)
date: 2016-12-18 23:26:43
updated: 2016-12-18 23:26:43
tags:
    - java
    - j2ee
    - sql
    - jdbc
---
### 前言
还是在写j2ee作业。。。（写这作业真的是遇到了好多坑啊。。。orz），servlet跟数据库交互，进行sql查询嘛。
本来都好好的也测试过的，然后sql语句换了一个复杂一点的就开始报如题的错。。。我想这是没读到我的问号喽？
可是问号明明就在那里orz
<!-- more -->
### 部分代码
```java
String sql="SELECT exam.id as examID,"+
"exam.name as examName,"+
"exam.date as examDate,"+
"selectC.cid as courseID,"+
"course.name as courseName "+
"FROM selectC,exam,course "+
"WHERE selectC.cid = exam.cid "+
"AND course.id=selectC.cid "+
"AND selectC.sid = ?";
pstmt=cnn.prepareStatement(sql);
pstmt.setString(1, param);
ResultSet rs=pstmt.executeQuery();
```
看起来没毛病吧!我后来都想是不是问号打成中文字符的问号了，改了一下还是不行
然后。。。求助google，也没人遇到类似问题，
<font color=red size=4>报同样错的网友们大多是,把问号用单引号包起来了</font>
<font color=red size=3>像这样`String sql="SELECT * FROM user WHERE id = '?'"`</font>
**** # 如果你是这样就把单引号去掉吧~再跑一下看看是不是不报这个错了^ ^
然后我也不知道哪来的灵感。。把sql变成了下面这样
```java
String sql="SELECT exam.id as examID,exam.name as examName,exam.date as examDate,selectC.cid as courseID,course.name as courseName FROM selectC,exam,course WHERE selectC.cid = exam.cid AND course.id=selectC.cid AND selectC.sid = ?";
```
是的没错就是把追加形式的变成了一个字符串而已
然后！！
<font size=5>然后！！！</font>
就好了。。。orz
我只是觉得码代码的时候这样写一串看着太累就拆开了呀。。。好委屈。。。
### 找原因
然后我想找一找为什么这么改就好了。。这两个有什么区别呢。。。没找到
而且我今天又试了试（写这里的时候是出现bug的第二天），把sql语句又改成原来追加的形式。。它又不报错了。。。
### 总结
所以不好意思浪费了看本篇文章的你这么多时间。。
不过还是希望你如果遇到了同样的报错，
可以通过文中红色字提到的 `把包裹问号的单引号去掉`成功修复~ ^ ^

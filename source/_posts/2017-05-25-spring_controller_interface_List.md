---
title: spring的controller接口参数类型List<MyClass>报错
date: 2017-05-25 10:35:37
tags:
    - spring
    - java
    - j2ee
---
### 前言
在做一个j2ee项目，原本设计成前端发送HTTP请求时提交的数据是几个基本类型和一个数组，这个数组类型是一个自定义的类，长下面这样
```java
public class GuestInputVO {
    private String userRealName;
    private String idCard;
    private int vipId=-1;
    //省略了每个成员变量的Get、Set方法
}
```
### 报错场景复原
<!-- more -->
#### Controller的接口
```java
@ResponseBody
@RequestMapping(value = "/liveIn",
method = RequestMethod.POST,
produces = "text/html;charset=UTF-8")
public String liveIn(int bookBillId,int roomId,List<GuestInputVO> guests)
```
#### 前端请求数据格式
前端的http请求我用的是JQuery，发送的数据格式如下
```javascript
//这里是假数据~
var guests=[{
    userRealName:"Andy",
    idCard:"111222200033338888",
    vipId:""
  },{
    userRealName:"Miley",
    idCard:"111222198810096666",
    vipId:"1000008"
}];
var data={
        roomId: $('#roomId').val(),
        bookBillId: $('#bookBillId').val(),
        guests: guests
    };
```
#### 报错信息
然后！后端就炸了。把JQuery请求回调函数的返回Error放在html里可以看到下面的错误提示
![](/image/2017-05-25-spring_controller_interface_List/list_error.png)

### 抢救一：封装到bean里
Google了好多，终于有一个办法让我看到了一点曙光
#### 创建一个类，让List作为它的成员变量
说白了就是再封装一层，于是我有创建了一个类`LiveInVO`,结构如下
```java
public class LiveInVO {
    private int bookBillId;
    private int roomId;
    private List<GuestInputVO> guests;
    //还是省略了每个成员变量的Get、Set方法~
}
```
#### 更新Controller接口
```java
//省略@注释啦
public String liveIn(LiveInVO liveInVO)
```
#### 效果
前端那边是不用改的~这次我重启项目，发现！**报错内容换了**！！！
哈哈哈不知道你们会不会懂这种奇怪的兴奋点~
因为起码说明刚刚那个bug解决了呀~
于是来看看这次是什么报错
![](/image/2017-05-25-spring_controller_interface_List/bean_error.png)
为啥会这么stupid的去访问 `guests[0][idCard]`？？？
应该访问`guests[0].idCard`才对呀！！
想了半天 大概是spring不认识我封装在LiveInVO里面的数组类型GuestVO吧，把它当成了数组？毕竟我这里是嵌套了两层自定义类（个人猜测，欢迎讨论~~ ^ ^）
### 抢救二
那现在List问题解决了，但是两层嵌套的自定义类不可行，那我就把第二层换成Map吧~这样前端代码还是不用改
#### 自定义类型LiveInVO更新
```java
public class LiveInVO {
    private int bookBillId;
    private int roomId;
    // 注意这里改成了Map~~~
    private List<Map<String,Object>> guests;
    //还是省略了每个成员变量的Get、Set方法~
}
```
#### 相应的修改
然后可以在后端的某个地方把map转换成可读性更高的自定义类型`GuestInputVO`~
觉得放在Controller或service里都不够好，为了低耦合高内聚，
我是把转换方法做成了一个`GuestInputVO`新的构造器。代码如下
```java
    public GuestInputVO(Map<String,Object> map){
        this.userRealName=(String)map.get("userRealName");
        this.idCard=(String)map.get("idCard");
        try{
            this.vipId=Integer.parseInt((String)map.get("vipId"));
        }catch (Exception e){
            this.vipId=-1;
        }
    }
    // 加完这个构造器 如果没有显示的写一个无参构造器要记得加一个！不然spring没法构造bean啦~~~
```
### 总结
搞定啦~
或者有小伙伴有其他解决办法欢迎讨论~~~邮件或下面评论都可以哦^ ^

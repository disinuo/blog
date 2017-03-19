---
title: hibernate持久化---new一个对象 从session.save到存入数据库发生了什么
date: 2017-03-07 00:02:37
<!-- updated: 2017-03-07  -->
tags:
    - j2ee
    - hibernate
---
update时报错！
```
Illegal attempt to associate a collection with two open sessions. Collection :
```
改成merge就好了！
然而！merge是什么鬼！！！有空看一下
--------
当实体里有外键，List其他实体的时候
controller把实体返回给jsp会出问题，
于是要创建实体对应的vo，属性都是基本类型


controller用来返回数据的一定要记得加@ResponseBody

利用ajax，从controller返回的数据中文乱码
@RequestMapping(value = "/checkUser" ,*produces = "text/html;charset=UTF-8"* )//,method = RequestMethod.POST)
   @ResponseBody
   public String checkUser(HttpServletResponse response, String name){
       response.setContentType("text/html;charset=UTF-8");
       System.out.println("in /checkUser post");
       ResultMessage msg=userService.checkUser(name);
       System.out.println(msg);
       return msg.toShow();
   }


   controller不能返回double
   大概是js不认识double吧   转化成string返回就好啦





VO记得给一个默认的构造器！我全都生成了带所有属性的参数的构造器，于是默认的构造器就相当于被我覆盖了。。。

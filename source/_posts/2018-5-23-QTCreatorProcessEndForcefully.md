---
title: qt开发遇到的各种坑
date: 2018-05-23 16:51:42
updated: 2018-5-23 17:23:51
tags:
    - C++
    - qt
---
### 前言
我本科毕设是使用 *qt框架* 和 *QTCreator* 作为IDE进行项目开发，过程中遇到了很多很迷的bug或者遇到的问题在这边记录一下。
<!-- more -->
### BUG_1: 程序异常退出的因素

程序在编译构建的时候没有报错，但是运行时控制台就会突然输出
【程序异常结束-The process was ended forcefully.】，然后程序就挂了。
在经过google以及自己的摸索后，总结了几个会导致报这个错误的原因，如下。
- 直接使用空指针
- 直接使用没实例化的指针
- 除以0
- 数组越界
- delete空指针

反正就是记得把代码写的健壮性高一点吧，上面这些问题控制台都不输出具体的bug信息真的让人很蛋疼，一句【程序异常结束】就把我们打发了。。。orz
PS：debug模式可能是我不太会用吗反正也没定位到具体bug。。。如果有成功使用debug的小伙伴欢迎评论补充！~~

--------------
## BUG_2: error: symbol(s) not found for architecture x86_64
这个也是特别头疼的一个报错。。。可能存在的问题如下。
- 环境配置的qt版本与代码使用的qt版本不兼容
- slots如果在head里声明了而在cpp里没定义

## 如何绑定带参数的slot
下面这段代码要给一个button数组里的每个button都绑定同一个slot，但是传入的参数不同
``` C++
  media_ImageBtns = new QPushButton*[MEDIA_MAX_NUM];
  //使用一个QSignalMapper作为媒介
  QSignalMapper *signalMapper = new QSignalMapper(this);
  for(int i=0;i<MEDIA_MAX_NUM;i++){
      media_ImageBtns[i] = new QPushButton(tr("Button"),this);
      media_ImageBtns[i]->setEnabled(false);
      //下面这行代码的'map()'是qt自带的方法，
      //大致意思就是将每个button的“点击”信号，绑定给signalMapper的map()槽
      connect( media_ImageBtns[i], SIGNAL(clicked()), signalMapper, SLOT(map()));
      //大意是将每个button与一个参数绑定，并将这些绑定好的组存在mapper里
      signalMapper->setMapping(media_ImageBtns[i], i);
    }
    //将mapper与你自定义的带参数的slot绑定，
    //并且指定将刚刚与每个button对应的参数一一传递给你的自定义槽函数
    connect(signalMapper, SIGNAL(mapped(int)), this, SLOT(your_slot(int)));
```
所以其实这个signalMapper就是一个中介啦~ 帮忙暂存并中转一下数据而已
哦记得要引用头文件 `#include <QSignalMapper>`~


------------
参考资料
[Qt鼠标右键创建菜单](http://bbs.itheima.com/thread-330670-1-1.html)
[Qt使用connect函数时向slot传递参数](https://blog.csdn.net/imred/article/details/72940365)

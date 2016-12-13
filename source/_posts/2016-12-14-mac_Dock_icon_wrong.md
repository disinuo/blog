---
title: mac Dock图标错误、消失
date: 2016-12-14 7:15:42
tags:
    - mac
    - CleanMyMac
---
突然发现自己Dock里面出现了两个ppt，鼠标分别移过去发现是firefox的图标变成ppt的图标了。。。
这么醉的事情我还是第一次遇到。。。然后赶紧去应用程序那边看，firefox的图标还是正常的。。。于是我就把它从Dock中删除再重新加入，无济于事
后来在Mac讨论区看到网友说是使用CleanMyMac的锅。。。
然后找到了解决办法！
**********
打开终端执行这两条命令就好
```bash
rm ~/Library/Application\ Support/Dock/*.db;
killall Dock
```
上述命令是重置dock的~适用于图标消失【有的网友遇到的问题】和博主这种图标错误的情况~
ps：我还怕出事情事先备份了一下里面的db文件，，亲测好用~可放心食用^ ^

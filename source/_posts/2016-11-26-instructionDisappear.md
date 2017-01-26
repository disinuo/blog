---
title: mac基本指令无法识别。。。
date: 2016-11-26 14:54:42
tags:
	- mac
---
**********************************
### 故事是这样的
本来在配环境的，结果不仅环境没配好，吃个饭之后再打开终端发现基本指令都不！能！用！了。。。
比如sudo、ls、vi。。。
按下回车的结果都是
```bash
-bash:sudo:command not found
```
<!--more-->
然后我就慌慌张张的谷歌了一波  还好有相同遭遇的小伙伴分享经验~~
************************************
### 解决方法
#### 在终端输入
```bash
export PATH="/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin"
```
执行完之后那些指令就暂时可用啦~不过还要赶紧改一下.bash_profile
#### 在终端输入
```bash
open -e ~/.bash_profile
```
在文本编辑器里打开该文件，加上这一行
```bash
export PATH="/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin"
```
然后保存~就好啦~~  ^ ^

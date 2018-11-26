---
title: git分支管理(一)---创建、提交、切换
date: 2016-12-14 16:03:43
updated: 2016-12-14 16:03:43
tags:
    - git
---
### 前言
因为怕代码被我玩坏。。。一直没用过branch。。。今天查了一波~把觉得比较有用的知识整理下~
<!--more-->
### 创建分支
```bash
git branch dev    #创建一个名叫dev的分支，dev不用加引号
git checkout dev  #切换到dev分支
#上述两步可以合并为一步
git checkout -b dev #创建并切换到dev分支
```
### 查看分支
```bash
git branch  #可以查看所有分支
#会看到如下输出，*代表当前处于那个分支下
* dev
master
```
### 在分支上提交
过程与普通的提交一样
比如对README.md进行了修改，想在dev分支上提交
首先要保证切换到分支下 `git checkout dev`
```bash
git add README.md
git commit -m "branch test"
```
### 切换分支
执行完4 就完成了在dev分支上的提交啦
现在我们可以`git checkout master`切换回主分支看看~
然后就会发现README.me是修改前的样子~~~
【再切换回dev分支README.me又变成修改后了。我不会承认我来回切换了好多次哈哈哈。。。还是觉得很神奇的】
* ####  5.1.一个小思考
  结合网友们的讨论，以及n多次的试验，
  我发现如果对某个文件改动，但是不add就切换分支，工作区的改动都是在的（工作区说白了就是本地你看到的代码），即便是add了但不commit就切换分支也依旧如此。
  而且如果你改动了某个文件，在分支A下`git add`，但是在分支B下`git commit`的，最终这个修改是会记录在分支B下的。
  这也算是一定程度上验证了我在[git修改撤销](http://localhost:4000/2016/12/12/2016-12-12-git_note/)那篇文章里画的图^ ^
  ![](/image/2016-12-12-git_note/git_4_things.png)

  那为什么在分支间的切换，本地的工作区代码会有不同呢？下面说下我的理解：
  add+commit相当于是把改动从工作区【**转移**】到本地版本库的过程,就是说commit之后工作区就clear了。
  - 对于工作区clear了的情况，工作区代码会显示成版本库最后一次commit的样子；
  - 对于工作区不是clear的（就是有改动没有commit），工作区代码会将这些没commit的代码显示出来~
  所以上面我玩的不亦乐乎的分支切换[捂脸]就是因为改动都commit了~工作区clear了~两个分支就会分别显示各自最后一次commit的代码啦!^ ^

### push新分支
接下来！如果想push这个新的分支以至于在github可以看到，就要执行这个带参数的push
```bash
  git push --set-upstream origin dev  
  # 直接git push会报错的
  # fatal: The current branch test has no upstream branch.
  # To push the current branch and test the remote as upstream
```
### 分支合并
我发现好像大部分情况还是本地保留分支的，只是在分支上进行开发，然后合并到主分支上。
所以分支的合并显得尤为重要~而且分支的合并还是有不同方式的
【感觉再写有点长了，想看分支的合并我们下一篇见吧~^ ^】

************
参考：
[Benjamin Sandofsky的Understanding the Git Workflow](https://sandofsky.com/blog/git-workflow.html)
[阮一峰的《git分支管理》](http://www.ruanyifeng.com/blog/2012/07/git.html)
[廖大大的《创建与合并分支》](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840038939c291467cc7c747b1810aab2fb8863508000#0)

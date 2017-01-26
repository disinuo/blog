---
title: git分支管理(二)---合并
date: 2016-12-14 20:35:43
tags:
    - git
---
### 前言
本来没想到分支能写这么长的。。于是就拆成两篇了~想看另一篇的可以[戳这里 git分支管理(一)---创建、提交、切换](http://disinuo.me/2016/12/14/2016-12-14-git_branch/)^ ^
<!--more-->
### 指针机制
git 的每个commit都是一个节点，每条分支都有一个head指针指向该分支最新的节点。
比如下图有紫色分支和绿色分支，每个圆圈是一个commit节点，红色外框的圆圈是该分支的head指针指向的节点。
- ** 紫色是master分支，绿色是dev分支 **
![](/image/2016-12-14-git_branch_merge/head.png)

### 两种合并方式
- #### Fast Forward `--ff`
这是一种当其中一个分支没有新节点时的默认的合并方式（以下简称FF）
(就是你不必加`--ff`这个参数，如果可以用这种方式合并git自然会用这种方式的)
```bash
git merge dev
# 当前在master分支下，执行此命令会默认进行fast forward 合并
# 将dev合并到master
# 合并之后提交线变成一条线，两个分支的head是同一个节点
```
![](/image/2016-12-14-git_branch_merge/fast.png)

- #### Not Fast Forward `--no-ff`
![](/image/2016-12-14-git_branch_merge/no-ff.png)
```bash
git merge --no-ff dev
# 新建一个节点！
# 合并之后依旧可以从提交线中清晰地看出不同分支的commit历史
# dev分支的head没有变哦
```
  - 执行完这个指令会进入vim，让你输入新节点要commit的信息
  - 作为一个刚会用vim没多久的小白。。贴一下vim基本用法好了~~
  - 输入`i`之后允许输入；输入结束后按`Esc`；`:wq`保存并退出vim

- #### 两者比较
    最主要的差别其实就是提交线图不同：FF是一条线，noFF是多条线。

    NoFF的优点很强势~
    比如项目开发过程中，一般都是用master放比较稳定的代码，而在另外创建一个分支上写代码，测试好了再合并到master上。可能会有不同的功能点用不同的分支啊，或者不同的开发者用不同的分支之类的。如果是NoFF合并，就可以用git可视化工具清晰地看到各个分支的详细情况，便于追踪。
    比如下图是我上学期的一个大作业。我们团队四个人的提交线的部分(一个master和每人一个分支)~

    ![](/image/2016-12-14-git_branch_merge/commit_graph.png)
    而如果是FF合并就惨了。。都在一条线上。。很难找的~
    不过要是很小型的项目的话用FF会省事一点喽，就不用每次合并都搞一个新节点出来~
    自己视情况权衡利弊吧 ^ ^
- #### 总结
  写这篇博客之前查了好多资料，好多人说FF合并，分支删掉之后会丢失分支信息，我当时还以为是节点会丢失~
  然而亲测并没有。所以猜测那些人指的应该是会丢失分支的独立性吧（就是都混在一起看不出谁是谁啦> <）~
  我还试了一下，两个分支如果相对于分离点都有新节点(就像本文的第一张图)，但是改动的代码没有冲突，git会不会默认使用FF合并。
  亲测不会。而且我还试着加`--ff`参数强制FF合并，也还是执行了NoFF合并~
  就这些吧~等哪天研究一下git提交冲突的再来发文~~

PS：为了测试各种情况，快看我今天github上activity程度哈哈哈~创我历史新高~~~
![](/image/2016-12-14-git_branch_merge/github_activity.png)
***********
参考：
[Benjamin Sandofsky的Understanding the Git Workflow](https://sandofsky.com/blog/git-workflow.html)
[阮一峰的《git分支管理》](http://www.ruanyifeng.com/blog/2012/07/git.html)
[廖大大的《创建与合并分支》](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840038939c291467cc7c747b1810aab2fb8863508000#0)
【文中前3张图都来自[Understanding the Git Workflow](https://sandofsky.com/blog/git-workflow.html)~】

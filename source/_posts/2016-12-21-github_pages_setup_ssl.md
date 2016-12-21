---
title: 给github pages设置ssl(包含改了域名的情况)
date: 2016-12-21 17:02:43
tags:
    - ssl
    - github
---
### 前言
最近才知道http和https的区别，然后看到好多文章说http相比较于https怎么怎么没有前景。本来也是觉得我一个博客的网站就http吧也无所谓。但是有的浏览器会警告，看着就很心塞啊，于是找了找教程尝试了一下~
如果是用github pages搭建的博客，有两种可能：有没有设置自己的域名~
<!-- more -->
### 情况1.没有重新设置域名
就是你的url是`yourname.github.io`这种的，
这个就比较简单啦~github直接就可以配置(不过这个我没试~)
在github上进入你名为yourname.github.io的仓库->settings
![](/image/2016-12-21-github_pages_setup_ssl/github.png)
然后找到图片中的`Enforce HTTPS`，勾选上就ok了（因为我设置了自己的域名所以它不让我选）
### 情况2.设置了自己的域名
比如我设置成了disinuo.me
1.  **[注册Cloudflare](https://www.cloudflare.com/a/login)**
  Cloudflare是什么？它是一家免费CDN网站加速服务公司，还提供实时安全保护服务和网络优化等。【恩这是google到的，我现在还不太懂CDN是什么。。想了解Cloudflare的话可以去他官网主页看看~^ ^】
2.  **填写域名**
![](/image/2016-12-21-github_pages_setup_ssl/0.png)
比如我是`disinuo.me`,点begin scan就开始扫描你的网站啦
扫描结果如下，然后点continue
![](/image/2016-12-21-github_pages_setup_ssl/1.png)
3.  **选择计划**
默认是Business Website。我选的是上面那个免费的~^ ^
![](/image/2016-12-21-github_pages_setup_ssl/2.png)

4.  **修改NameServer**
![](/image/2016-12-21-github_pages_setup_ssl/3.png)
会列出了两个nameserver，提示你去域名商那里修改
我是在万网购买的域名~于是去[阿里云的域名控制台](https://netcn.console.aliyun.com/core/domain/list)修改
![](/image/2016-12-21-github_pages_setup_ssl/4.png)
点`管理`会出现下图
![](/image/2016-12-21-github_pages_setup_ssl/5.png)
之后修改成cloudflare提供的nameserver就好啦~
5.  **设置为flexible ssl**
其实这个默认就是设置成flexible的，就是说这一步可以不用管~
不过还是在这里贴一下去哪设置好了，就在主页点图示的`flexible`就可以修改啦
![](/image/2016-12-21-github_pages_setup_ssl/6.png)
6.  **耐心的等待**
然后就好啦~就是等啦！他官网上面说最多24小时可以生效。
我以为它就是客气客气。。。没想到我真的等了24小时。。。
所以如果好久也没生效别急哈~再等等~~~
之后就可以通过`https://yoursite`访问啦~~~
### 总结
- 其实Cloudflare的引导做的很好的，每一步都说的很清楚~也许你都不需要跟着本教程一步一步的走，不过还是希望多少帮助到了你一点~^ ^
- 现在访问`http://yoursite`还不能自动跳转到https的，待我再研究研究再来更新~
*************
参考链接：
[Set Up SSL on Github Pages With Custom Domains for Free ](https://sheharyar.me/blog/free-ssl-for-github-pages-with-custom-domains/)
[cloudflare.com是什么？新手入门中文注册使用教程](http://www.xujiahua.com/4680.html)

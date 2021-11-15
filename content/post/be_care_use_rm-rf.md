---
title: "谨慎使用 rm-rf 命令"
date: 2021-11-15T23:36:53+08:00
lastmod: 2021-11-15T23:36:53+08:00
draft: true
keywords: []
description: ""
tags: []
categories: []
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

<!--more-->

## 前言

```shell
$ rm -rf /*
```

著名的跑路代码，这句命令会使整个根目录删除，且没有回头路可以走，这个操作及其危险，不要手快就点了回车，特别是在 root 权限下。

分享一个最近的故事，我准备想把当前目录下的文件移到某一个目录下，使用的命令为：

```shell
$ mv /* home/usr/mydir
```

系统友情提示我没有相关权限，我就直接 sudo 执行了相关操作，好家伙，这个命令直接将整个根目录给移动到了某个文件夹，导致系统所有命令不能用了(这也理所当然，因为根目录都不在了，所有环境变量找不到程序了)。这个时候 shell 远程连接还没有断，如果你不幸断了的话，不好意思，你再也连接不上去了。

## 解决方案

这个时候如果你使用的是 su root ，目前还是 root 账户的话，还是有一线生机起死回生的，如果你在普通用户下，那不好意思，重装系统吧，反正是很难解决。

```shell
#这个时候你需要手动运行程序，并找到其动态链接库的地址
#以下命令不同机器的位置不同，需要查询 cp 运行的环境链接库在哪个位置
$ /home/george/test/test_tomcat/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 --library-path /home/george/test/test_tomcat/lib/x86_64-linux-gnu /home/george/test/test_tomcat/bin/cp -rfp /home/george/test/test_tomcat/* /
#如果不是root，没有权限访问根目录
#这个命令就把错误移动的根目录给强行搬回来了!

```

那么问题来了，移动当前目录下的文件到某一目录的命令是什么呢：

```shell
$ mv * ../ # 移动到上一级目录，千万不要加 /*
```

## 教训

在 root 账户下做的所有事情都要谨慎，因为其有系统的最高权限，linux的设计者认为使用root账户的人具备良好的linux运维知识，然而很多人并没有，所有请谨慎对待每一次操作，如果在单位服务器上更是慎之又慎。

## 参考

[1]: https://www.cnblogs.com/miaodi/p/6893773.html


---
title: "在服务器后台运行Jupyter Notebook服务"
date: 2021-09-10T22:18:12+08:00
lastmod: 2021-09-10T22:18:12+08:00
draft: false
keywords: [Jupyter Notebook]
description: ""
tags: []
categories: []
author: "Gravpher"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true 
toc: true 
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

本文介绍如何在 Linux 服务器后台运行 Web 服务，以及 `nohup`命令的使用。

## 背景

每次使用`Jupyter Notebook`都需要在终端上键入 `jupyter notebook`才能在浏览器上访问服务，这样就显得比较麻烦，可以让其像系统服务一样运行在后台，方便日常使用。同样 静态网页生成器`Hugo`、文档编辑器 `sharelatex`服务都可以使用该方法，实现后台运行。

## nohup命令使用

Linux中每一个命令在执行中都会启动一个进程，该进程在退出终端时将自动终止。对于特定的服务和应用，如果想一直运行在后台的话，就需要用到 `nohup`命令，该命令可以让程序在后台挂起。本质上，`nohup`命令是通过阻止进程接受到 `SIGNUP`(信号挂起)，该信号负责关闭终端时退出进程。

执行 `nohup`后，用户无法使用 `stdin`，并将 `stdout`和 `stderr`的内容输出到 `nohup.out`文件中。

> 如果 `nohup` 命令的输出被重定向到某个其他文件，则不会生成 `nohup.out` 文件

将输出重定位到 `output.txt`

```shell
$ nohup bash test.sh > output.txt
```

### 命令后台运行

```shell
$ nohup bash test.sh &   #生成 nohup.out 存储输出日志
```

### 后台挂起 jupyter notebook

```shell
nohup jupyter notebook --allow-root > jupyter.log 2>&1 &
```

上述命令将 `jupyter notebook`挂在后台运行，`> jupyter.log` 目的是将 `nohup`输出重定位到 `jupyter.log`文件里面，`2>&1`的含义为**将标准错误输出重定向到标准输出**。

```
事实上，1和2都是Linux系统中的文件描述符，其中0表示标准输入(stdin)、1表示标准输出(stdout)、2表示标准错误输出(stderr)，
```

## 参考文献

 https://www.geeksforgeeks.org/nohup-command-in-linux-with-examples/


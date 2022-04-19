---
title: "通过路由器端口转发方式实现内网访问"
date: 2021-09-28T16:58:46+08:00
lastmod: 2021-09-28T16:58:46+08:00
draft: false
keywords: []
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

## 摘要

本文主要介绍如何通过路由器端口转发的方式与内网进行通信。

## 背景

由于公网 IP 有限，在 IPv4 地址紧张的情况下，许多主机都没有公网 IP，只有局域网内的 IP 地址，出差过程中外网环境下，没法连接内网服务器，这会对我们许多工作带来不便。

## 环境准备

1、H3C 路由器

2、出差携带的外网笔记本

3、只有局域网 IP 的 Ubuntu 服务器

## 步骤

查看公网 IP 地址，

```shell
curl cip.cc
```

前提是一定要有一个公网 IP，如果你是通过路由器拨号连接到的互联网。分配的 IP 本身就是局域网的话，得到类似于如下的 IP 地址：

```
10.x.x.x

172.x.x.x

192.168.x.x

100.x.x.x
```

**这种情况下就无法进行端口映射**，这样的话就需要参考第三方的内网穿透软件才能实现。

### 1、进行路由器设置端口映射

![image-20210928165448105](https://cdn.jsdelivr.net/gh/George-Gou/PictureBed@master/2022/202109281701832.png)

### 2、按照提示配置参数

![image-20210928165552450](https://cdn.jsdelivr.net/gh/George-Gou/PictureBed@master/2022/202109281701538.png)

### 3、连接内部服务器

```shell
ssh -p 外部端口 用户名@服务器外网ip
```


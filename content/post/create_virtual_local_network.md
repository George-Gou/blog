---
title: "如何通过zerotier建立虚拟局域网"
date: 2021-08-31T14:47:45+08:00
lastmod: 2021-10-27T14:47:45+08:00
draft: false
keywords: [zerotier]
description: ""
tags: [环境配置]
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

本文主要讲述如何使用zerotier构建虚拟局域网，实现内网服务器的远程登录

## 前言

如果您像我一样有多台计算机设备，台式机、笔记本、服务器、平板电脑等，他们也许处在不同的地方，如果遇到出差场景，需要访问实验室或者家里的电脑访问数据，总是显得有点棘手。我们知道，当所有设备处于同一局域网时，当开通设备的远程连接功能，通过某种协议可以实现远程登录进行相关操作。但是如果出差，设备没有在同一局域网该怎么办呢？本文就来解决这个痛点。

## 解决方案

解决方案有许多，针对不同的条件选择不同的方法。一般实验室的电脑分配的局域网IP，如192.168.0.0 到192.168.255.255。没有公网IP的话，就没有办法穿透内网，访问内网服务器上的资源，这时候可以选择实现内网穿透的互联网服务，如本文使用的[zerotail](https://www.zerotier.com/)等服务，互联网有许多这种服务的提供商，可以根据自己情况选择，但是大多数需要购买相关服务，才能够保证使用体验。

### VPN虚拟专用网络

可以通过内网路由器设置端口映射对外开放端口来实现对局域网服务器的访问，这是最简单也最安全的方式。通过路由器配置虚拟服务器就可以实现，相信可以参考这个[博客](https://blog.csdn.net/u011854789/article/details/53510272)。配置完VPN服务器，就可以通过账号远程连接到内网，再进行相关操作。

### 内网穿透

内网穿透的核心是需要借助一台具有公网IP的服务器进行桥接，这样两者就可以通过这台服务器转发流量进而实现通信。不少应用都可以实现内网穿透功能，即外网IP直接访问到内网的服务器，前提是都需要安装客户端，且流量带宽有限制，免费使用用户体验极差。

### zerotier搭建虚拟局域网

什么是虚拟局域网？

> VLAN（Virtual Local Area Network）的中文名为"虚拟局域网"。
> 虚拟局域网（VLAN）是一组逻辑上的设备和用户，这些设备和用户并不受物理位置的限制，可以根据功能、部门及应用等因素将它们组织起来，相互之间的通信就好像它们在[同一个网段](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/%E5%90%8C%E4%B8%80%E4%B8%AA%E7%BD%91%E6%AE%B5/10612240)中一样，由此得名虚拟局域网。VLAN是一种比较新的技术，工作在[OSI参考模型](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/OSI%E5%8F%82%E8%80%83%E6%A8%A1%E5%9E%8B)的第2层和第3层，一个VLAN就是一个[广播域](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/%E5%B9%BF%E6%92%AD%E5%9F%9F/5293530)，VLAN之间的通信是通过第3层的[路由器](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/%E8%B7%AF%E7%94%B1%E5%99%A8/108294)来完成的。与传统的[局域网技术](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/%E5%B1%80%E5%9F%9F%E7%BD%91%E6%8A%80%E6%9C%AF/2597024)相比较，VLAN技术更加灵活，它具有以下优点： 网络设备的移动、添加和修改的管理开销减少；可以控制[广播](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/%E5%B9%BF%E6%92%AD/656406)活动；可提高[网络](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/%E7%BD%91%E7%BB%9C/143243)的安全性。
> 在计算机网络中，一个二层网络可以被划分为多个不同的广播域，一个广播域对应了一个特定的用户组，默认情况下这些不同的广播域是相互隔离的。不同的广播域之间想要通信，需要通过一个或多个路由器。这样的一个广播域就称为VLAN。

zerotier是虚拟局域网的一种解决方案，其具有配置简单、速度快、跨平台等特性，能够实现任意物理位置的设备通过下载其提供的客户端，加入同一网络就可以实现虚拟局域网的构建。

### zerotier教程

1、首先在[官网](https://www.zerotier.com/)注册一个账号：

2、随后新建一个网络（Create a Network）,目前免费的账户可以同时在线50个设备，对于一般人也足够用了。

3、各个设备下载对应平台的[客户端](https://www.zerotier.com/download/)，操作非常简单，安装后，直接选择加入网络，加入到刚刚生成的网络ID中（一串字符），然后在网络ID管理页面中配置参数，这可以根据需要自行配置（注意网络安全问题），私有网络需要对加入的设备进行验证，才能够分配指定IP地址。

## 结语

通过构建虚拟局域网，可以轻松访问服务器，建议重要数据和代码的服务器不要采用此种连接方式，可能存在网络安全问题，大家根据情况自行选择。经过测试，上文连接方式可靠，且可以跑满带宽，速度可观。

## 2021-10-27更新：

升级或者重装 zerotier 出现错误：

```shell
zerotier-one: fatal error: cannot bind to local control interface port
```

查看端口占用情况：

```
# sudo lsof -i:9993
COMMAND     PID         USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
zerotier- 29046 zerotier-one    6u  IPv4 131195      0t0  TCP localhost:9993 (LISTEN)
zerotier- 29046 zerotier-one    7u  IPv6 131196      0t0  TCP ip6-localhost:9993 (LISTEN)
zerotier- 29046 zerotier-one    9u  IPv4 131204      0t0  UDP george:9993 
zerotier- 29046 zerotier-one   10u  IPv4 131205      0t0  TCP george:9993 (LISTEN)
zerotier- 29046 zerotier-one   11u  IPv4 131206      0t0  UDP george:9993 
zerotier- 29046 zerotier-one   12u  IPv4 131207      0t0  TCP george:9993 (LISTEN)
zerotier- 29046 zerotier-one   13u  IPv4 131208      0t0  UDP george:9993 
zerotier- 29046 zerotier-one   14u  IPv4 131209      0t0  TCP george:9993 (LISTEN)
```

发现端口已经被zerotier占用，这个时候直接杀掉该进程

```
sudo kill 29046
#PID号
```

之后重启即可。

```
systemctl stop zerotier-one           //关闭zerotier
systemctl status zerotier-one         //显示是否已启动
systemctl restart zerotier-one        //重新启动
```


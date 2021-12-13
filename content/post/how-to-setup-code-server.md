---
title: "如何部署code-server到服务器"
date: 2021-12-08T20:19:01+08:00
lastmod: 2021-12-08T20:19:01+08:00
draft: false
keywords: []
description: ""
tags: []
categories: []
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
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
## 摘要
本文介绍如何在 CentOS7 服务器上部署 code-server 在线 vscode 编辑服务器。

## 环境准备
1、CentOS 7
2、在 [官网](https://coder.com/docs/code-server/latest) 下载最新版本的 code-server

## 具体步骤
手动安装：

1、```$ git clone https://github.com/cdr/code-server.git ```

2、```$ tar -xvf code-server-3.12.0-linux-amd64.tar.gz ```

3、```$ cd code-server-3.12.0-linux-amd64/bin ```

4、```(base) bin % ./code-server --port 10000 --host 0.0.0.0 --auth password```

```shell
[2021-12-08T14:54:40.830Z] info  Wrote default config file to ~/.config/code-server/config.yaml
[2021-12-08T14:54:41.276Z] info  code-server 3.12.0 4cd55f94c0a72f05c18cea070e10b969996614d2
[2021-12-08T14:54:41.278Z] info  Using user-data-dir ~/.local/share/code-server
[2021-12-08T14:54:41.291Z] info  Using config file ~/.config/code-server/config.yaml
[2021-12-08T14:54:41.291Z] info  HTTP server listening on http://0.0.0.0:10000
[2021-12-08T14:54:41.291Z] info    - Authentication is enabled
[2021-12-08T14:54:41.291Z] info      - Using password from ~/.config/code-server/config.yaml
[2021-12-08T14:54:41.291Z] info    - Not serving HTTPS ```
```


**注意：你的CPU类型要与下载的版本一致，否则会出现应用程序不能运行的情况**

当然你可以选择官方提供的脚本自动安装[传送门](https://coder.com/docs/code-server/latest/install#installsh)
## 后台运行 code-server
一般来说，关闭了命令窗口，该会话就会中断，这样会造成我们的服务不稳定。可以利用 screen 程序来完成后台运行，它是一个多重视窗管理程序，意味着用户可以共享某个会话，只要Screen本身没有终止，在其内部运行的会话都可以恢复。
```$ Screen -S code-server```
 ```$./code-server ```

**注意**:
```$ sudo vi /etc/sysconfig/iptables```
<img src="https://gitee.com/georgegou/gravitychina/raw/picture/202112090032023.png"/>

-A INPUT -j REJECT --reject-with icmp-host-prohibited

-A FORWARD -j REJECT --reject-with icmp-host-prohibited

这两句一定要放在最后才能让端口生效。
![修改后](https://gitee.com/georgegou/gravitychina/raw/picture/202112091558315.png)
这样才会使开放的端口生效。

**以下是服务开启成功的窗口**
<img src="https://gitee.com/georgegou/gravitychina/raw/picture/202112091308587.png"/>

---
title: "Groops docker 镜像制作"
date: 2021-12-24T10:47:44+08:00
lastmod: 2021-12-24T10:47:44+08:00
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
contentCopyright: true
reward: true
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true

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
本文主要介绍如何在 Ubuntu 系统下编译*重力场恢复与GNSS数据处理软件* **Groops**，为了大家使用简单，我制作了 docker 镜像，方便大家直接在镜像里运行 Groops 程序，不需要在本地计算机再次编译。软件文档请参考作者文档[^1]。

## Groops 软件简介

Groops 是用 C++ 编写 (加上经典的Fortran代码) 的大地测量数据处理软件，主要功能包括利用卫星和地面重力数据恢复地球重力场、GNSS 卫星精密定轨、GNSS 数据处理等。大多数算法都经过 MPI 并行设计，加快了计算速度。

## Groops 源码编译

Groops 从源码编译成可执行文件还是比较繁琐的，具体可参考作者给出的[安装文档](https://github.com/groops-devs/groops/blob/main/INSTALL.md)，不知是何原因，笔者在 win 环境下编译没有成功，在 Ubuntu linux下成功编译成了可执行文件，作者还建议安装 gmt 绘图软件，以及配置一些环境变量。

## Docker 容器构建

鉴于自己编译和配置的复杂性，会给读者带来不少麻烦，这里我在 docker 中构建了可以运行 groops 的容器，大家只需要pull 我的容器地址就可以轻松在本地使用 docker 程序。

### 在docker中运行 groops

```shell
$ (base) [george@-c ~]$ docker pull georgegou/groops_ubuntu
(base) [george@-c groops]$ docker images
REPOSITORY                TAG       IMAGE ID       CREATED          SIZE
georgegou/groops_ubuntu   latest    d60f0033e85b   46 minutes ago   1.53GB
# 运行docker镜像
(base) [george@-c groops]$ docker run -it -d --name groops -v /home/george/groops:/root georgegou/groops_ubuntu  /bin/bash
# -v 的含义为挂载本地磁盘到镜像磁盘，镜像中的修改会同步到本地

# 将编译的程序添加到环境变量
root@978c4b5de2e6:/# export PATH=$PATH:/home/george/groops/bin
root@978c4b5de2e6:/# groops
=== Starting GROOPS ===
Gravity Recovery Object Oriented Programming System (GROOPS)
Usage: groops [--log <logfile.txt>] [--settings <groopsDefaults.xml>] [--silent] [--global name=value] <configfile.xml>
       groops --write-settings <groopsDefaults.xml>
       groops --xsd <schemafile.xsd>
       groops --doc <documentation/>

 -h, --help           this text
 -l, --log            append messages to logfile. If a directory is given, one time-stamped logfile will be created inside for each groops script.
 -g, --global         pass a global variable to config files as name=value pair
 -c, --settings       read constants from file (default search: groopsDefaults.xml)
 -s, --silent         runs silently
 -d, --doc            generate documentation files (latex/html/...)
 -x, --xsd            write xsd-schema of xml-configfile options
 -C, --write-settings write the users current settings to file

GitHub repository: https://github.com/groops-devs/groops
(Compiled: Nov 4 2021 00:51:04)
```

### 构建 groops 镜像

下面我将展示如何构建运行 groops 的docker镜像。
1、构建镜像第一步是写 Dockerfile[^2]
```shell
FROM ubuntu:20.04
RUN buildDeps='g++ gfortran cmake libexpat1-dev libopenblas-dev libnetcdf-dev liberfa-dev mpi-default-dev gmt gmt-gshhg' \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
```
2、使用docker构建镜像
```shell
$ docker build -t ubuntu:20.04-dev .
bash-5.0$ docker images                     
REPOSITORY         TAG          IMAGE ID       CREATED        SIZE
ubuntu         20.04-dev      52464fc32c86   1 day ago        885MB
```
3、运行镜像，自己编译源程序
```shell
$ docker run -it -d --name groops-dev -h groops-srv -v /Users/george/Desktop/groops/groops:/root ubuntu:20.04-dev /bin/bash 
```
/Users/george/Desktop/groops/groops:为软件所在目录，将其挂载的镜像中，修改任意一处文件都会发生改变。

编译的具体步骤参照 https://github.com/groops-devs/groops/blob/main/INSTALL.md

大概一个多小时的编译之后，会在`bin`目录下生成两个可执行文件
```shell
root@28ae6145f5bb:~/bin# ls
groops  groopsMPI
```
4、提交镜像到自己的 dockerhub

将能跑起来的容器发布到自己的 dockerhub，方便别人拉取后就能运行程序。
```shell
$ docker login
$ docker tag d60f0033e85b groops_ubuntu #镜像ID，编写个名字
$ docker commit -m="groops_ubuntu" -a="georgegou" d60f0033e85b groops_ubuntu
$ docker push groops_ubuntu
```

## 参考文献

[^1]:[https://groops-devs.github.io/groops/](https://groops-devs.github.io/groops/)
[^2]:[https://github.com/groops-devs/groops/issues/21](https://github.com/groops-devs/groops/issues/21)
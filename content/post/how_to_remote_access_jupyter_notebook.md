---
title: "如何远程访问服务器端Jupyter Notebook"
date: 2021-09-09T23:00:27+08:00
lastmod: 2021-09-09T23:00:27+08:00
draft: false
keywords: [jupyter]
description: ""
tags: []
categories: [环境配置]
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

本文主要介绍如何在服务器端布置Jupyter Notebook环境，以及配置远程访问服务。

## 介绍

[Jupyter](https://jupyter.org/) 是一个基于Web的开源交互计算软件，支持多种语言，其可被用来编写文档、开发测试、展示结果。Jupyter Lab 是 Notebook 的下一代产品，其集成了更多功能。其核心功能是能够将代码、文档、图件、数学方程组合到一个易于分析的文档中，这样极大地增加了代码的可读性。Jupyter 主要是为R、Julia、Python等数据分析语言设计的，目前其已经成为数据挖掘、机器学习等数据分析工作的必要工具了。

## 如何安装 Jupyter Notebook

您可以通过`pip`或者 `Conda`安装最新版的Jupyter Notebook或者Jupyter Lab，目前版本为 6.3.0 

#### 利用conda安装

```shell
conda install -c conda-forge jupyterlab/jupyter notebook
```

#### 利用pip安装

```shell
pip install jupyterlab/jupyter notebook
```

## 初识Jupyter

命令行输入 `Jupyter notebook`即可在浏览器端 `localhost:8888/tree` 访问Web应用，默认端口为8888，运行后显示如下所示界面：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gitee.com/georgegou/gravitychina/raw/picture/202109100015122.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Jupyter notebook 主界面</div>
</center>

左上角`Files`显示当前项目的文件目录，`Running`表示当前运行的任务，`Clusters`表示并行运算的集群。

#### hello Jupyter

如何创建`Hello World`项目呢？依次点击`New`，`python3`，编辑下列代码，即可实现代码、文档的混排，使得代码清晰客观。具体使用可以参考[官方文档](https://jupyter.org/documentation)

<img src="https://gitee.com/georgegou/gravitychina/raw/picture/202109100040037.png" alt="tempsnip" style="zoom:75%;" />

## 配置 Jupyter 远程访问

如果服务器端配置了conda、Jupyter notebook、TensorFlow环境，需要在服务器端进行Jupyter编写与测试，这个时候就需要在本地Web上去实现操作，但是Jupyter默认在LocalHost开启服务，即使本机地址 `127.0.0.1`。如果需要在本地Windows平台访问相关服务怎么办呢？

#### 首先生成配置文件

通过修改默认配置的方式实现远程访问，首先生成默认配置文件：

```shell
jupyter notebook --generate-config
```

修改默认设置：

```shell
vi ~/.jupyter/jupyter_notebook_config.py  
```

主要修改以下几个参数：

`c.NotebookApp.open_browser = False` //防止在服务器端打开Jupyter Notebook

`c.NotebookApp.ip = '*'` //任意的IP都可以访问

`c.NotebookApp.port = '8888'` //默认端口

`c.NotebookApp.password= ****` //Web服务哈希密码，详情见配置文件注释

#### 终端运行Jupyter

本地客户端运行以下地址就可以远程访问 Jupyter notebook 服务

```shell
http://user:8888
```


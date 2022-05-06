---
title: "如何在 github 中贡献代码"
date: 2022-05-06T16:09:07+08:00
lastmod: 2022-05-06T16:09:07+08:00
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

  Github 是全球最大的开源社区，Git 是广泛使用的版本控制工具，其可以很方便地对各个版本的软件进行管理。
<!--more-->
## 1、环境准备

注册一个 Github 账号，下载并安装 Git 软件，[配置 Git 信息](https://blog.csdn.net/qq_38977566/article/details/118076502)，并 [添加 SSH key](https://blog.csdn.net/qq_37294163/article/details/103099424) 到 github 的设置中，就可以使用 SSH 的方式下载或上传代码。使用 SSH 比 Http 更加安全。

## 2、贡献代码

1、进入 Github 中的项目地址，比如说重力中文社区中的 [r2de 项目](https://github.com/Gravity-Geodesy-China-Community/r2de)，点击右上角的 Fork 按钮，将代码仓库复制到自己的账户下，这样就可以随心所欲地修改或添加仓库代码。

``` shell
# 将 Fork 的代码克隆到本地
git clone git@github.com:George-Gou/r2de.git

# 添加原始仓库地址
git remote add upstream git@github.com:Gravity-Geodesy-China-Community/r2de.git

# 每次工作前先更新远程仓库的最新代码
git fetch upstream
git checkout main
git merge upstream/main
git checkout -b dev

# 代码修改完了，可以看一下本次修改的代码有哪些，有没有误修改的代码
git diff .

# 如果代码没有问题，就可以提交到远程分支上
git add .
git commit -m 'add xxx feature' #这句话写本次做了哪些重要修改
git push -u origin dev
```
然后在 Github 网页上刷新自己的代码仓库，就可以看到：`dev had recent pushes 1 minute ago` 字样，右边有一个`Compare & pull request`按钮，点击它，就进入了`Open a pull request`界面，这里可以写一个 comment 就可以提交代码了。点击 `Create pull request`提交代码即可。
然后我再`review`大家提交的代码，如果没有问题就合并到主分支里面。
![image.png](https://cdn.jsdelivr.net/gh/George-Gou/PictureBed@master/2022/github_pull_request.png)

## 3、项目文档本地预览
项目是由 mkdocs 软件包构建，其是基于 Python 开发的，我们只需要安装 Python 和 mkdocs 环境就可以运行，非常简单。
```shell
pip install mkdocs
pip install mkdocs-material
cd r2de
# 运行文档
mkdocs serve
#即可以在127.0.0.1:8000端口访问
```
---
title: "如何使用PicGo进行Markdown图片的在线上传"
date: 2021-08-21T18:14:41+08:00
lastmod: 2021-08-21T18:14:41+08:00
draft: false
keywords: []
description: ""
tags: [PicGo]
categories: [computer-config]
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

由于Markdown格式文本中附带的图片是利用相对路径引用的，需要将md文件与存放img的文件夹放在一起才能够显示，如果将其放在公网访问，很有可能找不到图片路径而显示不全。现在Markdown编辑器Typora搭配PicGo程序就能够实现图片的在线上传。本文主要介绍如何使用PicGo工具实现Markdown格式的图片在线上传到图床中。

## 简介

[Markdown](https://www.markdownguide.org/)是一种轻量级标记语言，而网页前端所用的[html](https://www.runoob.com/html/html-tutorial.html)则是一种超文本标记语言，其是创建网页的标准标记语言。两者有一定关系，Markdown可以快速转化为html代码，许多工具都支持Markdown格式编写网页，直接部署到网页中，如著名的[Hugo academic](https://academic-demo.netlify.app/)，许多学者基于此创建了自己的学术主页。也就是说，不需要懂任何前端知识，就能实现网站内容的创作与运维。

[Typora](https://www.typora.io/)是一款简单优雅且强大的Markdown文本编辑器，能将代码与预览一体化，通过极简的编辑模式，快速实现文档的编写与排版。熟悉了快捷键及相关代码后，可以实现手不离键盘快速产出文档内容。

[PicGo](https://molunerfinn.com/PicGo/)由北邮研究生[molunerfinn](https://molunerfinn.com/)基于前端框架electron-vue开发，是一款免费开源的图床工具，能够快速长传图片并获取图片的URL，配合Typora使用有非常友好的体验。

## 环境

系统：Ubuntu Ubuntu 20.04.2 LTS

软件：Typora(Markdown文本编辑器)、PicGo(图片上传工具)、Node.js v14.17.5(javaScrpit运行环境)

## 步骤

首先要保证上述环境已经配置好，以下是在linux环境中配置的，Windows还要更加方便一点。

### 安装PicGo

在PicGo的github仓库里面下载最新的release[Molunerfinn/PicGo/releases](https://github.com/Molunerfinn/PicGo/releases)，目前最新的版本为[2.3.0-beta.8](https://github.com/Molunerfinn/PicGo/releases/tag/v2.3.0-beta.8)，下载Linux系统版本的软件[PicGo-2.3.0-beta.8.AppImage](https://github.com/Molunerfinn/PicGo/releases/download/v2.3.0-beta.8/PicGo-2.3.0-beta.8.AppImage)。由于不是从linux系统官方软件仓库里面下载的程序，现在还不能执行，则需更改程序的权限。运行以下命令：

```shell
 $ chmod +x PicGo-2.3.0-beta.8.AppImage
```

------

运行后，再运行命令`ls -l`，可以发现文件的权限

> -rw-rw-r-- 1 george george 68012921 Aug 21 16:08 PicGo-2.3.0-beta.8.AppImage    (更改权限前)
>
> -rwxrwxr-x 1 george george  68012921 Aug 21 16:08  PicGo-2.3.0-beta.8.AppImage

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gitee.com/georgegou/gravitychina/raw/picture/linux%E6%96%87%E4%BB%B6%E6%9D%83%E9%99%90%E8%A7%A3%E9%87%8A.jpeg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">linux文件权限解释</div>
</center>

rw分别代表读写，x代表执行程序，我们通过命令增加了程序的执行条件，随后运行程序`./PicGo-2.3.0-beta.8.AppImage `

右键点击图像，打开详细窗口，如下所示

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gitee.com/georgegou/gravitychina/raw/picture/202108221543777.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">PicGo程序主页</div>
</center>

点击`插件设置`，搜索`gitee`，安装`gitee-uploader`插件，目前为1.1.2版本，注意安装前请安装`node.js`。随后点击`图床设置`，点击`gitee`设置，主要配置的参数如下

> repo[username/reponame] example: george/picturebed
>
> branch[master/or anyelse]
>
> token[a line of code]: 非常重要，登录[gitee官网](https://gitee.com/)，注册账号，点击头像，然后点击`setting`，在`security settings`里面，有一个`Personal access tokens`，点击新建一个Token，勾选`Projects`，然后`Commit`即可

### 配置Typora

PicGo配置完成后，在Typora软件内依次点击 `File`-`Preference` -`images`，在`image upload setting`中点击上传服务器为`PicGo(app)`配置程序环境变量即可。注意在英文界面不显示`PicGo(app)`，切换到中文界面即可。您也可以选择配置`PicGo-core`,按照`App`的方式修改配置文件即可。

<center> <img src="https://gitee.com/georgegou/gravitychina/raw/picture/202108221601331.png" width="100%" height="10%" /> 上传成功 </center>

## 结语

Markdown和html无缝衔接，我们仅仅通过Markdown书写文本，即可非常方便地准换为html，发布到自己的网站或者博客上去。

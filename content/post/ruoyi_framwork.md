---
title: "若依框架简易指南"
date: 2022-03-04T16:28:31+08:00
lastmod: 2022-03-04T16:28:31+08:00
draft: false
keywords: []
description: "若依框架简易指南"
tags: [Ruoyi]
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

<!--more-->
若依框架是一个基于SpringBoot的权限管理系统
# 环境准备

```shell
JDK >= 1.8  #Java 运行环境
Mysql >= 5.7 #数据库
Maven >= 3.0 #Java包管理工具
Redis = 6.2.6 #本文使用的最新稳定版  
Node = 16.14.0 #本文使用的最新稳定版
```

# 运行系统

1、克隆项目到本地 git clone https://gitee.com/y_project/RuoYi-Vue \
2、导入项目到 Idea 软件，Maven会自动下载依赖 \
3、修改必要配置 
```shell
主要修改两个配置文件
1、ruoyi-admin/src/main/resources/application.yml #配置Redis、服务器端口、日志、Mybatis等参数
2、application-druid.yml #配置主从数据库信息（名称、用户、密码等）
```
4、启动后台服务 \
$\quad$ 打开文件 [ruoyi-admin/src/main/java/com/ruoyi/RuoYiApplication.java]() , 运行主函数即可[^1]
```shell
(♥◠‿◠)ﾉﾞ  若依启动成功   ლ(´ڡ`ლ)ﾞ  
 .-------.       ____     __        
 |  _ _   \      \   \   /  /    
 | ( ' )  |       \  _. /  '       
 |(_ o _) /        _( )_ .'         
 | (_,_).' __  ___(_ o _)'          
 |  |\ \  |  ||   |(_,_)'         
 |  | \ `'   /|   `-'  /           
 |  |  \    /  \      /           
 ''-'   `'-'    `-..-'              
#启动成功界面
```


# 引用来源 {#references .no-underline}


[^1]:  http://doc.ruoyi.vip/ruoyi/document/hjbs.html
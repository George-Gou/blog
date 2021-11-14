---
title: "CentOS 7搭建深度学习环境"
date: 2021-11-04T15:37:06+08:00
lastmod: 2021-11-04T15:37:06+08:00
draft: false
keywords: []
description: "本文介绍如何使用CentOS 7搭建深度学习环境"
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

## 写在前面

最近要做深度学习方面的应用，考虑到云平台服务虽然方便，但是价格太贵，最便宜的亚马逊（AWS）对于学生党来说也比较贵，这里就用实验室的机器搭建深度学习集群。

## 1、系统安装

这里我选择的是 CentOS 7版本，这也是许多企业的服务器部署环境，因为其稳定。如果选择 Ubuntu 等发行版可能用了一段时间就会出各种问题，如重启进不了系统，或者崩溃等问题。这里我推荐使用 Linux Server 最小的安装版本，不需要使用到桌面环境。

- 从[清华源](https://mirrors.tuna.tsinghua.edu.cn/centos/7.9.2009/isos/x86_64/)或者官网【bing CentOS】下载 CentOS 镜像
- 使用 [UltraISO](https://cn.ultraiso.net/xiazai.html) 制作 Linux 启动盘（推荐 8G 以上的 U 盘）
- 开启进入自己机器的 BIOS， 设置 U 盘启动
- 进入安装画面

![img](https://gitee.com/georgegou/gravitychina/raw/picture/202111041557515.png)

- 点击 Install CentOS 7 ，回车。这里可能会出现问题，比如弹出异常，进入 dracut # 命令，这是因为安装程序找不到系统卷导致的，这里需要设置
- 按住 `Tab`键，弹出配置信息，按 `E`键进入编辑，将 `vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 rd.live.check quiet`后半部分修改为 `LABEL=CENTOS`即可，当然你也可以 `cd` 到 `dev` 目录下查找 U 盘所在盘符，将其修改为 `hd:/dev/sda4` 也可以，这里看具体的盘符。

## 2、安装 NVIDA 组件

GPU 相比 CPU 有更高算力，是机器学习应用的基础设施。CPU的指令我们都非常熟悉，我们可以轻松的开发 CPU 程序，如今 GPU 横空出世，CUDA 是一种由NVIDIA推出的通用[并行计算](https://baike.baidu.com/item/并行计算/113443)架构，我们可以利用其提供的接口开发并行程序。

## 安装CUDA

https://developer.nvidia.com/cuda-10.0-download-archive

- CUDA的版本适配非常重要。目前TensorFlow2.0在CUDA上最稳妥的选择是10.0，如果选择10.1以上，有机率不识别GPU

![image-20211104174613755](https://gitee.com/georgegou/gravitychina/raw/picture/202111041746094.png)

按照官网给的安装命令安装

```shell
Installation Instructions:
`sudo rpm -i cuda-repo-rhel7-10-0-local-10.0.130-410.48-1.0-1.x86_64.rpm`
`sudo yum clean all`
`sudo yum install cuda`
```

## 安装CuDNN

CuDNN是 CUDA 和 DNN 的结合，利用 CUDA 搭配深度学习任务的框架

下载 CuDNN 需要加入开发这社区，推荐采用微信注册，在[CuDNN](https://developer.nvidia.com/rdp/form/cudnn-download-survey)仓库里面下载 Runtime 版本

![image-20211104175233436](https://gitee.com/georgegou/gravitychina/raw/picture/202111041752116.png)

```shell
$ sudo rpm -ivh libcudnn7-7.6.3.30-1.cuda10.0.x86_64.rpm   #必须先安装 Runtime 环境，才能继续安装开发者版本
$ sudo rpm -ivh libcudnn7-devel-7.6.3.30-1.cuda10.0.x86_64.rpm 

```

## 安装显卡驱动

一般来说，安装最新的驱动是最佳选择。

- 去官网下载响应显卡的版本

```shell
$ lspci | grep -i nvidia # 查看显卡版本信息
#如果显示找不到指令，安装响应软件包即可
$ yum -y install pciutils
```

- 禁用自带的 nouveau 驱动程序，防止冲突

```shell
#查看内核版本与源码版本是否一致
[root@localhost include]# ls /boot | grep vmlinu
vmlinuz-0-rescue-3a38ede036ec417cad8ea0d147ea0c46
vmlinuz-3.10.0-1160.45.1.el7.x86_64
vmlinuz-3.10.0-1160.el7.x86_64
[root@localhost include]# rpm -aq |grep kernel-devel
kernel-devel-3.10.0-1160.el7.x86_64
kernel-devel-3.10.0-1160.45.1.el7.x86_64
#屏蔽 nouveau
[root@localhost include]# vim /lib/modprobe.d/dist-blacklist.conf
#blacklist nvidiafb 将该项注释掉
[root@localhost include]# echo -e "blacklist nouveau\noptions nouveau modeset=0" > /usr/lib/modprobe.d/dist-blacklist.conf
#重建  initramfs image
[root@localhost include]# mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
[root@localhost include]# dracut /boot/initramfs-$(uname -r).img $(uname -r)
#修改运行级别为文本模式
[root@localhost include]# systemctl set-default multi-user.target
#重启
[root@localhost include]# reboot
#查看nouveau是否被禁用
[root@localhost include]# lsmod | grep nouveau 
#没有返回内容，说明已经被禁用
[root@localhost include]# chmod 777 NVIDIA-Linux-x86_64-470.82.00.run 
#将文件权限设置为可执行
[root@localhost include]# ./NVIDIA-Linux-x86_64-470.82.00.run 
[root@localhost deeplearning]# nvidia-smi # 有返回内容则安装完毕
```

![image-20211104194538022](https://gitee.com/georgegou/gravitychina/raw/picture/202111041945245.png)

![image-20211104194658765](https://gitee.com/georgegou/gravitychina/raw/picture/202111041947802.png)

## 安装 TensorFlow

先安装 Anaconda，conda是 python 较为流行的包管理工具，可以建立多个虚拟环境，为各个开发项目提供隔离环境。

- 从官网下载 [Anaconda](https://www.anaconda.com/products/individual) 最新版本

```shell
[root@localhost deeplearning]# wget https://repo.anaconda.com/archive/Anaconda3-2021.05-Linux-x86_64.sh
[root@localhost deeplearning]# bash Anaconda3-2021.05-Linux-x86_64.sh
#可以自定义安装目录，并添加环境变量
[root@localhost deeplearning]# echo -e "PATH=$PATH:/home/george/deeplearning/anaconda3/bin\nexport PATH" > ~/.bashrc
[root@localhost deeplearning]# source ~/.bashrc
# 创建 tensorflow 虚拟环境
[root@localhost deeplearning]# conda create --name tf
[root@localhost deeplearning]# conda activate tf
[root@localhost deeplearning]# conda install tensorflow-gpu==2.0.0
#该命令会自动安装TensorFlow所有的依赖环境
[root@localhost deeplearning]# conda install jupyter notebook
#配置 jupyter notebook，方便远程使用网页应用编写代码和文档
[root@localhost deeplearning]# jupyter notebook --generate-config
#生成jupyter配置文件
[root@localhost deeplearning]# python
#进入python交互程序
>>> from notebook.auth import passwd
>>> passwd()
Enter password: 
Verify password: 
#这里会生成一段秘钥
(base) [george@localhost .jupyter]$ vi /home/george/.jupyterjupyter_notebook_config.py
# 修改几个配置参数
c.NotebookApp.ip = '*'
c.NotebookApp.password = 'password your created just now'
c.NotebookApp.open_browser = 'False' #默认不在本地打开窗口，因为这是服务端
```

### 端口管理

使用 iptables 工具可以方便地进行端口管理

```shell
# 关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl mask firewalld.service
# 安装 iptables
sudo yum install iptables-services -y
# 启动 iptables
systemctl enable iptables
systemctl start iptables
#查看其状态
systemctl statusiptables
```

![image-20211104210853275](https://gitee.com/georgegou/gravitychina/raw/picture/202111042108696.png)

```shell
#编辑防火墙软件
sudo vim /etc/sysconfig/iptables
#添加开放端口供服务访问
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8888 -j ACCEPT
#重启以生效
systemctl restart iptables.service
# 设置防火墙开启自启
systemctl enable iptables.service
```

## 配置 Jupyterhub 服务

由于 jupyter notebook 只能单用户登录使用，不符合实际要求，这里我们配置 jupyterhub 服务。

```shell
$ conda install JupyterHub  #conda 环境下安装jupyterhub
$ jupyterhub --generate-config   #生成配置文件                               Writing default config to: jupyterhub_config.py
c.JupyterHub.hub_ip = ''
c.JupyterHub.hub_connect_ip = '0.0.0.0'
c.JupyterHub.bind_url = 'http://0.0.0.0:8888'
c.JupyterHub.admin_users = set('george')
c.JupyterHub.ip = '192.168.1.14'
c.JupyterHub.hub_port = 8081
# 启动服务
#$ jupyterhub 
```

![image-20211114000436416](https://gitee.com/georgegou/gravitychina/raw/picture/202111140004651.png)

### 在 Kubernetes 上部署 jupyterhub 服务

[Kubernetes](https://kubernetes.io/) 简称 k8s，是一个开源的云服务后台管理工具，把服务直接打包成容器运行在服务器上，可以将环境与宿主机隔离，不影响宿主机开发环境。

![Deployment evolution](https://gitee.com/georgegou/gravitychina/raw/picture/202111051515336.svg)

具体可以参考：https://z2jh.jupyter.org/en/latest/





## 参考资料

[1]: https://zero-to-jupyterhub.readthedocs.io/en/latest/
[2]: https://www.cnblogs.com/learn-the-hard-way/p/12318980.html


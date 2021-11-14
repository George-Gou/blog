---
title: "如何利用Nginx实现反向代理"
date: 2021-09-11T11:14:05+08:00
lastmod: 2021-09-11T11:14:05+08:00
draft: false
keywords: [nginx]
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

本文主要介绍如何使用 `nginx`实现服务器端反向代理。

## 介绍

Nginx 是一个高性能的 HTTP 和反向代理 web 服务器，同时也提供了IMAP/POP3/SMTP服务。最早是由俄罗斯人 [Igor Sysoev](https://www.nginx.com/people/igor-sysoev/) 用 C 语言设计并开发的，并于 2004 年开放源代码，目前其在旧金山创办了 [Nginx 公司](https://www.nginx.com/)，主要维护该项目。Nginx 已经为超过 400 万个网站提供服务

[^1]: https://www.nginx.com/

，其主要功能为1、反向代理；2、[负载均衡](http://nginx.org/en/docs/http/load_balancing.html)；3、[HTTP 服务器](https://nginx.org/en/docs/http/configuring_https_servers.html)（静动分析）

------

要了解反向代理，首先要知道什么是代理(Proxy)。代理就是位于服务器与客户端的中间件，优化客户端与服务器之间的通信。所谓架构，其核心就是在服务端与客户端加几层中间件，实现复杂的应用场景，如高并发等。

![image.png](https://gitee.com/georgegou/gravitychina/raw/picture/202109111645129.webp)

### 正向代理

如图所示，正向代理是服务于客户端的，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。

**正向代理**可以使客户端访问它本身无法访问的服务器资源，比如 `Google`、`Youtubu`等服务，中国大陆的 ip 封禁了对外网的访问，通过 VPN 服务使客户端的 ip 地址位于国外，就可以实现科学上网。也就是说正向代理对我们是透明的，对服务端是非透明的，即服务端并不知道自己收到的是来自代理的访问还是来自真实客户端的访问。

**反向代理**服务于服务器端，其是以代理服务器的方式接受 Internet 上的连接请求，再转发给内部服务器，并将服务器的结果返回给客户端。我们可以设置一些规则来过滤掉危险的客户端请求，还可以将访问需求大的内容存在反向代理服务器端，这样就实现了负载均衡。这种方式也隐藏了真实服务器的地址，提升了服务器的安全性。

## 安装 Nginx

本文以 CentOS 系统为例，我是在 Docker 安装的 CentOS 镜像，故先输入命令进入到 CentOS 系统：

```shell
$ docker run -itd --name mycentos centos 
$ docker exec -it mycentos /bin/bash
```

运行命令

```shell
$ yum install nginx -y
```

安装 nginx

## 初识 Nginx

```shell
# 开机配置
systemctl enable nginx # 开机自动启动
systemctl disable nginx # 关闭开机自动启动

# 启动Nginx
systemctl start nginx # 启动Nginx成功后，可以直接访问主机IP，此时会展示Nginx默认页面

# 停止Nginx
systemctl stop nginx

# 重启Nginx
systemctl restart nginx

# 重新加载Nginx
systemctl reload nginx

# 查看 Nginx 运行状态
systemctl status nginx

# 查看Nginx进程
ps -ef | grep nginx

# 杀死Nginx进程
kill -9 pid # 根据上面查看到的Nginx进程号，杀死Nginx进程，-9 表示强制结束进程
```

![image-20210911173656583](https://gitee.com/georgegou/gravitychina/raw/picture/202109111737435.png)

上述命令查看 `nginx`的运行状态，可以看到是 `active`，通过 `hostname:8080`可以访问到默认页面。

![image-20210911174140718](https://gitee.com/georgegou/gravitychina/raw/picture/202109111741050.png)

## 实例测试

![How to Use Nginx as a Reverse Proxy on Ubuntu 20.04 LTS](https://gitee.com/georgegou/gravitychina/raw/picture/202109111827132.png)

上图所示的应用场景即是本次反向代理的主要应用之一。首先先要允许 80  端口能被外部访问到。

```shell
$ sudo ufw allow 80
```

随后新建一个测试环境，这里用的Conda：

```shell
$ conda create -n new_test python = XXXX
$ conda activate new_test
$ conda install httpie
```

HTTPie 是一个 HTTP 的命令行客户端，目标是让 CLI 和 web 服务之间的交互尽可能的人性化，本文通过 HTTPie 实现一个简单的 Web应用。

```shell
$ mkdir appA
$ cd appA
$ cat <<EOF >index.html
this is app A !
EOF
```

对于Python3 来说：

```
python3 -m http.server 3000 &
  Serving HTTP on 0.0.0.0 port 3000 (http://0.0.0.0:3000/) ...
```

测试appA

```shell
http http://localhost:3000
127.0.0.1 - - [24/May/2020 13:30:59] "GET / HTTP/1.1" 200 -
HTTP/1.0 200 OK
Content-Length: 16
Content-type: text/html
Date: Sun, 24 May 2020 13:30:59 GMT
Last-Modified: Sun, 24 May 2020 13:28:42 GMT
Server: SimpleHTTP/0.6 Python/2.7.18rc1

this is app A !
```

创建appB

```shell
$ mkdir appB
$ cd appB
$ cat <<EOF >index.html
this is app B !
EOF
python3 -m http.server 4000 &
Serving HTTP on 0.0.0.0 port 4000 (http://0.0.0.0:4000/) ...
```

测试appB

```shell
http http://localhost:4000
127.0.0.1 - - [12/Sep/2021 00:01:32] "GET / HTTP/1.1" 200 -
HTTP/1.0 200 OK
Content-Length: 15
Content-type: text/html
Date: Sat, 11 Sep 2021 16:01:32 GMT
Last-Modified: Sat, 11 Sep 2021 15:59:39 GMT
Server: SimpleHTTP/0.6 Python/3.9.6

this is app B!
```

修改`/etc/hosts`文件，将地址指向本机 ip 地址

### 配置 Nginx 反向代理

目前已经做好了准备工作，现在开始配置 Nginx 服务器。

```shell
$ sudo su  # 切换到超级用户
$ rm /etc/nginx/sites-enabled/default  #删除默认配置
$ touch /etc/nginx/sites-available/reverse-proxy #创建反向代理
$ ln -s /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites enabled/reverse-proxy # 创建软链接，任意修改一处会同步变化
$ vim /etc/nginx/sites-available/reverse-proxy #复制以下内容到配置文件
```

```
upstream appA {
        server 127.0.0.1:3000;
}

upstream appB {
        server 127.0.0.1:4000;
}


server {
  listen 80;
  server_name appa.domain.com;
  
  location / {
        proxy_pass http://appA;
    }
}

server {
  listen 80;
  server_name appb.domain.com;

  location / {
        proxy_pass http://appB;
    }
}
```

上述操作中，我们在上游定义了两个应用程序，一个转到3000端口，一个转到4000端口。利用 server 指令和 server_name 选项来确定子域名与上游映射关系

### 测试反向代理

```shell
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
$ sudo systemctl reload nginx
$ sudo systemctl status nginx
```

#### 测试结果：

```shell
$ http -v http://appa.domain.com
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Host: appa.domain.com
User-Agent: HTTPie/2.1.0



127.0.0.1 - - [12/Sep/2021 00:44:34] "GET / HTTP/1.0" 200 -
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 15
Content-Type: text/html
Date: Sat, 11 Sep 2021 16:44:34 GMT
Last-Modified: Sat, 11 Sep 2021 10:38:27 GMT
Server: nginx/1.18.0 (Ubuntu)

this is app A!

(makedoc) 

```

```shell
▶ http -v http://appb.domain.com
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Host: appb.domain.com
User-Agent: HTTPie/2.1.0



127.0.0.1 - - [12/Sep/2021 00:45:38] "GET / HTTP/1.0" 200 -
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 15
Content-Type: text/html
Date: Sat, 11 Sep 2021 16:45:38 GMT
Last-Modified: Sat, 11 Sep 2021 15:59:39 GMT
Server: nginx/1.18.0 (Ubuntu)

this is app B!


```

可以发现，通过指定域名能够访问到特定的应用容器。

## 结论

利用出色的开源工具 Nginx 可以轻松完成反向代理。

## 参考文献

https://blog.csdn.net/b9x__/article/details/80400697

https://juejin.cn/post/6942607113118023710

https://bitlaunch.io/blog/how-to-use-nginx-as-a-reverse-proxy-on-ubuntu-20-04/


---
title: "如何使用Docker在本地搭建Overleaf服务器"
date: 2021-08-22T15:29:46+08:00
lastmod: 2021-08-22T15:29:46+08:00
draft: false
keywords: [docker,overleaf]
description: ""
tags: [docker,overleaf]
categories: [computer config]
author: "Gravpher"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
postMetaInFooter: true
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: true
reward: true
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: true

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: true
  options: ""

sequenceDiagrams: 
  enable: true
  options: ""

---

# 介绍

相信大家在写论文中一定被Word中的格式所折磨过，如参考文献的添加、公式的编写标号、图文混排的格式编辑等，是一种所见即所得模式。而latex则是通过命令的方式编译生成pdf文件的。word的内容和格式耦合在一起，而latex则可以只用关心内容，使用模板即可创建高质量的文档。国际期刊一般都提供latex模板，这使得读者可以更加关注内容创作。但是Latex涉及到大量的命令和宏包，需要一定时间熟悉，相信我，习惯之后，你会爱上用Latex来写作。

### 安装

一般来说Latex安装分为两个步骤，一是安装[Tex Live](https://www.tug.org/texlive/)，类似于Tex的编译器，类比gcc、gfortran，二是安装编辑器，目前流行的编辑器有许多，如[Lyx](https://www.lyx.org/)、[TexStudio](http://texstudio.sourceforge.net/)、[VScode](https://code.visualstudio.com/)等。但是环境配置较为繁琐，参考以下博文[Windows](https://www.cnblogs.com/1625--H/p/11524968.html)、[linux](https://www.jianshu.com/p/4e2614064c6a)、[Macos](https://www.jianshu.com/p/50a813c8a6ea)，环境配置成功后方可进行写作。

### Overleaf

线下环境配置困难，Latex编写云服务近几年逐步出现。[Overleaf](https://www.overleaf.com/)是一个使用LaTeX进行多人协同编辑的平台，可以免费注册和使用，不用下载LaTeX软件，是最为著名的LaTeX在线协作系统。目前Overleaf已经有超过800万和3000个科研机构使用，但是国内网络问题，国内使用体验很差。如果自己有服务器的话，可以利用Docker搭建一个Overleaf社区版，也就是[sharelatex](https://www.sharelatex.com/)，其是Overleaf的一部分。

### Linux服务器搭建sharelatex服务

#### 1、Docker安装

[Docker](https://www.docker.com/)是一个由[Go](https://golang.org/)语言编写的开源的应用容器引擎，其是C/S架构，启动Docker本质上是在后台启动了一项服务，提供API供用户使用。实际上，开发者打包其应用以及依赖包到一个轻量级、可移植的容器中，可以在各个平台运行。在这个例子中，我们通过docker pull命令抓取sharelatex的镜像，开启其服务，即可以在本地创建用户，使用overleaf在线编辑器。

```shell
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
$ pip install docker-compose
# 检查是否安装成功
$ docker --version
Docker version 20.10.8, build 3967b7d
$ docker-compose --version
 docker-compose version 1.25.0, build unknown
```



#### 2、Sharelatex镜像安装

Docker[仓库](https://registry.hub.docker.com/)提供了多种常用应用的镜像，如[mogoDB数据库](https://www.mongodb.com/zh-cn)、[redis](https://redis.io/)等，本文需要用到前面两个容器和[sharelatex容器](https://hub.docker.com/u/sharelatex/)

```shell
$ sudo docker pull sharelatex/sharelatex
```

如果不想将服务器配置到根目录的话，需要修改配置文件docker-compose.yml，使用下面命令

```shell
$wget https://raw.githubusercontent.com/sharelatex/sharelatex/master/docker-compose.yml
$ sudo vim docker-compose.yml
```

可能网络不畅通，需要科学上网，下面直接附上docker-compose.yml 文件内容：

```txt
version: '2.2'
services:
    sharelatex:
        restart: always
        # Server Pro users:
        # image: quay.io/sharelatex/sharelatex-pro
        image: sharelatex/sharelatex
        container_name: sharelatex
        depends_on:
            mongo:
                condition: service_healthy
            redis:
                condition: service_started
        ports:
            - 80:80
 # 可能需要修改该端口，80端口可能被其他服务占据，如nginx或者tomcat           
        links:
            - mongo
            - redis
        volumes:
            - ~/sharelatex_data:/var/lib/sharelatex
            ########################################################################
            ####  Server Pro: Uncomment the following line to mount the docker  ####
            ####             socket, required for Sibling Containers to work    ####
            ########################################################################
            # - /var/run/docker.sock:/var/run/docker.sock
        environment:

            SHARELATEX_APP_NAME: Overleaf Community Edition

            SHARELATEX_MONGO_URL: mongodb://mongo/sharelatex

            # Same property, unfortunately with different names in
            # different locations
            SHARELATEX_REDIS_HOST: redis
            REDIS_HOST: redis

            ENABLED_LINKED_FILE_TYPES: 'url,project_file'

            # Enables Thumbnail generation using ImageMagick
            ENABLE_CONVERSIONS: 'true'

            # Disables email confirmation requirement
            EMAIL_CONFIRMATION_DISABLED: 'true'

            # temporary fix for LuaLaTex compiles
            # see https://github.com/overleaf/overleaf/issues/695
            TEXMFVAR: /var/lib/sharelatex/tmp/texmf-var

            ## Set for SSL via nginx-proxy
            #VIRTUAL_HOST: 103.112.212.22

            # SHARELATEX_SITE_URL: http://sharelatex.mydomain.com
            # SHARELATEX_NAV_TITLE: Our ShareLaTeX Instance
            # SHARELATEX_HEADER_IMAGE_URL: http://somewhere.com/mylogo.png
            # SHARELATEX_ADMIN_EMAIL: support@it.com

            # SHARELATEX_LEFT_FOOTER: '[{"text": "Powered by <a href=\"https://www.sharelatex.com\">ShareLaTeX</a> 2016"},{"text": "Another page I want to link to can be found <a href=\"here\">here</a>"} ]'
            # SHARELATEX_RIGHT_FOOTER: '[{"text": "Hello I am on the Right"} ]'

            # SHARELATEX_EMAIL_FROM_ADDRESS: "team@sharelatex.com"

            # SHARELATEX_EMAIL_AWS_SES_ACCESS_KEY_ID:
            # SHARELATEX_EMAIL_AWS_SES_SECRET_KEY:

            # SHARELATEX_EMAIL_SMTP_HOST: smtp.mydomain.com
            # SHARELATEX_EMAIL_SMTP_PORT: 587
            # SHARELATEX_EMAIL_SMTP_SECURE: false
            # SHARELATEX_EMAIL_SMTP_USER:
            # SHARELATEX_EMAIL_SMTP_PASS:
            # SHARELATEX_EMAIL_SMTP_TLS_REJECT_UNAUTH: true
            # SHARELATEX_EMAIL_SMTP_IGNORE_TLS: false
            # SHARELATEX_EMAIL_SMTP_NAME: '127.0.0.1'
            # SHARELATEX_EMAIL_SMTP_LOGGER: true
            # SHARELATEX_CUSTOM_EMAIL_FOOTER: "This system is run by department x"

            ################
            ## Server Pro ##
            ################

            # SANDBOXED_COMPILES: 'true'

            # SANDBOXED_COMPILES_SIBLING_CONTAINERS: 'true'
            # SANDBOXED_COMPILES_HOST_DIR: '/var/sharelatex_data/data/compiles'
            # SYNCTEX_BIN_HOST_PATH: '/var/sharelatex_data/bin/synctex'

            # DOCKER_RUNNER: 'false'

            ## Works with test LDAP server shown at bottom of docker compose
            # SHARELATEX_LDAP_URL: 'ldap://ldap:389'
            # SHARELATEX_LDAP_SEARCH_BASE: 'ou=people,dc=planetexpress,dc=com'
            # SHARELATEX_LDAP_SEARCH_FILTER: '(uid={{username}})'
            # SHARELATEX_LDAP_BIND_DN: 'cn=admin,dc=planetexpress,dc=com'
            # SHARELATEX_LDAP_BIND_CREDENTIALS: 'GoodNewsEveryone'
            # SHARELATEX_LDAP_EMAIL_ATT: 'mail'
            # SHARELATEX_LDAP_NAME_ATT: 'cn'
            # SHARELATEX_LDAP_LAST_NAME_ATT: 'sn'
            # SHARELATEX_LDAP_UPDATE_USER_DETAILS_ON_LOGIN: 'true'

            # SHARELATEX_TEMPLATES_USER_ID: "578773160210479700917ee5"
            # SHARELATEX_NEW_PROJECT_TEMPLATE_LINKS: '[ {"name":"All Templates","url":"/templates/all"}]'


            # SHARELATEX_PROXY_LEARN: "true"

    mongo:
        restart: always
        image: mongo:4.0
        container_name: mongo
        expose:
            - 27017
        volumes:
            - ~/mongo_data:/data/db
        healthcheck:
            test: echo 'db.stats().ok' | mongo localhost:27017/test --quiet
            interval: 10s
            timeout: 10s
            retries: 5

    redis:
        restart: always
        image: redis:5
        container_name: redis
        expose:
            - 6379
        volumes:
            - ~/redis_data:/data

    # ldap:
    #    restart: always
    #    image: rroemhild/test-openldap
    #    container_name: ldap
    #    expose:
    #        - 389

    # See https://github.com/jwilder/nginx-proxy for documentation on how to configure the nginx-proxy container,
    # and https://github.com/overleaf/overleaf/wiki/HTTPS-reverse-proxy-using-Nginx for an example of some recommended
    # settings. We recommend using a properly managed nginx instance outside of the Overleaf Server Pro setup,
    # but the example here can be used if you'd prefer to run everything with docker-compose

    # nginx-proxy:
    #     image: jwilder/nginx-proxy
    #     container_name: nginx-proxy
    #     ports:
    #       #- "80:80"
    #       - "443:443"
    #     volumes:
    #       - /var/run/docker.sock:/tmp/docker.sock:ro
    #       - /home/sharelatex/tmp:/etc/nginx/certs
```

可能需要修改该端口，80端口可能被其他服务占据，如nginx或者tomcat。如果你不想redis和mongdb存储在根目录的话，需要修改其路径。

#### 3、启动sharelatex镜像

```shell
sudo docker-compose up -d
```

#### 4、安装texlive

```shell
$ tlmgr option repository https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet/
$ docker exec -it sharelatex bash
$ tlmgr update --self --all
$ tlmgr install scheme-full &
```

一共4090个包，需要等几个小时

#### 5、启动sharelatex服务

```shell
$ sudo docker-compose up 
```

浏览器访问：http://hostname:port/launchpad（hostname为本机地址，如127.0.0.1，port为.yml文件配置的端口号。

注册一个admin账号，可以开始愉快的写作了，done！

<img src="https://cdn.jsdelivr.net/gh/George-Gou/PictureBed@master/2022/202108211802088.png" alt="Screenshot from 2021-08-14 23-10-21" style="zoom: 25%;" />

## 结语

本文是在下列环境下创建的，在win和Mac应该也能够创建成功。总体使用上来说还是挺不错的，特别是自己环境下有服务器的，能够极大地节省安装时间，专心投入生产。

```shell
$ Linux version 5.8.0-63-generic (buildd@lgw01-amd64-035) (gcc (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0, GNU ld (GNU Binutils for Ubuntu) 2.34) #71~20.04.1-Ubuntu SMP Thu Jul 15 17:46:08 UTC 2021
```

## 参考

[1] [https://github.com/overleaf/overleaf/wiki/Quick-Start-Guide](https://links.jianshu.com/go?to=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fgithub.com%2Foverleaf%2Foverleaf%2Fwiki%2FQuick-Start-Guide)




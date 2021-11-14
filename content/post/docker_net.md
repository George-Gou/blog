---
title: "Docker网络探秘"
date: 2021-11-03T16:48:18+08:00
lastmod: 2021-11-03T16:48:18+08:00
draft: false
keywords: []
description: ""
tags: [docker]
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

## Docker网络

Docker 是一个及其轻量化的应用容器，能够轻松将开发环境与应用打包至镜像进行发布，能达到“Develop Once，Run Anywhere”的效果。Docker 容器跟宿主机、不同容器之间究竟是如何通信的呢？这是Docker的核心内容。

### Docker0

```shell
(base) george@george ~/test$ ifconfig  
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:9eff:fe6d:8c1c  prefixlen 64  scopeid 0x20<link>
        ether 02:42:9e:6d:8c:1c  txqueuelen 0  (Ethernet)
        RX packets 134541  bytes 14183525 (14.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 501775  bytes 629260746 (629.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

开启了Docker之后，发现网卡中多了一个 Docker0 ，这可以充当一个路由器的功能，使得所有容器通过网络桥接的方式相互通信。如我们启动其他容器后：

```
docker1：tomcat01 172.17.0.2
docker2：tomcat02 172.17.0.3
```

![img](https://gitee.com/georgegou/gravitychina/raw/picture/docker%E7%BD%91%E7%BB%9C)

如果容器采用 host 模式，这种情况下容器与宿主机共享网络，其余情况通过 veth-pair 进行桥接，进而实现容器间的通信。

### 如何使用Docker部署 redis 集群

在如今的大数据与机器学习时代，几乎所有的服务都跑在集群上的。相比单机服务器来说，集群能够发挥负载均衡、高可用等特性。下面将演示如何在 docker 容器中部署 6 个redis 集群。

1、创建子网

```shell
$ docker network create redis --subnet 172.38.0.0/16
通过构建子网，使得redis集群处于同一子网
```

2、创建redis集群的配置文件

```shell
for port in $(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
sppendonly yes
EOF       
done 
```

3、依次启动6个redis集群

```shell
$ docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /redis_cluster/mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
依次替换 端口号 redis集群名称 ip地址，启动集群
```

4、进入集群

```shell
$ docker exec -it redis-1 /bin/bash
/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1

```


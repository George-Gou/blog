---

title: "Cetos7 下安装oracle数据库"
date: 2021-11-18T17:28:12+08:00
lastmod: 2021-11-18T17:28:12+08:00
draft: false
keywords: [oracel]
description: ""
tags: []
categories: []
author: "Gravpher"
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

<!--more-->

Oracel数据库是目前为止大企业中不可或缺的数据存储与管理方案，由于其昂贵的费用和笨重的软件体系让许多小公司望而却步，转向使用免费的MySQL社区版的数据库，这样的好处是能够节省很大一笔支出，但需要花更多精力维护和备份数据库。由于本单位使用Oracel数据库，为了学习，本人在CentOS 7 上安装Oracel数据库学习。

## 1、安装依赖包

```shell
$ yum -y install binutils \
compat-libstdc++-33 \
elfutils-libelf \
elfutils-libelf-devel \
expat \
gcc \
gcc-c++ \
glibc \
glibc-common \
glibc-devel \
glibc-headers \
libaio \
libaio-devel \
libgcc \
libstdc++ \
libstdc++-devel \
make \
pdksh \
sysstat \
unixODBC \
unixODBC-devel
#安装 pdksh 包，不然安装过程会提示
$ wget -O /tmp/pdksh-5.2.14-37.el5_8.1.x86_64.rpm http://vault.centos.org/5.11/os/x86_64/CentOS/pdksh-5.2.14-37.el5_8.1.x86_64.rpm
$ cd /tmp/
rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm
```

## 2、创建oracel用户组

```shell
$ groupadd oinstall
$ groupadd dba
$ groupadd asmadmin
$ groupadd asmdba
$ useradd -g oinstall -G dba,asmdba oracle -d /home/oracle
$ passwd oracle
# 本机ip地质映射
$ vim /etc/hosts
192.168.1.14 centos-oracle 
$ PING -c (192.168.1.14) 56(84) bytes of data.
64 bytes from -c (192.168.1.14): icmp_seq=1 ttl=64 time=0.022 ms
64 bytes from -c (192.168.1.14): icmp_seq=2 ttl=64 time=0.021 ms
64 bytes from -c (192.168.1.14): icmp_seq=3 ttl=64 time=0.022 ms
# 测试 
# 创建 oracel 安装目录
$ mkdir -p /home/oracle//db/app/oracle/product/11.2.0
$ mkdir /home/oracle/db/app/oracle/oradata
$ mkdir /home/oracle/db/app/oracle/inventory
$ mkdir /home/oracle/db/app/oracle/fast_recovery_area
$ chown -R oracle:oinstall /home/oracle/db/app/oracle
$ chmod -R 775 /home/oracle/db/app/oracle
# 配置环境变量
$ su - oracle
$ vim .bash_profile
umask 022
export ORACLE_HOSTNAME=centos-oracle
export ORACLE_BASE=/home/oracle/db/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/
export ORACLE_SID=ogeorge
export PATH=.:$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:$ORACLE_HOME/jdk/bin:$PATH
export LC_ALL="en_US"
export LANG="en_US"
export NLS_LANG="AMERICAN_AMERICA.ZHS16GBK"
export NLS_DATE_FORMAT="YYYY-MM-DD HH24:MI:SS"
$ cd db/
$ unzip linux.x64_11gR2_database_1of2.zip -d /home/oracle/db
$ unzip linux.x64_11gR2_database_2of2.zip -d /home/oracle/db
$ mkdir db/etc/
$ cp db/database/response/* db/etc/
#修改安装配置文件，参数配置如下
oracle.install.option=INSTALL_DB_SWONLY
DECLINE_SECURITY_UPDATES=true
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/home/oracle/db/app/oracle/inventory
SELECTED_LANGUAGES=en,zh_CN
ORACLE_HOSTNAME=centos-oracle
ORACLE_HOME=/home/oracle/db/app/oracle/product/11.2.0
ORACLE_BASE=/home/oracle/db/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=true
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=dba
```

## 3、开启安装 oracle

```shell
## 开启安装 oracel
$ su - oracle
$ ./runInstaller -silent -ignorePrereq -responseFile ./runInstaller -silent -ignorePrereq -responseFile /home/oracle/db/etc/db_install.rsp
[oracle@localhost database]$ ./runInstaller -silent -ignorePrereq -responseFile /db/etc/db_install.rsp
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 120 MB.   Actual 36093 MB    Passed
Checking swap space: 0 MB available, 150 MB required.    Failed <<<<

Some requirement checks failed. You must fulfill these requirements before

continuing with the installation,


Exiting Oracle Universal Installer, log for this session can be found at /tmp/OraInstall2021-11-18_10-10-22PM/installActions2021-11-18_10-10-22PM.log
# 出现 swap 空间不足
# 首先查看 swap 存在吗
$ swapon -s #返回为空，则表示没有 swap 文件
$ df -hal
# 创建并允许 Swap 文件下面使用 dd 命令来创建 Swap 文件。检查返回的信息，还剩余足够的硬盘空间即可
$ dd if=/dev/zero of=/swapfile bs=1024 count=512k
524288+0 records in
524288+0 records out
536870912 bytes (537 MB) copied, 1.00887 s, 532 MB/s
$ mkswap /swapfile
Setting up swapspace version 1, size = 524284 KiB
no label, UUID=d1990601-8abe-4218-a3dc-7b262618991f
$ swapon /swapfile

# 再开始安装
[oracle@-c database]$ ./runInstaller -silent -ignorePrereq -responseFile /home/oracle/db/etc/db_install.rsp
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 120 MB.   Actual 37938 MB    Passed
Checking swap space: must be greater than 150 MB.   Actual 511 MB    Passed
Preparing to launch Oracle Universal Installer from /tmp/OraInstall2021-11-19_01-39-35AM. Please wait ...[oracle@-c database]$ [WARNING] [INS-32055] The Central Inventory is located in the Oracle base.
   CAUSE: The Central Inventory is located in the Oracle base.
   ACTION: Oracle recommends placing this Central Inventory in a location outside the Oracle base directory.
[WARNING] [INS-32055] The Central Inventory is located in the Oracle base.
   CAUSE: The Central Inventory is located in the Oracle base.
   ACTION: Oracle recommends placing this Central Inventory in a location outside the Oracle base directory.
You can find the log of this install session at:
 /home/oracle/db/app/oracle/inventory/logs/installActions2021-11-19_01-39-35AM.log

$ su root
$ sh /home/oracle/db/app/oracle/inventory/orainstRoot.sh
$ sh /home/oracle/db/app/oracle/product/11.2.0/root.sh
Check /home/oracle/db/app/oracle/product/11.2.0/install/root_-c_2021-11-19_01-44-04.log for the output of root script
```

![image-20211119151840998](https://cdn.jsdelivr.net/gh/George-Gou/PictureBed@master/2022/202111191518402.png)

## 4、配置静默监听

```shell
[oracle@-c bin]$ su - oracle
[oracle@-c bin]$ netca /silent /responsefile /home/oracle/db/etc/netca.rsp

Parsing command line arguments:
    Parameter "silent" = true
    Parameter "responsefile" = /home/oracle/db/etc/netca.rsp
Done parsing command line arguments.
Oracle Net Services Configuration:
Profile configuration complete.
Listener "LISTENER" already exists.
Oracle Net Services configuration successful. The exit code is 0
[oracle@-c bin]$ netstat -tnulp | grep 1521 #查看监听的端口
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 :::1521                 :::*                    LISTEN      2861/tnslsnr
```

## 5、静默创建数据库

```shell
$ su root
$ vi /home/oracleGDBNAME = "ogeorge"
SID = "ogeorge"
SYSPASSWORD = "oracle"
SYSTEMPASSWORD = "oracle"
SYSMANPASSWORD = "oracle"
DBSNMPPASSWORD = "oracle"
DATAFILEDESTINATION =/home/oracle/db/app/oracle/oradata
RECOVERYAREADESTINATION=/home/oracle/db/app/oracle/fast_recovery_area
CHARACTERSET = "AL32UTF8"
TOTALMEMORY = "1638"/db/etc/dbca.rsp //打开文件后可通过搜索将“=”右边参数值补齐

# 执行静默数据库
$ su - oracle
$ dbca -silent -responseFile /home/oracle/db/etc/dbca.rsp #提示输入系统密码
Copying database files                                                                                             
1% complete                                                                                                        
3% complete                                                                                                        
11% complete                                                                                                       
18% complete                                                                                                       
26% complete                                                                                                       
37% complete                                                                                                       
Creating and starting Oracle instance                                                                              
40% complete                                                                                                       
45% complete                                                                                                       
50% complete                                                                                                       
55% complete                                                                                                       
56% complete                                                                                                       
60% complete                                                                                                       
62% complete                                                                                                       
Completing Database Creation                                                                                       
66% complete                                                                                                       
70% complete                                                                                                       
73% complete                                                                                                       
85% complete
96% complete
100% complete
Look at the log file "/home/oracle/db/app/oracle/cfgtoollogs/dbca/ogeorge11g/ogeorge11g.log" for further details.
[oracle@-c bin]$ tail /home/oracle/db/app/oracle/cfgtoollogs/dbca/ogeorge11g/ogeorge11g.log
DBCA_PROGRESS : 70%
DBCA_PROGRESS : 73%
DBCA_PROGRESS : 85%
DBCA_PROGRESS : 96%
DBCA_PROGRESS : 100%
Database creation complete. For details check the logfiles at:
 /home/oracle/db/app/oracle/cfgtoollogs/dbca/ogeorge11g.
Database Information:
Global Database Name:ogeorge11g.us.oracle.com
System Identifier(SID):ogeorge
# 查看oracle实例进程
[oracle@-c bin]$ ps -ef | grep ora_ | grep -v grep 
oracle   28293     1  0 04:31 ?        00:00:00 ora_pmon_DBUA0
oracle   28295     1  0 04:31 ?        00:00:00 ora_vktm_DBUA0
oracle   28299     1  0 04:31 ?        00:00:00 ora_gen0_DBUA0
oracle   28301     1  0 04:31 ?        00:00:00 ora_diag_DBUA0
oracle   28303     1  0 04:31 ?        00:00:00 ora_dbrm_DBUA0
oracle   28305     1  0 04:31 ?        00:00:00 ora_psp0_DBUA0
oracle   28307     1  0 04:31 ?        00:00:00 ora_dia0_DBUA0
oracle   28309     1  0 04:31 ?        00:00:00 ora_mman_DBUA0
oracle   28311     1  0 04:31 ?        00:00:00 ora_dbw0_DBUA0
oracle   28313     1  0 04:31 ?        00:00:00 ora_lgwr_DBUA0
oracle   28315     1  0 04:31 ?        00:00:00 ora_ckpt_DBUA0
oracle   28317     1  0 04:31 ?        00:00:00 ora_smon_DBUA0
oracle   28319     1  0 04:31 ?        00:00:00 ora_reco_DBUA0
oracle   28321     1  0 04:31 ?        00:00:00 ora_mmon_DBUA0
oracle   28323     1  0 04:31 ?        00:00:00 ora_mmnl_DBUA0
oracle   32107     1  0 04:38 ?        00:00:00 ora_pmon_ogeorge
oracle   32109     1  0 04:38 ?        00:00:00 ora_vktm_ogeorge
oracle   32113     1  0 04:38 ?        00:00:00 ora_gen0_ogeorge
oracle   32115     1  0 04:38 ?        00:00:00 ora_diag_ogeorge
oracle   32117     1  0 04:38 ?        00:00:00 ora_dbrm_ogeorge
oracle   32119     1  0 04:38 ?        00:00:00 ora_psp0_ogeorge
oracle   32121     1  0 04:38 ?        00:00:00 ora_dia0_ogeorge
oracle   32123     1  0 04:38 ?        00:00:00 ora_mman_ogeorge
oracle   32125     1  0 04:38 ?        00:00:00 ora_dbw0_ogeorge
oracle   32127     1  0 04:38 ?        00:00:00 ora_lgwr_ogeorge
oracle   32129     1  0 04:38 ?        00:00:00 ora_ckpt_ogeorge
oracle   32131     1  0 04:38 ?        00:00:00 ora_smon_ogeorge
oracle   32133     1  0 04:38 ?        00:00:00 ora_reco_ogeorge
oracle   32135     1  0 04:38 ?        00:00:00 ora_mmon_ogeorge
oracle   32137     1  0 04:38 ?        00:00:00 ora_mmnl_ogeorge
oracle   32139     1  0 04:38 ?        00:00:00 ora_d000_ogeorge
oracle   32141     1  0 04:38 ?        00:00:00 ora_s000_ogeorge
oracle   32293     1  0 04:38 ?        00:00:00 ora_qmnc_ogeorge
oracle   32308     1  0 04:38 ?        00:00:00 ora_cjq0_ogeorge
oracle   32382     1  0 04:38 ?        00:00:00 ora_q000_ogeorge
oracle   32384     1  0 04:38 ?        00:00:00 ora_q001_ogeorge

#查看监听状态
[oracle@-c bin]$ lsnrctl status 

LSNRCTL for Linux: Version 11.2.0.1.0 - Production on 19-NOV-2021 04:47:55

Copyright (c) 1991, 2009, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 11.2.0.1.0 - Production
Start Date                19-NOV-2021 02:17:27
Uptime                    0 days 2 hr. 30 min. 30 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /home/oracle/db/app/oracle/product/11.2.0/network/admin/listener.ora
Listener Log File         /home/oracle/db/app/oracle/product/11.2.0/log/diag/tnslsnr/-c/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=-c)(PORT=1521)))
Services Summary...
Service "DBUA0" has 1 instance(s).
  Instance "DBUA0", status BLOCKED, has 1 handler(s) for this service...
Service "ogeorge11g.us.oracle.com" has 1 instance(s).
  Instance "ogeorge", status READY, has 1 handler(s) for this service...
Service "ogeorgeXDB.us.oracle.com" has 1 instance(s).
  Instance "ogeorge", status READY, has 1 handler(s) for this service...
The command completed successfully

```

## 6、**登录sqlplus，查看实例状态**

```shell
[oracle@-c bin]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.1.0 Production on Fri Nov 19 04:48:35 2021

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> 
SQL> select status from v$instance;
select status from v$instance
ERROR at line 1:
ORA-01034: ORACLE not available
Process ID: 0
Session ID: 0 Serial number: 0
$ cp init.ora.101920214378 /home/oracle/db/app/oracle/product/11.2.0/dbs/initogeorge.ora

SQL> startup
ORA-01078: failure in processing system parameters
LRM-00109: could not open parameter file '/home/oracle/db/app/oracle/product/11.2.0/dbs/initogeorge.ora'
#怎么解决上述  LRM-00109 错误?

这是由于初始化文件不存在，或找不到，可能是启动 oracle 实例时配置文件没有写对，SID一定要和实例名字相同

SQL> select status from v$instance;

STATUS
------------------------
OPEN

# 查看数据库编码
SQL> select userenv('language') from dual;

USERENV('LANGUAGE')
--------------------------------------------------------------------------------
AMERICAN_AMERICA.AL32UTF8
# 查找数据库版本信息
SQL> select * from v$version;

BANNER
--------------------------------------------------------------------------------
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
PL/SQL Release 11.2.0.1.0 - Production
CORE	11.2.0.1.0	Production
TNS for Linux: Version 11.2.0.1.0 - Production
NLSRTL Version 11.2.0.1.0 - Production

#激活scott用户
SQL> alter user scott account unlock; 

User altered.
SQL> alter user scott identified by tiger;

User altered.
SQL> select username from all_users;

USERNAME
------------------------------------------------------------
SCOTT
OWBSYS_AUDIT
OWBSYS
APEX_030200
APEX_PUBLIC_USER
FLOWS_FILES
MGMT_VIEW
SYSMAN
SPATIAL_CSW_ADMIN_USR
SPATIAL_WFS_ADMIN_USR
MDDATA
```

## 7、远程访问Oracle数据库
```shell
#开放访问端口
[root@-c]~# firewall-cmd --zone=public --add-port=1521/tcp --permanent
success
[root@-c]~# firewall-cmd --reload
success
```

**安装 Navicat 数据可视化工具：**

![image-20211119234218799](https://cdn.jsdelivr.net/gh/George-Gou/PictureBed@master/2022/202111192346963.png)

![image-20211119234500605](https://cdn.jsdelivr.net/gh/George-Gou/PictureBed@master/2022/202111192346769.png)

![image-20211119234601864](https://cdn.jsdelivr.net/gh/George-Gou/PictureBed@master/2022/202111192346505.png)

配置 ip 地址、端口号、用户名、密码即可连接。
## 参考文献

[1] cnblogs.com/wanderwei/p/11458979.html


---
typora-root-url: pictures
---

### 任务背景

**今天运维接到2张ITSM工单，具体内容为：**

1. 测试人员需要运维人员协助安装一台Linux6.5操作系统，配置好基础环境，安装MySQL5.7的最新版本数据库用于测试。
2. 开发人员需要运维人员协助将原有的测试环境中的samba软件升级到最新版本并且安装一个ftp服务的软件。

### 任务要求

#### 任务1：

1. 最小化安装Centos6.5操作系统，配置好基本的网络环境（静态IP）、主机名为testdb.itcast.cc
2. 安装MySQL数据库，版本为5.7.23
3. 配置好本地yum

#### 任务2：

1. 开发环境的操作系统版本为Centos6.5，最小化安装。
2. 开发环境中原有samba服务软件包：samba-3.6.9。现需要将服该务的软件升级为：==samba-3.6.23==版本，并且安装vsftpd软件。

### 课程目标

- 能够根据需求获取到适合当前系统平台的软件包（二进制和源码包）
- 能够使用rpm工具安装rpm包
- 能够根据需求安装源码包

- 理解yum源如何解决依赖关系
- 掌握本地和网络yum源的配置（==重点==）
- 熟练使用yum工具安装软件包

### 涉及知识点

- 操作系统最小化安装（旧知识点）
- rpm工具使用（旧知识点）
- yum源配置（本地和网络源）**新知识点**
- yum工具使用  **新知识点**

### 具体实施

#### 任务1：

#####1. 最小化安装Linux6.5系统

~~~powershell
思路：
1. 根据对方需求
a. 最小化安装
b. 配置基本网络环境（静态IP、配置好主机名）
c. 安装5.7版本的最新数据库
2. 思考：
内存、swap、分区、基本软件包
使用lvm逻辑卷管理，方便日后扩容：
/boot	300M
swap	1024M
/		10g	至少
/data		剩余空间

补充：Linux的组成
内核、应用程序、文件系统、shell(命令解释器)
/etc	/boot   /var	/home	 /dev	/root
yum rpm 安装mysql的数据目录：/var/lib/mysql
/dev/sda1
/dev/sr0
/dev/cdrom

~~~

##### 2. 静态IP和主机名配置

~~~powershell
NAT模式：自动获取IP，可以访问互联网
[root@localhost network-scripts]# cat ifcfg-eth0
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=dhcp

host-only模式：配置静态IP，不可以访问互联网
[root@localhost network-scripts]# cat ifcfg-eth1
DEVICE=eth1
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=none
IPADDR=10.1.1.1
NETMASK=255.255.255.0

配置主机名：
临时：
hostname	testdb.itcast.cc
永久：
vi /etc/sysconfig/network
...
HOSTNAME=testdb.itcast.cc

将IP和主机名一一对应起来保存到/etc/hosts文件中：

[root@testdb ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.159.152	testdb.itcast.cc testdb
10.1.1.1	testdb.itcast.cc testdb
注意：
前两行内容不要动！！！

补充：
1. 关闭防火墙
临时关闭：
service iptables stop
开机不要自启动：
chkconfig iptables off
# chkconfig --list|grep iptables		查看是否关闭
iptables       	0:off	1:off	2:off	3:off	4:off	5:off	6:off

2. 关闭selinux
三种模式：
enforcing：强制模式
permissive：警告模式
disabled：关闭
临时放到警告模式：
[root@MissHou ~]# setenforce 0
查看状态：
[root@MissHou ~]# getenforce 
Permissive
永久关闭：
[root@MissHou ~]# vi /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled		改为disabled
...
注意：关闭需要重启系统才生效
~~~

##### 3. 安装MySQL 5.7.23版本

~~~powershell
思路：
1. 优先到系统的光盘里找	（没有）
2. 软件的官方网站https://dev.mysql.com/downloads/mysql/5.7.html#downloads（找到）
3. 安装
4. 确保成功安装

步骤：
挂载光盘到本机的/mnt目录下
[root@testdb soft]# lsblk 
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                          11:0    1  4.2G  0 rom  /mnt
...
删掉版本不一致的依赖
 ~]# rpm -qa | grep -i mysql
mysql-libs-5.1.73-7.el6.x86_64
 ~]# rpm -e --nodeps mysql-libs-5.1.73-7.el6.x86_64
 ~]# rpm -qa | grep -i mysql

[root@testdb soft]# mount -o ro /dev/sr0 /mnt

[root@testdb ~]# mkdir /soft
[root@testdb ~]# cd /soft/
[root@testdb soft]# ls
mysql-5.7.23-1.el6.x86_64.rpm-bundle.tar
[root@testdb soft]# tar xf mysql-5.7.23-1.el6.x86_64.rpm-bundle.tar
[root@testdb soft]# ls
mysql-5.7.23-1.el6.x86_64.rpm-bundle.tar
mysql-community-embedded-devel-5.7.23-1.el6.x86_64.rpm
mysql-community-client-5.7.23-1.el6.x86_64.rpm
mysql-community-libs-5.7.23-1.el6.x86_64.rpm
mysql-community-common-5.7.23-1.el6.x86_64.rpm
mysql-community-libs-compat-5.7.23-1.el6.x86_64.rpm
mysql-community-devel-5.7.23-1.el6.x86_64.rpm
mysql-community-server-5.7.23-1.el6.x86_64.rpm
mysql-community-embedded-5.7.23-1.el6.x86_64.rpm
mysql-community-test-5.7.23-1.el6.x86_64.rpm

安装指定的软件包：
mysql-community-server
mysql-community-client

[root@testdb soft]# rpm -ivh mysql-community-server-5.7.23-1.el6.x86_64.rpm mysql-community-client-5.7.23-1.el6.x86_64.rpm 
warning: mysql-community-server-5.7.23-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
error: Failed dependencies:
	libaio.so.1()(64bit) is needed by mysql-community-server-5.7.23-1.el6.x86_64
	libaio.so.1(LIBAIO_0.1)(64bit) is needed by mysql-community-server-5.7.23-1.el6.x86_64
	libaio.so.1(LIBAIO_0.4)(64bit) is needed by mysql-community-server-5.7.23-1.el6.x86_64
	mysql-community-common(x86-64) = 5.7.23-1.el6 is needed by mysql-community-server-5.7.23-1.el6.x86_64
	mysql-community-libs(x86-64) >= 5.7.9 is needed by mysql-community-client-5.7.23-1.el6.x86_64

分析：
安装mysql-server软件包时有依赖，libaio.so.1()(64bit)和mysql-community-common(x86-64)
安装mysql-client软件包时有要求，mysql-libs软件包版本必须>= 5.7.9
解决：
1. 解决依赖安装
2. 安装或更新mysql-libs高版本的软件

[root@testdb soft]# rpm -ivh /mnt/Packages/libaio-0.3.107-10.el6.x86_64.rpm /mnt/Packages/libaio-devel-0.3.107-10.el6.x86_64.rpm 
warning: /mnt/Packages/libaio-0.3.107-10.el6.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID c105b9de: NOKEY
Preparing...                ########################################### [100%]
   1:libaio                 ########################################### [ 50%]
   2:libaio-devel           ########################################### [100%]

[root@testdb soft]# rpm -ivh mysql-community-common-5.7.23-1.el6.x86_64.rpm 
warning: mysql-community-common-5.7.23-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                ########################################### [100%]
	file /usr/share/mysql/czech/errmsg.sys from install of mysql-community-common-5.7.23-1.el6.x86_64 conflicts with file from package mysql-libs-5.1.71-1.el6.x86_64

分析：
有相同功能的软件已经被安装，导致该软件在指定的位置创建文件时冲突
解决：
忽略依赖关系卸载已经安装的软件mysql-libs
[root@testdb soft]# rpm -e mysql-libs --nodeps

最后，重新安装：
[root@testdb soft]# rpm -ivh mysql-community-common-5.7.23-1.el6.x86_64.rpm mysql-community-client-5.7.23-1.el6.x86_64.rpm mysql-community-server-5.7.23-1.el6.x86_64.rpm  mysql-community-libs-5.7.23-1.el6.x86_64.rpm mysql-community-libs-compat-5.7.23-1.el6.x86_64.rpm 
warning: mysql-community-common-5.7.23-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                ########################################### [100%]
   1:mysql-community-common ########################################### [ 20%]
   2:mysql-community-libs   ########################################### [ 40%]
   3:mysql-community-client ########################################### [ 60%]
   4:mysql-community-server ########################################### [ 80%]
   5:mysql-community-libs-co########################################### [100%]
[root@testdb soft]# 

~~~

##### 4. yum源配置

###### 理论储备：

**yum源的作用：**软件包管理器，类似360的软件管家。

![yum](/yum.png)

**yum源的好处：**解决软件包之间的依赖关系，提高运维人员的工作效率。

**常见的yum源分类：**

- **本地yum源**—*yum仓库在本地*（系统光盘/镜像文件）
- **网络yum源**—*yum仓库不在本地*（远程）
  - 国内较知名的网络源（aliyun源，163源，sohu源，知名大学开源镜像等）
  - 国外较知名的网络源（centos源、redhat源、扩展[^epel]源等）
  - 特定软件相关的网络源（Nginx、MySQL、Zabbix等）

###### 具体操作：

~~~powershell
思路：
1. 本机需要挂载光盘或者镜像文件（仓库在本机）
2. 通过修改配置文件告诉yum工具去哪个仓库里找软件包

步骤：
1. 挂载光盘到本机
注意：如果前面已经挂载成功，则省略
[root@testdb soft]# mount -o ro /dev/sr0 /mnt
[root@testdb soft]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   18G  3.2G   14G  20% /
tmpfs                         491M     0  491M   0% /dev/shm
/dev/sda1                     485M   33M  427M   8% /boot
/dev/sr0                      4.2G  4.2G     0 100% /mnt

2. 修改配置文件告诉yum工具到哪个仓库里找
[root@testdb ~]# cd /etc/yum.repos.d/
[root@testdb yum.repos.d]# ll
total 16
-rw-r--r--. 1 root root 1926 Nov 27  2013 CentOS-Base.repo
-rw-r--r--. 1 root root  638 Nov 27  2013 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  630 Nov 27  2013 CentOS-Media.repo
-rw-r--r--. 1 root root 3664 Nov 27  2013 CentOS-Vault.repo
//备份默认的yum源文件
[root@testdb yum.repos.d]# mkdir backup
[root@testdb yum.repos.d]# mv CentOS-* backup/
[root@testdb yum.repos.d]# ls
backup
[root@testdb yum.repos.d]# pwd
/etc/yum.repos.d
//创建一个自定义文件，文件名必须以.repo结尾
[root@testdb yum.repos.d]# vi local.repo
[local]					//仓库的名字，不要有特殊符号（自定义）
name=local yum			//仓库描述
baseurl=file:///mnt		//指定yum仓库的路径（重要）；file://表示本地仓库；ftp://或者http://
enabled=1			//启用仓库，1表示启用；0表示不启用
gpgcheck=0			//不用校验软件包的签名，0表示不校验；1表示校验

3. 清空yum缓存并创建缓存
[root@testdb yum.repos.d]# yum clean all
Loaded plugins: fastestmirror
Cleaning repos: local
Cleaning up Everything
[root@testdb yum.repos.d]# yum makecache
Loaded plugins: fastestmirror
Determining fastest mirrors
local               | 4.0 kB     00:00 ... 
local/group_gz      | 220 kB     00:00 ... 
local/filelists_db  | 5.8 MB     00:00 ... 
local/primary_db    | 4.4 MB     00:00 ... 
local/other_db      | 2.7 MB     00:00 ... 
Metadata Cache Created

~~~

**使用yum工具安装MySQL5.7.23的rpm包**

~~~powershell
清空环境：
[root@testdb yum.repos.d]# rpm -e libaio libaio-devel --nodeps
[root@testdb yum.repos.d]# rpm -e mysql-community-server mysql-community-client mysql-community-common mysql-community-libs-compat mysql-community-libs
error: Failed dependencies:
	libmysqlclient.so.16()(64bit) is needed by (installed) postfix-2:2.6.6-2.2.el6_1.x86_64
	libmysqlclient.so.16(libmysqlclient_16)(64bit) is needed by (installed) postfix-2:2.6.6-2.2.el6_1.x86_64
	mysql-libs is needed by (installed) postfix-2:2.6.6-2.2.el6_1.x86_64
[root@testdb yum.repos.d]# rpm -e mysql-community-server mysql-community-client mysql-community-common mysql-community-libs-compat mysql-community-libs --nodeps
确认环境清理完毕：
[root@testdb yum.repos.d]# rpm -aq|grep mysql-

yum工具安装：
[root@testdb yum.repos.d]# cd /soft/
[root@testdb soft]# yum localinstall mysql-community-server-5.7.23-1.el6.x86_64.rpm mysql-community-client-5.7.23-1.el6.x86_64.rpm mysql-community-libs-* mysql-community-common-5.7.23-1.el6.x86_64.rpm

注意：libaio软件会自动安装，说明yum帮我们解决了该依赖关系。
~~~

#### 任务2：

##### 思路：

1. 配置好了本地yum源直接安装vsftpd软件
2. 查看samba软件是否安装，没安装先安装，然后再升级

##### 具体操作：

~~~powershell
1. 安装vsftpd软件
# yum list|grep vsftpd		查看仓库里是否有vsftpd软件
# yum install vsftpd		安装vsftpd软件
# yum list|grep vsftpd		@local	@表示已经安装成功，local表示仓库名
vsftpd.x86_64                            2.2.2-11.el6_4.1                  @local
# rpm -q vsftpd		确认是否安装
vsftpd-2.2.2-11.el6_4.1.x86_64

2. 升级samba软件
a. 先通过本地仓库安装3.6.9版本
yum -y install samba		
b. 配置网络yum源
[root@MissHou yum.repos.d]# cat 163.repo 
[163]
name=163 network yum
baseurl=http://mirrors.163.com/centos/6/os/x86_64/
enabled=1
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/6/os/x86_64/RPM-GPG-KEY-CentOS-6

c. 清空缓存创建缓存
yum clean all
yum makecache

d. 升级samba
yum update samba

e. 确认成功升级
# rpm -q samba
samba-3.6.23-51.el6.x86_64
~~~



### 扩展总结

#### 1. 二进制包获取方式

1.1 RedHat/Centos光盘或官方网站 ftp://ftp.redhat.com

1.2 推荐网站 

- www.rpmfind.net
- rpm.pbone.net（可搜索）

1.3 相应软件官方网站 

- http://www.mysql.com
- http://nginx.org/packages/

#### 2. 二进制包如何选择

2.1选择适合当前系统的版本号：

- 找不到适合的，才去尝试别的系统版本号  
- el6兼容el5；el5无法安装 el6 

2.2 选择适合cpu的架构：  

- x86_64包，只能安装在64位的系统上  
- i386,i586,i686的软件包可以安装在32和64位系统上  
- noarch表示这个软件包与硬件构架无关，可以通用  
- 32位系统不能安装64位包

**建议**： 建议不要跨大版本号去安装软件包，尽量使用当前版本自带软件包安装

#### 3. 源码包编译安装优缺点

- **优点：**

1. 可以在任意平台上编译安装，编译出来的软件包非常适应所在机器。  
2. 可以在编译的时候，通过配置，对某些功能进行定义，开启或关闭相应的功能。

- **缺点：**

  1. 安装麻烦  
  2. 卸载麻烦  
  3. 升级麻烦

- **源码安装三部曲：**

  - 根据需求配置：

    ~~~powershell
    --prefix=... Directories to put files in /usr/local 软件家目录
    --bindir=... $prefix/bin 	命令的目录
    --etcdir=... $prefix/etc 	配置文件的目录
    --mandir=... $prefix/share/man man	文档路径
    --locale=... $prefix/share/locale 	语言编码
    ...
    ~~~

  - 编译：

    ~~~powershell
    make	（使用gcc编译器进行编译）
    ~~~

  - 安装：

    ~~~powershell
    make install 类似 rpm -ivh
    ~~~

**源码安装示例：**

~~~powershell
需求：将axel软件安装到/opt/axel目录下
[root@MissHou soft]# ls
axel-1.0a.tar.gz

1. 解压
[root@MissHou soft]# tar xf axel-1.0a.tar.gz 
[root@MissHou soft]# ls
axel-1.0a  axel-1.0a.tar.gz

2. 进入到解压目录进行安装
[root@MissHou soft]# cd axel-1.0a
[root@MissHou axel-1.0a]# ls
API     axel.h          conf.c     conn.c   CREDITS  ftp.h   http.h    README    tcp.c
axel.1  axelrc.example  conf.h     conn.h   de.po    gui     Makefile  search.c  tcp.h
axel.c  CHANGES         configure  COPYING  ftp.c    http.c  nl.po     search.h  text.c

根据需求配置：
[root@MissHou axel-1.0a]# ./configure --prefix=/opt/axel

编译：
[root@MissHou axel-1.0a]# make
...

安装：
[root@MissHou axel-1.0a]# make install

确认成功安装：
[root@MissHou axel-1.0a]# ls /opt/axel/
bin  etc  RPM-GPG-KEY-CentOS-6  share

使用axel下载文件：
[root@MissHou axel]# axel http://mirrors.163.com/centos/6/os/x86_64/RPM-GPG-KEY-CentOS-6
-bash: axel: command not found

分析可能原因：
1. 命令没有安装
2. 命令安装了但是找不到
原因：命令安装了但是不能找到，原因是不知道去/opt/axel/bin目录里找。
说明：系统中有PATH环境变量，该变量里保存的是命令的路径，只要PATH中有命令的所在路径，就可以找到。

解决：
[root@MissHou axel-1.0a]# echo $PATH
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
临时添加到环境变量中：
[root@MissHou axel-1.0a]# export PATH=$PATH:/opt/axel/bin
[root@MissHou axel-1.0a]# echo $PATH
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:/opt/axel/bin

永久添加到环境变量：
[root@MissHou axel-1.0a]# vi /etc/profile
在文件最后增加如下内容：
export PATH=$PATH:/opt/axel/bin

重新读取配置文件：
[root@MissHou axel-1.0a]# source /etc/profile

再次下载：成功
[root@MissHou axel-1.0a]# axel http://mirrors.163.com/centos/6/os/x86_64/RPM-GPG-KEY-CentOS-6
Initializing download: http://mirrors.163.com/centos/6/os/x86_64/RPM-GPG-KEY-CentOS-6
File size: 1706 bytes
Opening output file RPM-GPG-KEY-CentOS-6
Starting download

查看axel更详细的帮助：
[root@MissHou axel]# man axel
No manual entry for axel
原因：找不到指定的man文档

解决：
[root@MissHou axel]# vi /etc/man.config
...
增加如下内容：
MANPATH /opt/axel/share/man
~~~

**源码卸载软件：**

~~~powershell
进入到解压包目录里：
[root@MissHou axel-1.0a]# make uninstall		//卸载安装
rm -f /opt/axel/bin/axel
rm -f /opt/axel/etc/axelrc
rm -f /opt/axel/share/man/man1/axel.1
[root@MissHou axel-1.0a]# make distclean	//卸载编译和配置过程
rm -f *.o axel search core *.mo
rm -f Makefile.settings config.h

~~~



#### 4. 自建yum仓库

**思考：**什么情况下需要自建yum仓库？

**具体步骤：**

~~~powershell
1. 创建一个目录来保存相应的软件
[root@MissHou soft]# mkdir /soft
[root@MissHou soft]# ls
esound-libs-0.2.41-3.1.el6.x86_64.rpm  xlockmore-5.31-2.el6.x86_64.rpm
2. 使用createrepo扫描该目录下的所有软件
[root@MissHou soft]# yum -y install createrepo
[root@MissHou soft]# createrepo /soft/
[root@MissHou soft]# ls
esound-libs-0.2.41-3.1.el6.x86_64.rpm  repodata  xlockmore-5.31-2.el6.x86_64.rpm
说明：
扫描后多出一个repodata目录，存放的软件之间的依赖关系

3. 配置本地yum源
[root@MissHou yum.repos.d]# cat local.repo 
[local]
name=local yum
baseurl=file:///mnt
enabled=1
gpgcheck=0

[myself]
name=myself
baseurl=file:///soft
enabled=1
gpgcheck=0

4. 测试安装锁屏软件
[root@MissHou yum.repos.d]# yum -y install xlockmore
~~~

#### 5. 配置yum源核心思想

- 需要有个仓库（仓库里有软件并且还有软件之间的依赖关系）
  - 如果仓库在本地——>本地仓库（本地源）
  - 如果仓库在网络上（外网|内网）——>网络仓库（网络源）
- 需要修改/etc/yum.repos.d/目录里以*.repo结尾的文件告诉yum工具去哪个仓库找软件包

~~~powershell
[repositoryid]									  //仓库名或ID不要有特殊符号
name=Some name for this repository			//仓库描述
baseurl=url://path/to/repository/			//仓库路径
enabled=1										//启用仓库
gpgcheck=0										//不校验仓库里软件包的签名

[repositoryid]
name=Some name for this repository
baseurl=url://server1/path/to/repository/
        url://server2/path/to/repository/
        url://server3/path/to/repository/
enabled=1
gpgcheck=0
~~~

#### 6. yum和rpm工具使用

#####6.1 rpm命令

~~~powershell
rpm -ivh	package
安装  
rpm -e package
卸载
rpm -Uvh
升级，如果已安装老版本,则升级;如果没安装,则直接安装
rpm -Fvh
升级，如果已安装老版本,则升级;如果没安装,则**不**安装
rpm -ivh --force
强制安装
rpm --nodeps
忽略依赖关系
rpm -ql
查看已经安装的软件的文件列表
rpm -qlp  package.rpm 
查看未安装的rpm包里的文件列表
rpm -qa  查看已经安装的所有rpm包
rpm -qd  查看软件的文档列表
rpm -qc  查看软件的配置文件
rpm -qi  查看软件的详细信息
rpm -qf  filename
查看文件来自哪个rpm包
rpm --import    key_file
导入公钥用于检查rpm文件的签名
rpm -checksig   package.rpm
检查rpm包的签名
~~~

##### 6.2 yum命令

~~~powershell
# yum install package -y
默认是安装来自仓库里的软件，指定的是软件名字。多个包空格隔开；-y （取消交互）
# yum install ./xlockmore-5.31-2.el6.x86_64.rpm
或者
# yum localinstall ./xlockmore-5.31-2.el6.x86_64.rpm
安装来自本地指定路径下的rpm包，而不是来自仓库
# yum remove 或者 erase package
卸载软件包
# yum update
更新仓库里所有比本机已经安装过的软件要的软件	
# yum update package
指定升级的软件
# yum search mysql
搜索出所有软件名字或者软件描述包含“mysql”关键字的软件
# yum provides  "*libmysqlclient.so*"
找出模块由哪些软件包提供
# yum provides "*xeye*"
搜索一个包含xeye关键字的软件包
# yum clean all
清空之前的yum列表缓存
# yum makecache
创建新的缓存
# yum list
列出仓库里的所有软件包
# yum repolist
列出已配置的软件仓库
# yum list|tail
查看未安装的软件包
# yum list |grep 关键字
@代表已经安装成功
# yum list installed
查看已安装的包
# yum grouplist
查看包组
# yum groupinstall  "包组"
安装包组
# yum groupremove "包组"
# md5sum +包名
直接校验第三方提供的软件包 
~~~

**课堂练习：**

1. 使用网络yum仓库安装MySQL5.6的最新版本
2. 自建yum仓库安装xlockmore锁屏软件



### 课后实战

#### 1. 需求1

Centos默认使用自带的国外源，由于网络问题请将其替换为国内的阿里云源，163源，sohu源其中之一



#### 2. 需求2

开发人员需要安装某个软件包（epel源中有），发现现有yum源中没有，需要运维协助配置EPEL源（两种方式搭建EPEL源）



#### 3. 需求3

给开发人员搭建能够提供指定软件包的安装，如Nginx和MySQL的yum源









~~~powershell
puppet:
http://yum.puppetlabs.com/puppetlabs-release-el-6.noarch.rpm
http://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm
http://yum.puppetlabs.com/puppetlabs-release-fedora-20.noarch.rpm

Zabbix:
http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm
http://repo.zabbix.com/zabbix/2.2/rhel/6/x86_64/zabbix-release-2.2-1.el6.noarch.rpm

mysql:
https://repo.mysql.com/mysql-community-release-el6.rpm
https://repo.mysql.com/mysql-community-release-el7.rpm

~~~



[^epel]: 为“红帽系”的操作系统提供额外的软件包，适用于RHEL、CentOS等系统




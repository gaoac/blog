---
title: 可能是目前最详细简明的CentOS7安装与管理教程
date: 2018-02-22 22:22:46
tags: [Linux, CentOS]
categories: [Linux, CentOS]
---

#### 说在前面

俗话说好记性不如烂笔头，考虑到每次安装部署都要各种查阅资料，很是不便，故决定重头开始安装一遍常用服务，作为以后的参照。

#### 第一步：确定发行版本，安装系统

首先明确自己需要的版本，本人不习惯桌面版（作为服务器，推荐熟悉命令行系统，毕竟效率上不是一个量级的），而且也不喜欢集成好的第三方镜像，故直接在官网下载最小化版本[Minimal ISO](https://www.centos.org/download/)。这里我们以目前最新版本 CentOS7 64 位系统为例进行安装（具体安装过程不在叙述，大家肯定可以的）。<!-- more -->

#### 第二步：root 密码重置

首次安装， root 账号密码默认为空（当然也可以在安装过程中设置），但本人经常忘记密码，所以难免有要重置密码的时候。

1. 重启系统，开机过程中，出现下图画面时，通过快速按下`↑`和`↓`来暂停引导程序（对手速自信的同学请无视），如图：

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517912748698.png)

2. 使用`↑`和`↓`选择第一行（背景高亮即为选中），按下键盘上的 e，进入编辑模式;

3. 将光标一直移动到 LANG=en_US.UTF-8 后面，空格，再追加 init=/bin/sh。这里特别注意，需要写在 UTF-8 后，保持在同一行，并注意空格。由于屏幕太小，会自动添加\换行，这个是正常的。如图：

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517913276601.png)

4. 按下`CTRL+X`进行引导启动，成功后进入该界面，如图：

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517913476481.png)

5. 接下来逐步输入以下命令：

   1. 挂载根目录
      `mount -o remount, rw /`
   2. 选择要修改密码的用户名，这里选择 root 用户进行修改，可以更换为你要修改的用户
      `passwd root`
   3. 输入 2 次一样的新密码，注意输入密码的时候屏幕上不会有字符出现。
      如果输入的密码太简单，会提示警告（BAD PASSWORD：The password fails the dictionary check - it is too simplistic/systematic），可以无视它，继续输入密码，不过建议还是设置比较复杂一些的密码，以保证安全性
   4. 如果已经开启了 SElinux（这个后面会讲），则需要输入以下命令
      `touch /.autorelabel`
   5. 最后输入以下命令重启系统即可
      `exec /sbin/init`或 `exec /sbin/reboot`

#### 第三步：开启网卡

因为最小化安装以后，centos 默认未开启网卡，所以首先需要开启网卡：

1. 执行命令`cd /etc/sysconfig/network-scripts`，看到下图：

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517914521832.png)

2. 执行命令`vi ifcfg-ens33`（vi/vim 编辑器用法相信小伙伴都很熟悉了，这里不再涉及），将 `ONBOOT=no` 改为 `ONBOOT=yes`，如图：

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517915250037.png)

3. 由于我是在虚拟机里安装的 centos,同时作为服务供给其他局域网用户使用，所以选择桥接模式，将 centosIP、网关、DNS 等信息进行配置，如图：

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517916354021.png)

> `BOOTPROTO=dhcp` -->`BOOTPROTO=static` IP 获取方式改为静态获取</br>
> `ZONE=public` firewalld zone=piblic（公共）：在公共区域内使用，不能相信网络内其他计算机不会对你造成危害，只能接受经过选取的连接。</br>
> `PADDR=10.82.17.71` IP 地址</br>
> `ATEWAY=10.82.17.1` 网关 ，与虚拟机虚拟网卡 VMnet8 中设置的网关保持一致</br>
> `ETMASK=255.255.255.0` 子网掩码</br>
> `DNS1=10.82.1.4`</br>
> `DNS2=10.82.1.6`

然后执行`systemctl restart network`，重启网络服务

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517915336851.png)

测试网络是否连通：`ping www.baidu.com`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517971505116.png)
出现以下信息，说明可以正常访问互联网了（至于上图为什么画风变了，额，这个纯属个人喜好，小伙伴可以自己选择喜欢的终端工具以及主题配色)

#### 第四步：关闭 SELinux

> SELinux(Security-Enhanced Linux) 是美国国家安全局（NSA）对于强制访问控制的实现，是 Linux 历史上最杰出的新安全子系统。NSA 是在 Linux 社区的帮助下开发了一种访问控制体系，在这种访问控制体系的限制下，进程只能访问那些在他的任务中所需要文件。
> SELinux 是一种基于 域-类型 模型（domain-type）的强制访问控制（MAC）安全系统，它由 NSA 编写并设计成内核模块包含到内核中，相应的某些安全相关的应用也被打了 SELinux 的补丁，最后还有一个相应的安全策略。任何程序对其资源享有完全的控制权。假设某个程序打算把含有潜在重要信息的文件扔到/tmp 目录下，那么在 DAC 情况下没人能阻止他。SELinux 提供了比传统的 UNIX 权限更好的访问控制。

但是，很多服务都有 SELinux 的限制，比如常见的/tmp 文件夹无访问权限，改起来颇为麻烦，个人使用还是关闭 SELinux，省心。

**查看 SELinux 状态**

执行命令：`getenforce`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517989977313.png)

如上图显示`Enforcing`，说明 SELinux 处于开启状态。

**临时关闭**

```properties
##设置SELinux 成为permissive模式
##setenforce 1 设置SELinux 成为enforcing模式
setenforce 0
```

**永久关闭**
直接修改配置文件
执行命令：`vi /etc/selinux/config`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517990948894.png)

将`SELINUX=enforcing`改为`SELINUX=disabled`
然后执行命令`reboot`重启系统生效
再次查看，状态已变为`disabled`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517991162515.png)

#### 第五步：防火墙基础配置

在 centos7 时代防火墙已由 iptable 转向 firewalld，既然本文讲的是 centos7，那么我们就直接接受并适应它。:smile:
在此之前，要提一提`systemctl`：

> systemd 是一个 Linux 系统基础组件的集合，提供了一个系统和服务管理器，运行为 PID 1 并负责启动其它程序。功能包括：支持并行化任务；同时采用 socket 式与 D-Bus 总线式激活服务；按需启动守护进程（daemon）；利用 Linux 的 cgroups 监视进程；支持快照和系统恢复；维护挂载点和自动挂载点；各服务间基于依赖关系进行精密控制。systemd 支持 SysV 和 LSB 初始脚本，可以替代 sysvinit。除此之外，功能还包括日志进程、控制基础系统配置，维护登陆用户列表以及系统账户、运行时目录和设置，可以运行容器和虚拟机，可以简单的管理网络配置、网络时间同步、日志转发和名称解析等。

简单说就是：systemctl 是 CentOS7 的服务管理工具中主要的工具，它融合之前 service 和 chkconfig 的功能于一体。在系统服务管理中推荐使用 systemctl 来管理。

**下面以 firewalld 服务为例：**

1. firewalld 服务启用/停用

> 启动一个服务：systemctl start firewalld.service</br>
> 关闭一个服务：systemctl stop firewalld.service</br>
> 重启一个服务：systemctl restart firewalld.service</br>
> 显示一个服务的状态：systemctl status firewalld.service</br>
> 在开机时启用一个服务：systemctl enable firewalld.service</br>
> 在开机时禁用一个服务：systemctl disable firewalld.service</br>
> 查看服务是否开机启动：systemctl is-enabled firewalld.service</br>
> 查看已启动的服务列表：systemctl list-unit-files|grep enabled</br>
> 查看启动失败的服务列表：systemctl --failed

2.配置 firewalld-cmd

> 查看版本： firewall-cmd –version</br>
> 查看帮助： firewall-cmd –help </br>
> 显示状态： firewall-cmd –state </br>
> 查看所有打开的端口： firewall-cmd –zone=public –list-ports </br>
> 更新防火墙规则： firewall-cmd –reload </br>
> 查看区域信息: firewall-cmd –get-active-zones </br>
> 查看指定接口所属区域： firewall-cmd –get-zone-of-interface=eth0 </br>
> 拒绝所有包：firewall-cmd –panic-on </br>
> 取消拒绝状态： firewall-cmd –panic-off </br>
> 查看是否拒绝： firewall-cmd –query-panic

3.端口管理：

> 添加： firewall-cmd --zone=public --add-port=80/tcp --permanent （--permanent 永久生效，没有此参数重启后失效）</br>
> 重新载入：firewall-cmd --reload</br>
> 查看：firewall-cmd --zone= public --query-port=80/tcp</br>
> 删除：firewall-cmd --zone= public --remove-port=80/tcp --permanent</br>

正式环境下，看需要选择是否使用防火墙，这里为了方便后续配置，就先将其关闭：

关闭防火墙 `systemctl stop firewalld`</br>
禁止开机自启`systemctl disable firewalld`</br>
查看防火墙状态`systemctl status firewalld`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518068337892.png)

#### 第六步：添加常用 yum 源（软件包）

linux 下软件安装方式有很多，比如 RMP、YUM、源代码安装等。其中 CentOS 内置的 yum 命令安装非常的简单实用，能自动帮助我们解决依赖，在此推荐 yum 方式安装软件应用，但 CentOS 最小化安装后，内置的 yum 源可用的软件偏少或者版本过低，通常我们需要使用一些第三方的 yum 源，这里向大家推荐两个比较常用和权威的 yum 源，EPEL 和 REMI。

**EPEL**

> EPEL 是 Extra Packages for Enterprise Linux 的缩写（EPEL），是用于 Fedora-based Red Hat Enterprise Linux (RHEL) 的一个高质量软件源，所以同时也适用于 CentOS 或者 Scientific Linux 等发行版。

**REMI**

> Remi repository 是包含最新版本 PHP 和 MySQL 包的 Linux 源，由 Remi 提供维护。有个这个源之后，使用 YUM 安装或更新 PHP、MySQL、phpMyAdmin 等服务器相关程序的时候就非常方便了。

首先查看目前系统中已存在的 yum 源：`yum repolist`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517998950801.png)
可以看到目前系统 yum 源有三个，接下来我们开始添加新的 yum 源。

由于现在安装 REMI 源的时候会自动安装 EPEL 作为依赖包。所以我们只需要直接安装 REMI 即可：`yum install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517999270175.png)

然后确认，安装完毕
![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517999341889.png)

再次查看`yum repolist`
![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1517999454043.png)

可以看到，我们已经多出了两个 yum 源（后续可继续增加其他源，这里就不再补充了）。

可在`cd /etc/yum.repos.d` 中查看对应 repo 文件。

**接下来在安装软件之前，我们先来熟悉下 yum 常用命令：**

> yum repolist all： 显示所有仓库</br>
> yum repolist 或 yum repolist enabled： 显示可用仓库</br>
> yum repolist disabled：显示禁用仓库</br>
> yum list 或 yum list all：显示所有的程序包</br>
> yum list available：显示可安装的程序包</br>
> yum list updates：显示可更新程序包</br>
> yum list installed：显示已安装程序包</br>
> yum list recent： 显示最近新增的程序包</br>
> yum search xxx：搜索 xxx 程序包</br>
> yum install xxx ：安装 xxx 程序包</br>
> yum update xxx ：升级 xxx 程序包</br>
> yum remove xxx 或 yum erase xxx：卸载 xxx 程序包</br>
> yum info xxx：查看程序包 xxx 信息</br>
> yum deplist xxx：查看程序包 xxx 依赖</br>
> yum clean all ：清理本地缓存</br>
> yum clean plugins ：清除插件缓存</br>
> yum makecache：构建缓存</br>
> yum history：查看 yum 事务历史

我们先执行命令：`yum makecache` 把服务器的包信息下载到本地电脑缓存起来,以提高搜索 、安装软件的速度，如图：
![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518000705761.png)

#### 第七步：软件应用安装与配置

首先我们从常规的 LMAP 套装开始：

##### MariaDB

> CentOS 6 或早期的版本中提供的是 MySQL 的服务器/客户端安装包，但 CentOS 7 已使用了 MariaDB 替代了默认的 MySQL。MariaDB 数据库管理系统是 MySQL 的一个分支，主要由开源社区在维护，采用 GPL 授权许可 MariaDB 的目的是完全兼容 MySQL，包括 API 和命令行，使之能轻松成为 MySQL 的代替品。

**在这里先介绍下常用的 RPM 命令：**

> 查询软件包</br>
> rpm -q xxx</br>
> rpm -qp **_.rpm： 获取当前目录下的 rpm 包相关信息</br>
> rpm -qa | less ：列出所有已安装的软件包</br>
> rpm -qa | grep xxx ：列出所有被安装的 xxx</br>
> rpm -qf /usr/sbin/httpd ：查看某个文件属于哪个软件包，可以是普通文件或可执行文件，跟文件的绝对路径</br>
> rpm -qi xxx：列出已安装的 xxx 包的标准详细信息</br>
> rpm -ql xxx：列出 rpm 包 xxx 的文件内容
> 安装软件包</br>
> rpm -ivh _**.rpm：其中 i 表示安装，v 表示显示安装过程，h 表示显示进度</br>
> 升级软件包</br>
> rpm -Uvh \*\*\*.rpm</br>
> 删除软件包</br>
> rpm -e xxx</br>
> rpm -e -–nodeps xxx：不考虑依赖包</br>
> rpm -e –-allmatches xxx：删除所有跟 xxx 匹配的所有版本的包

###### 安装

首先查看系统是否安装过 mariadb：
`rpm -qa | grep mariadb`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518002173008.png)

先卸载系统中的 mariadb：
`rpm -e --nodeps mariadb-libs-****.x86_64`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518003583701.png)

查看可安装版本：
`yum list mariadb*`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518004073357.png)

**这里要说明一下:**

> 出于实用的目的，MariaDB 是同一 MySQL 版本的二进制替代品(例如 MySQL 5.1->MariaDB 5.1, MariaDB5.2 和 MariaDB 5.3 是兼容的。MySQL 5.5 将会和 MariaDB 5.5 保持兼容)。简单说 5.x 就是为了兼容 MySQL5.x 的，接口几乎一致，体验上几乎无差别。
> 但是从 2012 年 11 月 12 日发布的 mariadb10.0.0 开始，不在依照 MySQL 的版本号，10.0.x 版本是以 5.5 版为基础，加入移植自 MySQL5.6 版的新功能和自行开发的新功能。

所以，为了更好的兼容已有 MySQL(5.6 以前)版本，这里我们不安装最新版 marisdb10，而是选择 5.5 版本。

这里我们安装`mariadb`与`mariadb-server`即可。
执行命令`yum install -y mariadb mariadb-server`
![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518004392140.png)
![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518004502275.png)

程序会自动分析其需要的依赖并下载安装，我们静等完成就好。
![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518004524407.png)
到此，mariadb 安装结束。

###### 启动配置

启动 mariadb
`systemctl start mariadb`
查看运行状态
`systemctl status mariadb`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518004813058.png)
设置开启自启
`systemctl enable mariadb`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518054872049.png)

###### 密码配置

登陆数据库：
`mysql -u root -p`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518054933348.png)
首次安装后，root 账号默认密码为空，下面我们为 root 账号设置密码
执行命令：`mysql_secure_installation`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518060529264.png)

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518061621496.png)
使用刚设置的密码登陆数据库：
`mysql -u root -p`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518061780929.png)

###### 字符集与排序规则

接下来，让我么你看下 mariadb 数据库字符集(Character set)和排序规则(Collation)：
执行：`show variables like "%character%";show variables like "%collation%";`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518066545688.png)

这里再普及下字符集的概念：

> character_set_client: 代表客户端字符集，客户端最简单的来说，就是指命令行，或者其它操作数据库的网页，应用等等，客户端字符集就代表了用户输入的字符，用什么字符集来编码。</br>
> character_set_connection: 代表与服务器连接层的字符集，mysql 是连接 mysqld 服务器的客户端，两者连接层，采用的字符集。</br>
> character_set_database: 数据库采用的字符集。</br>
> character_set_filesystem: 文件采用的肯定是二进制最合适，不用修改。</br>
> character_set_result: 结果字符集，返回结果时采用的字符集。</br>
> character_set_server: mysql 服务器采用的字符集，也就是操作默认的字符集。</br>
> character_set_system: 系统元数据(字段名等)字符集，比如我们输入的命令'insert ...'这些语句字符串采用的字符集。</br> >**\_collation\_\_类同**

为了保证统一，避免出现编码不一致导致的乱码问题，我们统一设置成 UTF-8:

**这里不得不再次强调一下：**

> MariaDB / MySQL 中 的 "utf8" 并不是真正的 UTF-8，其中的 "utf8mb4" 才是真正的 UTF-8。"utf8" 只支持每个字符最多三个字节，而真正的 UTF-8 是每个字符最多四个字节。MySQL 在 5.5.3 之后增加了这个 "utf8mb4" 的编码，mb4 就是 most bytes 4 的意思，专门用来兼容四字节的 unicode。好在 "utf8mb4" 是 "utf8" 的超集，除了将编码改为 "utf8mb4" 外不需要做其他转换，如果要想完美兼容，或者想存储 emoji 表情之类的，最好还是设置成 "utf8mb4" 当然，为了节省空间，一般情况下使用 "utf8' 也就够了。所以，现在网络上出现的设置"utf8' 的文章，可以说都多少有些过时了。

查看数据库支持字符集：
执行 `SHOW CHARSET;`
可以看到当前版本的 MariaDB 是支持 "utf8mb4" 的。由于个人需要，为了节省空间，我们接下来还是设置成 "utf8"，大家知道这个事情就好，再根据项目需要进行选择。

![202001171430.png](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/202001171430.png)

**临时修改（重启后失效）**

> 字符集</br>
> – mysql> SET character_set_client = utf8 ; </br>
> – mysql> SET character_set_connection = utf8 ;</br>
> mysql> SET character_set_database = utf8 ; </br>
> mysql> SET character_set_results = utf8 ; </br>
> mysql> SET character_set_server = utf8 ;
> 排序规则</br>
> – mysql> SET collation_connection = utf8_general_ci;</br>
> mysql> SET collation_database = utf8_general_ci;</br>
> mysql> SET collation_server = utf8_general_ci ;

这里对 mysql 中的排序规则 utf8_unicode_ci、utf8_general_ci 的区别总结：

> ci 是 case insensitive, 即 "大小写不敏感"</br>
> utf8_general_ci 不区分大小写</br>
> utf8_general_cs 区分大小写</br>
> utf8_unicode_ci 和 utf8_general_ci 对中、英文来说没有实质的差别。</br>
> utf8_general_ci 校对速度快，但准确度稍差。</br>
> utf8_unicode_ci 准确度高，但校对速度稍慢。</br>
> tf8_unicode_ci 比较准确，utf8_general_ci 速度比较快。通常情况下 utf8_general_ci 的准确性就够我们用的了，如果你的应用有德语、法语或者俄语，请一定使用 utf8_unicode_ci。

**永久修改**
首先修改 my.cnf 文件：
`vi /etc/my.cnf`
在[mysqld]下添加

```properties
init_connect='SET collation_connection = utf8_general_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_general_ci
# To ignore client information and use the default server character set
# 忽略客户端字符集信息，并使用服务器默认字符集
skip-character-set-client-handshake
```

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518077165045.png)

重启 mariadb
`systemctl restart mariadb`
重新登录，再次查看
`show variables like "%character%";show variables like "%collation%";`
都已设置成 utf8。

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1518075868100.png)

###### 用户与权限

创建用户：
`CREATE USER username IDENTIFIED BY 'password';`

为用户设置权限：

> 授予 username 用户在所有数据库上的所有权限：`GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost' IDENTIFIED BY 'password';`</br>
> 撤销 username 用户在所有数据库上的所有权限`REVOKE ALL PRIVILEGES ON *.* FROM 'username'@'localhost';`</br>
> 授予 username 用户在 xxx 数据库上的所有权限：`GRANT ALL PRIVILEGES ON xxx.* TO 'username'@'localhost' IDENTIFIED BY 'password';`</br>
> 授予 username 用户在 xxx 数据库上的 SELECT, UPDATE 权限：`GRANT SELECT, UPDATE ON xxx.* TO 'username'@'localhost' IDENTIFIED BY 'password';`

注意：上述命令中`@localhost`指的是本地，如果需要远程登录数据库，则使用`@'%'`

刷新权限：
`FLUSH PRIVILEGES;`

删除用户：
`DROP USER username@localhost;`

##### Apache

查看可安装版本
`yum list httpd`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519261606246.png)
这里我们直接安装最新版 2.4.6
`yum install -y httpd`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519261693393.png)
安装完毕，启动 Apache
`systemctl start httpd`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519261943199.png)
Apache 默认端口 80，所以在浏览器访问`http://localhost`，出现以下界面，说明 Apache 启动成功

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519262020266.png)
设置开机自启
`systemctl enable httpd`

##### PHP

查看可安装版本
`yum list php`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519264617630.png)

显示版本为 5.4.16，想要使用 php7 的话，需要安装升级 PHP7 的 rpm 源
`rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm`

查看 php7 安装包
`yum list php`及`yum list php*w`,可以看到目前可以安装的各版本

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519265639829.png)

这里我们不是以 PHP 为主，就选系统默认版本 5.4.16
`yum install php`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519265796957.png)

安装完毕，查看 php 版本
`php -v`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519265856347.png)
重启 Apache
`systemctl restart httpd`
apache 默认根目录`/var/www/html`,添加文件 phpinfo.php，输入以下内容:

```php
<?php
    phpinfo();
?>
```

访问`http://localhost/phpinfo.php`，查看 php 相关信息

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519266278349.png)

###### 安装 PHP 模块

查看已安装模块
`php -m`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519266378610.png)

这里我们需要再安装常用的一些模块，推荐使用 pecl 安装 php 扩展

> PECL 的全称是 The PHP Extension Community Library ，是一个开放的并通过 PEAR(PHP Extension and Application Repository，PHP 扩展和应用仓库)打包格式来打包安装的 PHP 扩展库仓库。通过 PEAR 的 Package Manager 的安装管理方式，可以对 PECL 模块进行下载和安装。

安装 pecl：
`yum install php-pear php-devel`

安装模块：
`pecl install dom mbstring mcrypt mysql mysqli PDO pdo_mysql pdo_sqlite posix sqlite3 sysvmsg sysvsem sysvshm wddx xmlreader xmlwriter xsl`

如遇到 pecl 找不到的扩展模块，再尝试 yum 安装，如：
`yum install php-gd php-mbstring php-mcrypt php-mysql php-mysqli php-pdo php-pdo_sqlite php-posix php-sqlite3 php-ldap`

重启 Apache
`systemctl restart httpd`

查看新增 php 模块

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519270131023.png)

###### 安装 phpMyAdmin

> phpMyAdmin 是一个以 PHP 为基础，以 Web-Base 方式架构在网站主机上的 MySQL 的数据库管理工具，让管理者可用 Web 接口管理 MySQL 数据库。借由此 Web 接口可以成为一个简易方式输入繁杂 SQL 语法的较佳途径，尤其要处理大量资料的汇入及汇出更为方便。其中一个更大的优势在于由于 phpMyAdmin 跟其他 PHP 程式一样在网页服务器上执行，但是您可以在任何地方使用这些程式产生的 HTML 页面，也就是于远端管理 MySQL 数据库，方便的建立、修改、删除数据库及资料表。也可借由 phpMyAdmin 建立常用的 php 语法，方便编写网页时所需要的 sql 语法正确性。

安装：
`yum install -y phpmyadmin`

> phpMyAdmin 的默认安装目录是 /usr/share/phpMyAdmin，同时会在 Apache 的配置文件目录中自动创建虚拟主机配置文件 /etc/httpd/conf.d/phpMyAdmin.conf（区分大小写）。默认情况下，CentOS 7 上的 phpMyAdmin 只允许从回环地址(127.0.0.1)访问。为了能远程连接，你需要改动它的配置。

修改配置：
`vi /etc/httpd/conf.d/phpMyAdmin.conf`

```properties
<Directory /usr/share/phpMyAdmin/>
   AddDefaultCharset UTF-8

   <IfModule mod_authz_core.c>
     # Apache 2.4
     <RequireAny>
      # Require ip 127.0.0.1  #注释掉
      # Require ip ::1   #注释掉
      Require all granted   #新添加(允许所有请求访问资源)
     </RequireAny>
 </IfModule>
 <IfModule !mod_authz_core.c>
     # Apache 2.2
     Order Deny,Allow
     Deny from All
     Allow from 127.0.0.1
     Allow from ::1
   </IfModule>
</Directory>

<Directory /usr/share/phpMyAdmin/setup/>
   <IfModule mod_authz_core.c>
     # Apache 2.4
     <RequireAny>
      #Require ip 127.0.0.1  #注释掉
      #Require ip ::1   #注释掉
      Require all granted   #新添加(允许所有请求访问资源)
     </RequireAny>
   </IfModule>
   <IfModule !mod_authz_core.c>
     # Apache 2.2
     Order Deny,Allow
     Deny from All
     Allow from 127.0.0.1
     Allow from ::1
   </IfModule>
</Directory>

作者：TyiMan
链接：https://www.jianshu.com/p/bc14ff0ab1c7
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

然后重启 Apache 服务器：
`systemctl restart httpd`
访问`http://ip/phpmyadmin`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519280996686.png)

##### JDK

查看可安装 JDK
`yum search java|grep jdk`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519286456125.png)

Linux 发行版中用的多是 OpenJDK（关于 OpenJDK 与 Oracle JDK 的区别这里不再赘述）。

我们选择安装 OpenJDK1.8 即可：
`yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519287864542.png)

`java -version`

![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519287894436.png)

Linux 上使用 yum 命令后，会将 OpenSDK 安装到 /usr/lib/jvm/ 目录下。
设置 JAVA-HOME，让系统上的所有用户使用 java(OpenSDK )
`vi /etc/profile`
在末尾添加：

```properties
#set java environment
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64
JRE_HOME=$JAVA_HOME/jre  CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

使配置文件生效：
`source /etc/profile`

验证环境变量是否生效:
`echo $PATH`
![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519290488857.png)

##### Tomcat

下载当前 Tomcat8 最新版的安装文件 apache-tomcat-8.0.27.tar.gz(<https://tomcat.apache.org/download-80.cgi)；>

将 apache-tomcat-8.0.28.tar.gz 文件放到/usr/local 目录下，执行如下脚本：

`cd /usr/local` </br>
`tar -zxvf apache-tomcat-8.5.28.tar.gz` 解压压缩包 </br>
`rm -rf apache-tomcat-8.5.28.tar.gz` 删除压缩包 </br>
`mv apache-tomcat-8.5.28 tomcat` 重命名

通过 systemctl 管理 tomcat：

1）首先，为 tomcat 增加一个 pid 文件

在 tomca/bin 目录下面，增加 setenv.sh 配置，catalina.sh 启动的时候会调用，同时配置 java 内存参数；

`vi setenv.sh`

```sh
#add tomcat pid
CATALINA_PID="$CATALINA_BASE/tomcat.pid"
#add java opts
JAVA_OPTS="-server -XX:PermSize=256M -XX:MaxPermSize=1024m -Xms512M -Xmx1024M -XX:MaxNewSize=256m"
```

保存文件；
修改文件为可执行：
`chmod a+x /usr/local/tomcat/bin/setenv.sh`
2）增加 tomcat.service

在/usr/lib/systemd/system 目录下增加 tomcat.service，目录必须是绝对目录。
`vi tomcat.service`

```properties
[Unit]
Description=Tomcat
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/tomcat/tomcat.pid
ExecStart=/usr/local/tomcat/bin/startup.sh
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

> [unit] 配置了服务的描述，规定了在 network 启动之后执行。</br> >[service] 配置服务的 pid，服务的启动，停止，重启。</br> >[install] 配置了使用用户。

执行`systemctl daemon-reload`,使 tomcat.service 生效

启动 tomcat：
`systemctl start tomcat`
开机启动：
`systemctl enable tomcat`

> tomcat 启动时会在 tomcat 的根目录/usr/local/tomcat 下生成 pid 文件 tomcat.pid，停止后会删除，用 systemctl 管理 tomcat 不会出现同时启动多个 tomcat，这样可以保证始终只有一个 tomcat 在运行

访问<http://ip:8080/,出现以下界面说明启动成功>
![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519350528632.png)
但当我们点击红色框中按钮，进入管理时，提示无访问权限；
![](https://raw.githubusercontent.com/gaoac/images-library/master/blog/CentOS7/1519350569459.png)
这时我们按提示，进入/usr/local/tomcat/conf，编辑 tomcat-users.xml，设置用户：
在`<tomcat-users></tomcat-users>`内部添加：

```xml
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="用户名" password="密码" roles="manager-gui,admin-gui"/>
```

另外远程登录 tomcat 管理界面权限，注释掉/usr/local/tomcat/webapps/manager/META-INF/context.xml 和/usr/local/tomcat/webapps/host-manager/META-INF/context.xml 中：

```xml

  <!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
  -->

```

再次使用刚设置的账号密码登陆即可成功登录 tomcat 管理系统。

##### Node

###### nvm

为了方便管理 node，我们使用 NVM（node 版本管理器）
安装（先确保安装过 curl /wget 工具，没有就安装下）：
`curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash`
或者
`wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash`

安装完后，重新打开终端，查看安装情况：
`nvm --version`

```bash
[root@localhost ~]# nvm --version
0.33.8
```

> nvm 常用命令：</br>
> nvm install <version> ## 安装指定版本，可模糊安装，如：安装 v8.9.4，既可 nvm install v8.9.4，又可 nvm install 8.9.4</br>
> nvm uninstall <version> ## 删除已安装的指定版本，语法与 install 类似</br>
> nvm use <version> ## 切换使用指定的版本 node</br>
> nvm list ## 列出所有安装的版本</br>
> nvm list-remote ## 列出所有远程服务器的版本（官方 node version list）</br>
> nvm current ## 显示当前的版本</br>
> nvm alias <name> <version> ## 给不同的版本号添加别名</br>
> nvm unalias <name> ## 删除已定义的别名</br>
> nvm reinstall-packages <version> ## 在当前版本 node 环境下，重新全局安装指定版本号的 npm 包

我们安装当前 LTS（长期稳定版）v8.9.4 以及最新版
`nvm install 8.9.4`
`nvm install 9.6.0`

查看已安装版本：

```bash
[root@localhost ~]# nvm list
         v8.9.4 *
->       v9.6.0 *
default -> 8.9.4 (-> v8.9.4 *)
node -> stable (-> v9.6.0 *) (default)
stable -> 9.6 (-> v9.6.0 *) (default)
iojs -> N/A (default)
lts/* -> lts/carbon (-> v8.9.4 *)
lts/argon -> v4.8.7 (-> N/A)
lts/boron -> v6.13.0 (-> N/A)
lts/carbon -> v8.9.4 *
```

然后使用 8.9.4：
`nvm use 8.9.4`

```bash
[root@localhost ~]# nvm use 8.9.4
Now using node v8.9.4 (npm v5.6.0)
```

查看当前版本：
`nvm current`

```bash
[root@localhost ~]# nvm current
v8.9.4
```

###### nrm

接下来我们安装 nrm（管理 npm 源切换的利器）

安装：
`npm install -g nrm`

> nrm 常用命令：</br> ></br>
> nrm ls : 显示所有 registry</br>
> nrm current : 显示当前 registry</br>
> nrm use xxx : 使用 xxx registry

`nrm ls`

```bash
[root@localhost ~]# nrm ls

* npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
  taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
```

`nrm use taobao`
再次查看,npm 源已切换到 taobao：

```bash
[root@localhost ~]# nrm ls

  npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
* taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
```

> 淘宝 NPM 镜像
> 是一个完整 npmjs.org 镜像，你可以用此代替官方版本(只读)，同步频率目前为 10 分钟 一次以保证尽量与官方服务同步。

接下来，就可以随意使用 npm 安装 node 模块包了。
如：`npm install -g npm-check yarn serve pm2 typescript`

##### MongoDB

> MongoDB  是一个基于分布式文件存储的数据库。由 C++语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。
> MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。他支持的数据结构非常松散，是类似 json 格式，因此可以存储比较复杂的数据类型。Mongo 最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

###### 安装 MongoDB

首先创建源，创建 repo 文件，下面我们[官方安装方法](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)安装：

```properties
# 在/etc/yum.repos.d/目录下创建文件mongodb-org-3.6.repo，它包含MongoDB仓库的配置信息，内容如下：
# 复制代码, 代码如下:
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
# $releasever 为你的Linux发行版本
```

yum 安装 MongoDB

```bash
yum install -y mongodb-org-3.6.3
```

启动 MongoDB</br>
`systemctl start mongod`

设置开机自启</br>
`systemctl enable mongod`

###### 配置 MongoDB

> MongoDB 默认是不开启权限认证的，但自从上次 MongoDB 爆发了[赎金门事件](http://coolshell.cn/articles/17607.html)，还是很有开启 MongoDB 的权限认证的必要。

开启认证也很简单，在配置文件（默认是/etc/mongodb.conf）里面进行配置即可：

```properties
security:
  authorization: enabled
#2.6前的版本为auth = true
```

重启数据库后，再次进入数据库进行插入等操作，就会提示错误了。这说明权限认证生效了，未认证通过的用户再也不能使用数据库了（即使能进 mongo shell）。

这时我们需要一个“超级管理员”来创建、分配管理员给指定数据库。

> MongoDB 的开发者早已经想到了这一步。MongoDB 自带一个数据库叫 admin，这个数据库用来管理所有数据库的，类似于 MySQL 的 mysql 数据库。如果这个数据库的管理员账户还没有建立，那么任何人都可以在 admin 数据库里面新建管理员账户。

```sql
--进入admin数据库
use admin;
--运行db.createUser方法新建用户
db.createUser({user: '超级管理员用户名', pwd: '密码', roles: [{role: 'userAdminAnyDatabase', db: 'admin'}]})
```

> createUser 方法必须传入一个有 user（用户名）、pwd（密码），roles（角色）三个属性的 JSON 对象。不同 roles 拥有不同权限，比如：数据库读、数据库写、数据库用户管理等等，其中角色 userAdminAnyDatabase 可以看做是“超级管理员”，我们建立该角色的用户后，就可以用这个用户来管理其他用户了。

MongoDB 内置角色:

> (1).数据库用户角色</br>
> 针对每一个数据库进行控制。</br>
> read :提供了读取所有非系统集合，以及系统集合中的 system.indexes, system.js,system.namespaces</br>
> readWrite: 包含了所有 read 权限，以及修改所有非系统集合的和系统集合中的 system.js 的权限。</br>
> (2).数据库管理角色</br>
> 每一个数据库包含了下面的数据库管理角色。</br>
> dbOwner：该数据库的所有者，具有该数据库的全部权限。</br>
> dbAdmin：一些数据库对象的管理操作，但是没有数据库的读写权限。（参考：<http://docs.mongodb.org/manual/reference/built-in-roles/#dbAdmin）></br>
> userAdmin：为当前用户创建、修改用户和角色。拥有 userAdmin 权限的用户可以将该数据库的任意权限赋予任意的用户。</br>
> (3).集群管理权限</br>
> admin 数据库包含了下面的角色，用户管理整个系统，而非单个数据库。这些权限包含了复制集和共享集群的管理函数。</br>
> clusterAdmin：提供了最大的集群管理功能。相当于 clusterManager, clusterMonitor, and hostManager 和 dropDatabase 的权限组合。</br>
> clusterManager：提供了集群和复制集管理和监控操作。拥有该权限的用户可以操作 config 和 local 数据库（即分片和复制功能）。</br>
> clusterMonitor：仅仅监控集群和复制集。</br>
> hostManager：提供了监控和管理服务器的权限，包括 shutdown 节点，logrotate, repairDatabase 等。</br>
> 备份恢复权限：admin 数据库中包含了备份恢复数据的角色。包括 backup、restore 等等。</br>
> (4).所有数据库角色</br>
> admin 数据库提供了一个 mongod 实例中所有数据库的权限角色：</br>
> readAnyDatabase：具有 read 每一个数据库权限。但是不包括应用到集群中的数据库。</br>
> readWriteAnyDatabase：具有 readWrite 每一个数据库权限。但是不包括应用到集群中的数据库。</br>
> userAdminAnyDatabase：具有 userAdmin 每一个数据库权限，但是不包括应用到集群中的数据库。</br>
> dbAdminAnyDatabase：提供了 dbAdmin 每一个数据库权限，但是不包括应用到集群中的数据库。</br>
> (5). 超级管理员权限</br>
> root: dbadmin 到 admin 数据库、useradmin 到 admin 数据库以及 UserAdminAnyDatabase。但它不具有备份恢复、直接操作 system.\*集合的权限，但是拥有 root 权限的超级用户可以自己给自己赋予这些权限。</br>
> (6). 备份恢复角色：backup、restore。</br>
> (7). 内部角色：\_\_system</br>

十分复杂，为了简单起见，就讲其中两个：read、readWrite 也就是常用的读数据库和读写数据库。

> 这里有一个不大不小的坑，就是你要给其他数据库创建用户，都必须先到 admin 数据库，认证刚才新建的那个 admin 用户，然后再切换到其他数据库才能建立用户。

建立了 admin 用户之后，还必须先进入 admin 数据库进行认证：

步骤如下：

```sql
use admin;
db.auth('超级管理员用户名', '密码')
```

然后切换到数据库 a，给数据库 a 创建用户

```sql
use 数据库a;
db.createUser({user: '用户a', pwd: '密码', roles: [{role: 'readWrite', db: '数据库a'}]})
db.auth('用户a', '密码')
```

现在，就可以使用`用户a`管理`数据库a`，进行正常的读写了。

开启远程登录：

在配置文件（默认是/etc/mongodb.conf）中，注释掉 bindIp，或者将 127.0.0.1 改为 0.0.0.0

```properties
# network interfaces
net:
  port: 27017
  #bindIp: 127.0.0.1  # Listen to local interface only, comment to listen on all interfaces.
```

最后是 MongoDB 图形化管理工具：

推荐使用[Studio 3T](https://studio3t.com/)（前身是 robomongo），虽然收费，但是基础功能免费，足够了。

嗯，待续吧。。。\*\*

---
layout: mypost
title: CentOS 7 安装大全 之 PXE网络篇（CentOS系统）
categories: [CentOS]
---

>本文基于CentOS7部署PXE服务进行网络安装CentOS7。

## 准备工作
1、CentOS7系统环境

2、CentOS7系统镜像（[官网下载](https://www.centos.org/download/)/[清华源](http://mirrors.tuna.tsinghua.edu.cn/centos/7.5.1804/isos/x86_64/CentOS-7-x86_64-DVD-1804.iso)）

## 什么是PXE网络
- PXE(preboot execute environment，预启动执行环境)是由Intel公司开发的技术，工作于Client/Server的网络模式，支持工作站通过网络从远端服务器下载映像，并由此支持通过网络启动操作系统，在启动过程中，终端要求服务器分配IP地址，再用TFTP（trivial file transfer protocol）或MTFTP(multicast trivial file transfer protocol)协议下载一个启动软件包到本机内存中执行，由这个启动软件包完成终端（客户端）基本软件设置，从而引导预先安装在服务器中的终端操作系统。
- PXE可以引导多种操作系统。
- PXE client集成在网卡ROM中，当计算机引导时，BIOS把PXE client调入内存执行，获取PXE server配置，显示菜单，根据用户选将远程操作系统下载到本机运行。
PXE组件及过程的分析。
- 部署PXE需要哪些服务：
  - DHCP服务，分配IP地址，定位引导程序
  - DNS服务，为客户机分配主机名
  - TFTP服务，提供引导程序下载
  - HTTP服务(或ftp/nfs)，提供yum安装源
- 客户机应具备的条件：
  - 网卡ROM必须支持PXE协议
  - 主板支持网络启动
## 部署PXE服务器

#### 1、软件需求
- dhcpd: 　　动态分配IP
- xinetd: 　　对服务访问进行控制，这里主要控制tftp
- tftp:　　	　从服务器端下载pxelinux.0、default文件
- httpd:　 　在网络上提供安装源，也就是ISO镜像文件中的内容
- syslinux: 　用于网络引导

```
~$ sudo yum install dhcp xinetd syslinux tftp-server httpd
...
```
#### 2、 配置IP
将服务器的IP配置为192.168.0.1，让DHCP能够正常启动，TFTP，HTTP都是运行在这个IP上。

```
~$ sudo ip addr add 192.168.0.1/24 brd + dev ensxx      # ensxx是网卡名称
~$ ip addr show                                         # 查看网卡ip
```
#### 3、配置DHCP
编辑dhcp的配置文件/etc/dhcp/dhcpd.conf的内容：

```
~$ sudo vi /etc/dhcp/dhcpd.conf
```

```
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp*/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
# 1. 整体的环境设定
ddns-update-style none;
ignore client-updates;
default-lease-time 259200;
max-lease-time 518400;
option domain-name-servers 192.168.0.1;
# 上面是 DNS 的 IP 设定,这个设定值会修改客户端的 /etc/resolv.conf

# 2. 关于动态分配的 IP
subnet 192.168.0.0 netmask 255.255.255.0 {
       range 192.168.0.100 192.168.100.200;
       option routers 192.168.0.1; 
       option subnet-mask 255.255.255.0;
       next-server 192.168.0.1;
       # the configuration  file for pxe boot
       filename "centos7/pxelinux.0";
}
```
#### 4、 配置TFTP
tftp是由xinetd管理的，所以需要配置 /etc/xinetd.d/tftp文件，这个文件中只需要改一个参数即可

```
~$ sudo vi /etc/xinetd.d/tftp
```

```
service tftp
{
    socket_type     = dgram
    protocol        = udp
    wait            = yes
    user            = root
    server          = /usr/sbin/in.tftpd
    server_args     = -s /var/lib/tftpboot
    disable         = no          #将此值改为no，表明开启此服务
    per_source      = 11
    cps             = 100 2
    flags           = IPv4
}
```
#### 5、文件准备
首先将已经下载好的CentOS的ISO镜像文件挂载到一个目录中，然后复制可引导的、压缩的内核文件`vmlinuz`，以及包含一些模块和安装文件系统的`initrd`。因为安装过程中以http的方式提供镜像源，所以这里直接将镜像文件挂在到httpd访问目录中(/var/www/html)。

```
~$ sudo mkdir  /var/www/html/centos7
~$ sudo mount -o loop CentOS7.iso /var/www/html/centos7/
```
复制`vmlinuz`，和 `initrd.img` 到TFTP访问目录的centos7子目录中，因为以后会引导其它的系统，所以这里通过子目录将不同的系统区分开。
```
~$ sudo mkdir /var/lib/tftpboot/centos7
~$ sudo cp /var/www/html/centos7/images/pxeboot/initrd.img  /var/lib/tftpboot/centos7/
~$ sudo cp /var/www/html/centos7/images/pxeboot/vmlinuz  /var/lib/tftpboot/centos7/
```
#### 6、设置syslinux加载器
`vesamenu.c32`和`menu.c32`是syslinux所拥有众多模块中的两个，它们的功能是制定启动器使用什么模式的背景。`vesamenu.c32`图形模式，`menu.c32`文本模式。我选择的是`menu.c32`。

同时还需要`pxelinux.0`文件，它对整个引导器的作用就如同内核对系统的作用，它可以解释`default`文件（配置引导菜单文件）中的每个配置项，并根据配置项做出不同的反应。如等待的时间、启动器背景、启动菜单、内核引导等等。

需要将这两个文件复制到tftp的访问目录中：

```
~$ sudo cp  /usr/share/syslinux/menu.c32  /var/lib/tftpboot/
~$ sudo cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
```
`pxelinux`被执行后，它会扫描该目录下是否存在指定的配置文件，如果存在，则引用被指定的配置文件。 
`default`文件存放于pxelinux.cfg目录中，`pxelinux`程序最后扫描的配置文件名就是`default`。所以，把启动器配置项都写入该文件中。 

所以就要建立pxelinux.cfg目录，并在此目录下建立default文件，编辑引导菜单。

```
~$ sudo mkdir /var/lib/tftpboot/pxelinux.cfg/
~$ sudo vi /var/lib/tftpboot/pxelinux.cfg/default
```
default文件内容为:

```
default menu.c32
prompt 0
timeout 300
ONTIMEOUT local
 
menu title ########## PXE Boot Menu ##########
 
label 1
menu label ^1) Install CentOS 7 x64 with HTTP
kernel centos7/vmlinuz
append initrd=centos7/initrd.img method=http://192.168.0.1/centos7 devfs=nomount
```
#### 7、启动服务
开启dhcpd, xinetd, tftp, http这些服务，在开启的时候没有发生错误，说明配置没问题。

```
~$ sudo systemctl start dhcpd.service 
~$ sudo systemctl start xinetd.service 
~$ sudo systemctl start tftp.service
~$ sudo systemctl start httpd.service
```
> 可以在浏览器输入192.168.0.1验证http是否运行正常

为了防止意外的发生可以关闭防火墙和selinux。

```
~$ sudo systemctl stop firewalld.service 
~$ sudo setenforce 0
```
如果要通过MAC地址分配IP，可以编辑dhcp的配置文件/etc/dhcp/dhcpd.conf

```
subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.100 192.168.0.200;
    option routers 192.168.0.1;
}

host centos7 {
    hardware ethernet 00:0C:29:C7:57:0C;    # 客户端的mac地址
    next-server 192.168.0.1;
    fixed-address 192.168.0.125;            # 分配给客户端的IP
    filename "centos7/pxelinux.0";
}
```

#### 部署PXE服务器完成

## 安装CentOS系统 
1、启动要安装系统的电脑，在开机Log界面按F12选择从网络启动：（BIOS设置？自行解决！）
出现界面，选择“Install CentOS liunx 7”，回车继续：
![](CentOS1.jpg)
2、如下界面，默认选择English，点击Continue继续：
![](CentOS2.jpg)
3、主要配置界面有3个部分：
- Localization和software部分不需要进行任何设置，其中sofrware selection选项，默认值即最小化安装，不包含图形界面，至于其他组件，待后期使用通过yum安装即可。
![](CentOS3.jpg)
- system部分需要必须规划配置的是磁盘分区规划。
![](CentOS4.jpg)
- 点击“installation destination”，进入如下界面，选中创建的虚拟硬盘，点选“i will configure partitioning”，即自定义磁盘分区，最后点击左上角done：
![](CentOS5.jpg)

4、继续进行磁盘规划，（可按需自行规划）：
- /boot：1024M，标准分区格式创建。
- swap：4096M，标准分区格式创建。
- /：剩余所有空间，采用lvm卷组格式创建。
规划完成后，点击done完成分区规划，在弹出对话框中点击“accept changs”：
![](CentOS6.jpg)
![](CentOS7.jpg)
5、完成磁盘规划后，点击“network & host name”部分。
![](CentOS8.jpg)
- 修改操作系统主机名，然后点击done完成主机名配置，返回主配置界面：
![](CentOS9.jpg)
6、如下界面，右下角“begin installtion”按钮已经变成蓝色，点击“begin installtion”开始进行操作系统安装。
![](CentOS10.jpg)
7、进入用户设置界面，要做的是修改root的密码，点击“root password”，
![](CentOS11.jpg)
- 设置密码root密码，点击Done完成。
![](CentOS12.jpg)
> 设置成功再次返回用户界面，可以点击“user setting”创建用户。（建议新建一个管理员账号）

8、安装完成后，点击“reboot”重启系统。
- 使用root用户登录，启用网卡：

```
~$ ip addr     # 查看网卡
... 
~$ sudo vi /etc/sysconfig/network-scripts/ifcfg-xxx    # 编辑网卡配置文件 
```

![](CentOS13.jpg)
- 编辑修改/etc/sysconfig/network-scripts/ifcfg-ens32文件内容：

```
ONBOOT=yes   # 网卡随系统启动， 默认为no
```
- 重启网卡

```
~$ sudo service network restart        #重启网卡
~$ ping -c 4 www.baidu.com            #测试网络
```
# ...大功告成...


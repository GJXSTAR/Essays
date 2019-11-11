---
layout: mypost
title: apache与Subversion
categories: [apache, Subversion]
---

# apache

>Apache是世界使用排名第一的Web服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。它快速、可靠并且可通过简单的API扩充，将Perl/Python等解释器编译到服务器中。

安装 apache
: - **sudo apt-get install apache2**

启动apache
: **sudo service apache2 start**

#Subversion
>subversion是一个自由开源的版本控制系统。Subversion将文件存放在中心版本库里，这个版本库很像一个普通的文件服务器，不同的是，它可以记录每一次文件和目录的修改情况，这样就可以借此将数据恢复到以前的版本，并可以查看数据的更改细节。


安装Subversion
: **sudo apt-get install subversion**
**sudo apt-get install libapache2-mod-svn**

创建subversion用户组，并且把用户加到组里
: **sudo addgroup subversion**
**sudo usermod -G subversion -a www-data**

创建项目文件夹，并设置权限
: **sudo mkdir /home/svn**
**cd /home/svn**
**sudo mkdir myproject** 
**sudo chown -R www-data:subversion myproject**

创建 SVN 文件仓库
: **sudo svnadmin create /home/svn/myproject**

赋予组成员相应的权限
: **sudo chmod -R g+rws myproject**

##访问 Subversion 文件仓库

直接访问文件仓库(file://)
>这是所有访问方式中最简单的。它不需要事先运行任何 SVN 服务。这种访问方式用于访问本地的 SVN 文件仓库。
: **svn co file:///home/svn/myproject**
**svn co file://localhost/home/svn/myproject**

通过 WebDAV 协议访问(http://)
: **配置 /etc/apache2/mods-available/dav_svn.conf**

```
<Location /svn/myproject>
DAV svn
SVNPath /home/svn/myproject
AuthType Basic
AuthName "myproject Subversion Repository"
AuthUserFile /etc/subversion/passwd
#<LimitExcept GET PROPFIND OPTIONS REPORT>
Require valid-user
#</LimitExcept>
</Location>
```

重启 Apache 2 Web 服务器
: **sudo service apache2 restart**

添加用户
: **sudo htpasswd -c /etc/subversion/passwd user_name**

访问文件仓库
: **svn co http://hostname/svn/myproject myproject --username user_name**


同步文件仓库和本地副本，执行 update 子命令
: **svn update**

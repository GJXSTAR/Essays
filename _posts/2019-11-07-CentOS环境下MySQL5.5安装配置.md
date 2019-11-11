---
layout: mypost
title: CentOS 7 环境下 MySQL-5.5.60 安装配置
categories: [MySQL]
---

参考 [MySQL-5.5 官方文档](https://dev.mysql.com/doc/refman/5.5/en/binary-installation.html)

# 准备工作
### 删除 Mariadb
由于CentOS7自带的是 Mariadb, 所以需要删除。
[Mariadb官网](https://mariadb.org/)
[Mariadb中文网](https://mariadb.com/kb/zh-cn/mariadb/)

```
~$ rpm -qa | grep mariadb          # 查看版本
mariadb-libs-x.x.x-xxx.x84_64      # 
~$ sudo rpm -e --nodeps mariadb-libs-x.x.x-xxx.x84_64  # 删除上面查到的相应软件
~$ sudo rm /etc/my.cnf        # 如果有 my.cnf 配置文件，就删除
```
### 查看 libaio
MySQL依赖于libaio库。
```
~$ yum search libaio              # 搜索libaio
~$ sudo yum install libaio        # 安装libaio,如果以有会更新或提示无更改
```

#  安装
### 下载 MySQL

```
~$ wget https://cdn.mysql.com/archives/mysql-5.5/mysql-5.5.60-linux-glibc2.12-x86_64.tar.gz
```
### 复制文件到 `/usr/local`

```
~$ sudo cp mysql-5.5.60-linux-glibc2.12-x86_64.tar.gz /usr/local/
```
### 解压文件

```
~$ cd /usr/local
~$ sudo tar -zxvf mysql-5.5.60-linux-glibc2.12-x86_64.tar.gz
~$ sudo mv mysql-5.5.60-linux-glibc2.12-x86_64 mysql-5.5.60
~$ sudo ln -s mysql-5.5.60 mysql      # 创建符号链接
```
#### 创建 mysql 用户和组

```
~$ sudo groupadd mysql
~$ sudo useradd -r -g mysql -s /bin/false mysql
```
### 设置权限

```
~$ cd mysql
~$ sudo chown -R mysql ./
~$ sudo chgrp -R mysql ./
```
#### 初始化数据目录
```
~$ sudo scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
...
```
### 更改权限

```
~$ sudo chown -R root ./
~$ sudo chown -R mysql data/
```
----
# .配置服务
### 配置`.cnf`文件

```
~$ sudo cp /usr/local/mysql/support-files/my-large.cnf /etc/my.cnf
~$ sudo vim /etc/my.cnf
```
### 修改`my.cnf`文件

```
#MySQL客户端配置
[client]
#password	= your_password
port		= 3306			    #客户端链接端口
socket		= /tmp/mysql.sock	#socket文件路径

# MySQL服务器配置
[mysqld]
port		= 3306			    #客户端链接端口
socket		= /tmp/mysql.sock	#socket文件路径
skip-external-locking			#避免外部锁定
key_buffer_size = 256M			#执行索引的缓冲区大小
max_allowed_packet = 1M			#服务器发送和接受的最大包长度
table_open_cache = 256			#允许缓存已打开表的份数
sort_buffer_size = 1M			#执行排序使用的缓冲大小
read_buffer_size = 1M			#随机读缓冲区大小
read_rnd_buffer_size = 4M		#通信时缓存数据的大小
myisam_sort_buffer_size = 64M   #线程使用的堆大小
thread_cache_size = 8			#在cache中保留多少线程用于重用
query_cache_size= 16M			#查询缓冲区大小
thread_concurrency = 8			#允许的线程数量

#skip-networking	    		#关闭通过TCP/IP连接MySQL

log-bin=mysql-bin	    		#复制二进制日志

binlog_format=mixed	    		#二进制日志格式=混合

server-id	= 1		        	#表示本机序号为1,也就master的意思

#如果使用InnoDB表，请对以下内容取消注释
#innodb_data_home_dir = /usr/local/mysql/data	#InnoDB表空间文件目录
#innodb_data_file_path = ibdata1:10M:autoextend	#InnoDB表空间文件路径
#innodb_log_group_home_dir = /usr/local/mysql/data #InnoDB的日志文件目录
#innodb_buffer_pool_size = 256M			#InnoDB缓冲池大小
#innodb_additional_mem_pool_size = 20M		#InnoDB附加的内存池大小
#innodb_log_file_size = 64M			#日志组中每个日志文件的大小
#innodb_log_buffer_size = 8M			#日志文件所用的buffer大小
#innodb_flush_log_at_trx_commit = 1		#每隔几秒日志文件刷新到磁盘
#innodb_lock_wait_timeout = 50			#事务在被回滚之前等待锁定的超时秒数

#mysqldump备份数据库工具配置
[mysqldump]
quick			            			#不将内存中的整个结果写入磁盘之前缓存
max_allowed_packet = 16M	            #服务器发送和接受的最大包长度

#mysql是针对客户端操作相关的配置
[mysql]
no-auto-rehash			        #命令不自动补全
#safe-updates		    		#仅允许使用带有键值的UPDATEs和DELETEs操作，也就是新手模式

#MyISAM表维护程序配置
[myisamchk]
key_buffer_size = 128M	        #关键词缓冲大小, 一般用来缓冲MyISAM表的索引块
sort_buffer_size = 128M	        #排序缓冲大小
read_buffer = 2M	        	#读取缓冲大小
write_buffer = 2M       		#写入缓冲大小

#mysql数据备份程序配置
[mysqlhotcopy]
interactive-timeout
```

### 配置 MySQL 启动脚本

```
~$ sudo cp /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/mysqld
~$ sudo chmod +x /etc/rc.d/init.d/mysqld
~$ sudo chkconfig --add mysqld
~$ sudo chkconfig --list mysqld
~$ sudo chkconfig mysqld on
```

### 配置PATH

```
~$ vi ~/.bash_profile
# 在文件最后面加入一行内容:
export PATH=$PATH:/usr/local/mysql-5.5.60/bin
# 刷新
~$ source ~/.bash_profile
```
### 启动MySQL

```
~$ sudo /usr/local/mysql/bin/mysqld_safe --user=mysql &
```
### 登录 MySQL

```
~$ mysql -uroot -p
# 修改root密码
mysql> use mysql
mysql> update user set password=password('设置的密码') where user='root' and host='localhost';
mysql> flush privileges;
# 设置远程登录
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '刚才设置的root密码' WITH GRANT OPTION;
```

# ... 大功告成...

title: MySQL开启远程访问授权
isupdate: false
date: 2015-08-20 17:39:55
updatetime:
tags:
 - MySQL
categories: 数据库
---

在做开发测试的时候，经常会用到MySQL数据库。测试使用的数据库不在本机上而在虚拟机里，访问数据库就比较蛋碎了。因为MySQL数据库默认是没有开放远程访问授权的。不能够直接访问，下面介绍两种常用的开启MySQL远程访问权限开启方法:



### 方法一：修改`user`表开启远程访问

在终端中键入以下命令登录MySQL数据库:

```bash
mysql -uROOT -pPASSWORD
```
其中，<font color=red>`ROOT`</font>表示登录的用户名，<font color=red>`PASSWORD`</font>表示登录的该用户的密码。登录成功之后，访问`mysql`数据库，依次执行以下SQL语句:

```bash
mysql>  use mysql;
mysql>  select host,user from user;
```
<!--more-->

将会打印出MySQL数据库中所有用户以及其授权访问地址，以博主的为测试数据库为例，打印信息如下:

```bash
+-------------------------+------------------+
| host                    | user             |
+-------------------------+------------------+
| 127.0.0.1               | root             |
| ::1                     | root             |
| localhost               | debian-sys-maint |
| localhost               | root             |
| xiamo-virtual-machine   | root             |
+-------------------------+------------------+
5 rows in set (0.00 sec)

```

可以看到，<font color=red>`host`</font>栏表示登录主机，可以是ip地址或者主机名，不难发现其实他们都是虚拟机本地地址，因此我们是没办法从物理机访问该数据库的。因此，我们只需要更改登录主机的<font color=red>`host`</font>栏目里的地址，便可以在远程主机上访问了。

比如博主通过`桥接模式`与虚拟机相连接，物理机的ip地址为`192.168.1.101`,虚拟机的ip地址为`192.168.1.103`，我们可以执行以下SQL语句来达到远程访问MySQL:

```bash
mysql>  use mysql;
mysql>  update user set host='192.168.1.101' where user='root' and host='127.0.0.1';
```
然后再执行以下命令重启MySQL即可通过物理机访问MySQL啦:
```bash
sudo service mysql restart
```
也可以执行以下SQL语句刷新授权信息来达到不重启MySQL即可远程访问:
```bash
mysql>  FLUSH PRIVILEGES;
```

当然，在大多数时候，我们都希望可以从任意ip地址访问，而不是仅限于指定的ip，这样就算ip地址变动也不会再次折腾，这个时候我们可以执行以下SQL:

```bash
mysql>  use mysql;
mysql>  update user set host='%' where user='root' and host='127.0.0.1';
```
这样，将<font color=red>`host`</font>值改为<font color=red>`%`</font>,就可以在任意主机登录到MySQL咯。

### 方法二：通过授权语句开启远程访问

与修改user表的方法相同，在终端中连接到MySQL数据库。然后执行以下SQL语句即可授权指定用户远程访问MySQL:
```bash
mysql>  grant all privileges  on *.* to USERNAME@'%' identified by "PASSWORD";
mysql>  FLUSH PRIVILEGES;
```

需要注意的是，其中`*.*`的含义为：
 - 第一个<font color=red>`*`</font>为MySQL中的数据库名。
 - 第二个<font color=red>`*`</font>为MySQL中的指定的数据库的表名。
 - <font color=red>`USERNAME`</font>为授权用户的用户名。
 - <font color=red>`%`</font>为任意主机，也可以写指定的ip地址或者主机名。
 - <font color=red>`PASSWORD`</font>为授权改用户在指定的数据访问MySQL时的密码，**如果该密码和本地用户同名用户的密码不一致，远程访问时务必使用该授权密码**。


### 解决Linux下，MySQL中文乱码问题

首先，当然是确保表或者创建的数据库的编码方式为`UTF-8`啦，在这些都没问题的情况下，执行以下SQL语句，查看MySQL的编码情况：
```bash
mysql>  show variables like 'character%';
```
一般情况下，将会得到以下结果:
```bash
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```
此时，我们修改位于<font color=red>`/etc/mysql/my.cnf`</font>的<font color=red>`my.cnf`</font>文件:
```bash
vim /etc/mysql/my.cnf
```

在<font color=red>`[client]`</font>中加入:
```bash
default-character-set=utf8
```
在<font color=red>`[mysqld]`</font>中加入:
```bash
character-set-server=utf8
```
退出保存之后重启MySQL，再次执行，得到如下结果:
```bash
mysql>  show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

```
即可看到编码都为UTF-8啦~






title: "Digitalocean配置shadowsocks服务器、优化笔记"
date: 2015-06-17 23:45:30
isupdate: true
updatetime: 2015-10-07
tags:
 - shadowsocks
 - digitalocean
 - 科学上网
categories: Linux
---

[![alt text](http://7xkj6q.com1.z0.glb.clouddn.com/blog/image/jpg/digitalocean_logo.jpg "阅读全文")](https://notes.wanghao.work/2015-06-17-Digitalocean%E9%85%8D%E7%BD%AEshadowsocks%E6%9C%8D%E5%8A%A1%E5%99%A8-%E4%BC%98%E5%8C%96%E7%AC%94%E8%AE%B0.html)

<!--more-->
<a ahref="#2"></a>

### 前言

>由于博主一段时间更换服务器到[Digitalocean]()的伦敦机房，结果发现丢包率以及稳定性都不如原有的旧金山机房，于是又打算更换回来。（想来真是作死啊╮(╯▽╰)╭）

值得一提的是：[Digitalocean](https://www.digitalocean.com/?refcode=f07fa4a5aaf1)是具有数据、镜像迁移功能的。只不过由于一些这样或者那样的原因，导致博主打算重新安装`Shadowsocks`。

### Shadowsocks版本

目前`Shadowsocks`服务器端拥有四种语言版本，分别是：
 
 - [Shadowsocks-Python](https://github.com/shadowsocks/shadowsocks)
 - [Shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)
 - [Shadowsocks-Go](https://github.com/shadowsocks/shadowsocks-go)
 - [Shadowsocks-NodeJS](https://github.com/shadowsocks/shadowsocks-nodejs)

虽然[Shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)版本占用内存较低，效率较高，但是考虑到不支持多用户，安装比较麻烦。因此博主采用[Shadowsocks-Python](https://github.com/shadowsocks/shadowsocks)。好了，废话不多说。下面看准备工作吧。

### Linux发行版本

Digitalocean 服务器镜像提供了多种选择，博主选择的是`Debian 7`的32位版本，之所以选择这个版本，基于了以下情况考虑：
 - 博主还是个新手，由于Shadowsocks作者clowwindy本人也建议除非有必要的理由，否则[不要选择对新手不友好的CentOS](
https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E
)。
 - 虽然Ubuntu对新手更为友好，但是Digitalocean家的Ubuntu系统内核，之后要用到的[锐速加速](http://www.serverspeeder.com/)不能很好的支持。
 
>锐速仅支持Digitalocean下Ubuntu的`ubuntu14.04 x64 3.13.0-24`以及、`Ubuntu 14.04 x64 vmlinuz-3.13.0-24-generic`内核,又由于`ubuntu14.04 x64 3.13.0-24`内核版本较低，采用这一内核版本，若同时部署anyconnect或者Cisco IPSec VPN，会造成Cisco VPN无法正常使用。并且，开启锐速会影响Anyconnect，暂时没找到解决办法。
 - Debian下锐速的支持以及方便的包管理。

### 安装Shadowsocks

>以下操作假定已经取得root权限下操作完成。

执行以下命令，将会安装`Shadowsocks-Python`版本：

```bash
apt-get update  #更新软件源

apt-get install python-pip  #安装pip包管理器

pip install shadowsocks

apt-get install python-m2crypto     #安装加密库 
```
稍等片刻，系统便会自动完成Shadowsocks-Python的安装。此时Shadowsocks-Python便已近安装到系统了，接下编辑Shadowsocks的配置文件。

### 配置Shadowsocks

完成安装之后，我们当然是要配置Shadowsocks服务器的账号咯，Shadowsocks的配置文件是采用JSON格式的，执行以下命令:
```bash
vim /etc/shadowsocks.json #编辑配置文件
```
将会在`/etc`目录下新建一个文本文件，如果没有安装`vim`，请执行:
```bash
apt-get install vim     #安装vim
```
来完成`vim`文本编辑器的安装，你也可以只用`vi`或者`nano`编辑器来完成文本的配置。

### Shadowsocks.json语法详解

 - 如果你是当用户只需要一个账号的话，编辑Shadowsocks.json文件的内容为:

```json
{
    "server":"填写你的服务器ip", 
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```


如果要设置多个端口号和密码，则使用下面的配置：

```json
{
    "server":"填写你的服务器IP", 
    "local_address": "127.0.0.1",
    "local_port":1080,
    "port_password":{
         "端口1":"密码",
         "端口2":"密码",
         "端口3":"密码",
         "端口4":"密码",
         "端口5":"密码"
    },
    "timeout":300,
    "method":"aes-256-cfb", 
    "fast_open": false
}
```

字段对应的含义如下表：


|      字段    |     含义              |
|------------ |-----------------------------------|
|      server |     填写你的服务器ip地址              |
| server_port |     填写服务器的端口号                |
| local_port  |     填写本地监听地址，一般为127.0.0.1 |
| local_port  |     填写本地端口号（一般为1080）      |
| password    |     填写你的Shadowsocks密码           |
| timeout     |     超时时间，单位为秒                |
| method      |     [加密方式](https://github.com/shadowsocks/shadowsocks/wiki/Encryption)，推荐使用aes-256-cfb加密 |
| fast_open   |     打开或者关闭[TCP TASTOPEN](https://github.com/shadowsocks/shadowsocks/wiki/TCP-Fast-Open)      |

编辑完毕，保存退出即可。接来下我们就可以使用以下命令来运行我们的Shadowsocks服务器啦~

 - 前台运行

```bash
ssserver -c /etc/shadowsocks.json
```

 - 后台运行

```bash
ssserver -c /etc/shadowsocks.json -d start  #开始

ssserver -c /etc/shadowsocks.json -d stop   #停止
```

### 加入开机自启

虽然这样子就可以运行Shadowsocks了，但是我们肯定不希望每次我们重启服务器之后还要手动启动Shadowsocks吧，因此，我们需要将Shadowsocks设置为开机自启。

设置过程也非常的简单，只需要编辑`/etc/rc.local`文件，加入Shadowsocks命令即可。

执行以下命令:
```bash
vim /etc/rc.local       #编辑rc.local文件
```
默认是什么都没有的空文件，如果原本有内容，只需要在`exit 0`前面加上`/usr/local/bin/ssserver -c /etc/shadowsocks.json -d start`然后退出保存即可。


到此，Shadowsocks-Python的安装配置就完成啦，通过我们本地的客户端连上服务器，连接[YouTube](https://www.youtube.com/watch?v=-YGDyPAwQz0)测试下速度，是这个样子的：
![](http://7xkj6q.com1.z0.glb.clouddn.com/blog/image/jpg/1134591194eba9a1aao.jpg)

嗯，600左右的速度，远远达不到我们心目中想要的那个速度~接下来就是我们的优化时间啦~



### 内核部分的优化


**十分感谢@Way在本博文的留言所反馈的问题，针对这个问题，我重写编写了内核优化部分的配置，来解决通过Shadowsocks下载文件可能导致Shadowsocks-Python服务器端内存占用"过高"而被系统Kill的问题，该问题是因为内核参数调的太高而引起的内存异常占用。因此下面的`启用hybla算法`部分配置作废，已经配置过的朋友们请在`/etc/rc.local`文件中删除以下两句命令并重启系统再按照本部分重新配置，对此造成的不便深表歉意！**


```bash
#请删除以下两句并重启系统来使异常的配置不生效
/sbin/sysctl -p /etc/sysctl.d/local.conf

ulimit -n 51200
```

***接下来就是我们重新配置的时间啦~***

编辑`/etc/sysctl.conf `文件，将文件原内容全部使用`#`号注释，如果你不在意的话，也可以清空这个文件的内容，然后再添加以下内容：

```bash
fs.file-max = 51200
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
```
这样我们内核的参数就重新调整完成了，


编辑`/etc/security/limits.conf`文件，在文件末尾加入以下内容：

```bash
* soft nofile 51200
* hard nofile 51200
```

编辑`/etc/pam.d/common-session`文件，在文件末尾加入以下内容：

```bash
session required pam_limits.so
```

编辑`/etc/profile`文件，在文件末尾加入以下内容：
```bash
ulimit -SHn 51200
```
配置完成之后，重启下系统，重启完成之后，执行以下命令：
```bash
ulimit -n
```

如果没有什么特殊情况，应该会得到`51200`这个结果，这就是我们想要的结果咯~这样子配置之后，就不用将`ulimit -n 51200`这句命令添加到``/etc/rc.local`中了。这样子Shadowsocks优化就完成了，如果你使用了这部分的配置，那么下面的`启用hybla算法`部分的配置请直接跳过，因为可能会造成Shadowsocks下载文件内存异常占用而被kill的问题。





### ~~启用hybla算法（已弃用）~~

**~~按照此方法配置可能会造成Shadowsocks下载文件而导致Shadowsocks服务器端内存异常占用而被kill，请参考上面`内核部分的优化`的配置来替换，本部分配置已经弃用！~~**

~~在`/etc/sysctl.d`目录下创建`local.conf`文件：~~

```bash
vim /etc/sysctl.d/local.conf
```language
```
~~添加以下内容:~~

```bash

# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for high-latency network
net.ipv4.tcp_congestion_control = hybla

# for low-latency network, use cubic instead
# net.ipv4.tcp_congestion_control = cubic

```

~~然后保存退出，执行以下命令:~~
```bash
sysctl --system
```
~~或者执行以下命令：~~
```bash
sysctl -p /etc/sysctl.d/local.conf
```

~~待命令执行完毕，请**务必执行以下命令**:~~
```bash
ulimit -n 51200
```
~~>为了在重启系统的时候 ，这些优化也生效，所以我们需要在`/etc/rc.local`中的`exit 0`前面加入这两句:~~
```
/sbin/sysctl -p /etc/sysctl.d/local.conf

ulimit -n 51200
```

至此，Shadowsocks的优化也就结束了。但是现在的速度并没有提升太多，所以接下来需要安装[锐速](http://www.serverspeeder.com/)加速器了，它提供了20M的免费版本，个人使用时足够了。

### 安装锐速

 - 首先，要去锐速官网注册一个账户，[传送门](http://www.serverspeeder.com/)
 - 然后，[在此页面得到锐速的下载地址](http://my.serverspeeder.com/w.do?m=lsl)。
 - 在服务器上执行以下命令来完成锐速的安装:

```bash
wget http://my.serverspeeder.com/d/ls/serverSpeederInstaller.tar.gz     #获取锐速安装文件

tar zxvf serverSpeederInstaller.tar.gz      #解压

bash serverSpeederInstaller.sh      #执行安装脚本

```
接下来便会验证你的邮箱和密码，输入即可。
```bash
************************************************************
*                                                          *
*               ServerSpeeder Installer (1.2)              *
*                                                          *
************************************************************

Email address: #注册邮箱
Password: #密码
```
接下来是配置部分，按照如下进行~:

```bash

Enter your accelerated interface(s) [eth0]: #回车默认

Enter your outbound bandwidth [1000000 kbps]: #回车默认

Enter your inbound bandwidth [1000000 kbps]: #回车默认

Configure shortRtt-bypass [0 ms]: #回车默认

Auto load ServerSpeeder on linux start-up? [n]:y #是否开机自启

Run ServerSpeeder now? [y]:y #是否现在启动

```


待安装完成，使用以下命令即可启动锐速加速:

```bash
/serverspeeder/bin/serverSpeeder.sh start
```
启动之后，是不是感觉速度不升反降啦？这就对啦~所以接下来我们还需要对锐速进行一些优化配置：

编辑` /serverspeeder/etc/config`文件：
```bash
vim  /serverspeeder/etc/config
```
改动以下参数:

```bash
advinacc="1"

maxmode="1"

rsc="1"

gso="1" #主要是针对Digitalocean，其他的VPS小伙伴就请自测啦

accppp="1" #开启VPN加速~

```

修改完毕之后，保存退出，执行以下命令重启锐速然修改生效~
```bash
/serverspeeder/bin/serverSpeeder.sh restart
```

不知道为何，博主虽然在安装锐速的时候配置的开机自启，但是重启系统之后，锐速并没有自启，因此，在`/etc/rc.local`中`exit 0`前加上一句：
```bash
/serverspeeder/bin/serverSpeeder.sh start
```
退出保存，现在再连上[YouTube](https://www.youtube.com/watch?v=-YGDyPAwQz0)看看速度~如下图所示：

![](http://7xkj6q.com1.z0.glb.clouddn.com/blog/image/jpg/113459120364b2ec84o.jpg)

是不是相对之前就快多了呢~~博主调的是4K分辨率，虽然有些卡顿，不过1080P是完全木有问题的~
至此`Shadowsocks-Python`在[Digitalocean](https://www.digitalocean.com/?refcode=f07fa4a5aaf1)上的搭建、优化就完成啦~

### [Digitalocean优惠](https://www.digitalocean.com/?refcode=f07fa4a5aaf1)

>如果你还没有Digitalocean账户，想注册一个来试试的话，可以试试博主的推荐链接哦~这样我们都会得到$10美元的账户充值，购买最低配置的服务器5美元一个月足够使用两个月咯~

 - [**点我注册Digitalocean账户**](https://www.digitalocean.com/?refcode=f07fa4a5aaf1)
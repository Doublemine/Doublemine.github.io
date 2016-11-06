title: "Debian下配置exim4 邮件服务器"
date: 2015-04-27 11:19:53
isupdate: true
updatetime: 2015-07-19
categories: Linux
tags:
 - Debian
 - Linux
 - Exim4
 - 服务器
---

###前言

>由于博主在[DigitalOcean](https://www.digitalocean.com/?refcode=f07fa4a5aaf1)上购入了一台VPS,并在上面搭建托管了一些服务，不定时的需要登录到服务器上进行管理查看错误日志，比较麻烦。于是打算编写一个`shell`脚本，来监控服务器的一些问题，并在发现问题的时候自动邮件通知到我。


<!-- more -->

>因此，第一步当然得是有一个可用的MTA服务器了。还好Debian默认安装了`Exim4`,可以直接使用。


###安装exim4

 - 如果你的服务器或者Linux上没有安装`exim4`，请执行以下命令进行安装：

```bash
# apt-get install exim4-daemon-light mailutils
```

###配置exim4

>配置`exim4`有两种方式:

 1. 使用图形化界面配置
 2. 编辑配置文件进行配置


在这里我们还是选择比较亲民的图形化界面配置吧。。（博主懒~~）

####图形化界面配置exim4



在终端中运行以下命令:

```bash
# dpkg-reconfigure exim4-config
```

#####配置邮件系统类型

>将会出现一个图形化的配置界面。如下图所示:

<center>![](http://i3.piimg.com/5468f666400f4765.jpg)</center>

根据自己的需要选择:

 - 这里博主考虑到只需要能够发送邮件即可，加上安全性考虑，选择`mail sent by smarthost;no local mail`。
 - 如果你是要搭建邮件服务器，进行接收和发送邮件。请选择`internet site;mail is sent and received directly using SMTP`。

#####系统邮件名称

>回车过后，将会来到配置系统邮件名称的界面，如图:

<center>![](http://i3.piimg.com/6b56408a64016c4f.jpg)</center>

此处填写你用来发送邮件的邮件域名。例如:你的邮件地址为:<font color=red>`xiamo@gmail.com`</font>,则此处填写为<font color=red>`gmail.com`</font>

#####监听入站STMP连接IP地址

>回车过后，将会配置exim4要监听的入站STMP连接的ip地址，如图:

![](http://i3.piimg.com/60bfc1ae6462025a.jpg)


博主填写的为`127.0.0.1`，这表示只监听本地端口，也就是只有本机才能够发邮件，外部不能访问。


#####其它可接收邮件的目的地址

>回车过后，将会配置其它可以接收邮件的目的地址，如图：

![](http://i3.piimg.com/8c8a5fd6997e07f8.jpg)

此处直接留空即可。


#####配置本地可见域名

>回车确认之后，将会配置本地用户可见的域名，如图:

![](http://i3.piimg.com/0361c3bae4e09b4b.jpg)


此处和配置`系统邮件名称`一样，填写发邮件的邮箱域名。例如:`gmail.com`。


#####配置发邮件的主机名。

>回车确认之后，将会配置发邮件使用的smarthost的ip地址或主机名，如图:

![](http://i3.piimg.com/fb1bc44dba574379.jpg)

此处填写外部的STMP地址，因为博主使用的gmail，所以填写的是Gmail的STMP地址:`smtp.gmail.com`。

#####配置DNS查询量

>回车确认之后，将会配置DNS查询量，如图:

![](http://i3.piimg.com/869d4e0d3ef4c9bc.jpg)

此处选择`NO`，来提高响应速度。如果您的服务器资源宝贵可以使用`Yes`。不过可能会出现一些延迟问题。

#####配置文件拆分

>回车过后，将会询问你是否要将配置文件拆分为小文件，如图:

![](http://i3.piimg.com/142101ddb8358d8f.jpg)

此处建议选择`NO`，如果你不搭建复杂的邮件服务器的话，保持默认就可以了。不用拆分配置文件。


>至此，图形化界面就完成配置了。不过呢，现在还不能发送邮件，还需要一些其他的配置。

####修改配置文件

#####配置发送邮箱的账号和密码

现在需要配置发送邮件的邮箱账号和密码，需要编辑`/etc/exim4/passwd.client`这个文件。使用命令:

```bash
# vim /etc/exim4/passwwd.client
```
在文件末尾加入以下内容:
```bash
*:xiamo@gmail.com:password
```
格式为: <font color=red>`发件邮箱STMP服务器:发件邮箱账号:发件邮箱密码`</font>,因为我们在图形化界面中已经配置过STMP服务器地址了，所以这里可以使用通配符`*`代替。当然，你也可以填写SMTP地址:

```bash
    stmp.gmail.com:xiamo@gmail.com:password
```


#####配置系统邮箱

现在，我们需要配置系统`mail`命令的默认邮箱，需要编辑`/etc/email-addresses`这个文件。使用命令:

```bash
# vim /etc/email-addresses
```

在文件末尾加入以下内容:

```bash
    root: xiamo@gmail.com
```

格式为:<font color-red>`系统用户名: 发件邮箱地址`</font>。注意：冒号`:`后有一个空格。

#####配置exim4，使其支持明文密码

现在，我们需要对`/etc/exim4/exim4.conf.template`这个文件进行编辑，使用命令:

```bash
# vim /etc/exim4/exim4.conf.template
```
在`1996`行上一行，也就是`cram_md5:`上方（博主的文件是1996行）加入以下内容:

```bash
AUTH_CLIENT_ALLOW_NOTLS_PASSWORDS=1
```

然后保存退出。

键入以下命令重启`exim4`：
```bash
# service exim4 restart 
```
至此，exim4邮件配置就完成了。是不是很简单呢~~

###发送测试邮件

配置完成需要发送一封测试邮件看看服务可用不呢~，键入命令:

```bash
# mail wanghaokoko1994@gmail.com
```
其中，`wanghaokoko1994@gmail.com`为你想要发送到的邮箱地址，接着按提示输入主题、邮件内容。然后再使用`Ctrl` + `D`组合键完成输入，即可完成发送:
```bash
mail wanghaokoko1994@gmail.com
Subject: test email
hello,this is a test email!
EOT
```

###可能出现的错误

有的时候可能会遇到exim4抛出的错误，如下:
```bash
debian ALERT: exim paniclog /var/log/exim4/paniclog has non-zero size, mail system possibly broken
```
针对这个情况，可以使用以下方法解决:

 - 停止`exim4`

```bash
service exim4 stop
```
 - 删除`paniclog`文件

```bash
 rm /var/log/exim4/paniclog
```
 - 启动`exim4`

```bash
service exim4 start
```


这样，错误问题就完美解决了~


 



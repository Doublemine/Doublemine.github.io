title: 部署PPTP and OpenConnect VPN
isupdate: true
date: 2015-08-28 23:17:55
updatetime:  2015-09-2 12:31:55
tags: Proxy
categories: 折腾
---

由于几天之前[@clowwindy](https://github.com/clowwindy)被某部门请喝茶，导致现Github上Shadowsocks项目源码被迫全部清除。非常感谢clowwindy所做的一切，希望他一切安好。



虽然目前Shadowsocks还能正常使用，但是由于项目不在更新维护，还是要准备Shadowsocks的替代工具，以备不时之需，毕竟经常要上Google以及stackoverflow以及时不时看看YouTube。



##PPTP Server

如果对速度要求较高，又不那么在乎连接的安全性的话，PPTP VPN 是一个不错的选择，有着兼容性较好,速度快配置简单的优点。当然也有着加密强度不够，连接强度、抗干扰能力弱等缺点，但是基本上够用咯~

<!--more-->

###配置方法

如何配置PPTP VPN服务器，博主写了一个简单的脚本来自动一键配置PPTP，脚本地址：[https://github.com/smilexiamo/AutoProxyScript](https://github.com/smilexiamo/AutoProxyScript)

具体用法此处就不再赘述，运行脚本等着配置完成即可使用。

**需要注意的是该脚本运行环境为Debian或者Ubuntu。不适用于CentOS之类的Red Hat系Linux。**

###添加PPTP用户

编辑`/etc/ppp/chap-secrets`文件：
```bash
vim /etc/ppp/chap-secrets 
```
按照如下格式添加用户，每行一个即可：
```bash
 # Secrets for authentication using CHAP
 # client    server       secret       IP addresses
   vpn_user01  pptpd    vpn_password01     *
   vpn_user02  pptpd    vpn_password02     *

```

##OpenConnect Server

对于某一些安全性要求比较高的场合，PPTP可能就不那么适合了，这个时候就可以使用OpenConnect VPN了。它通过实现Cisco的AnyConnect协议，用DTLS作为主要的加密传输协议。其主要的有点有：

 - AnyConnect的VPN协议默认使用UDP DTLS作为数据传输，但如果有什么网络问题导致UDP传输出现问题，它会利用最初建立的TCP TLS通道作为备份通道，降低VPN断开的概率。
 - AnyConnect作为Cisco新一代的VPN解决方案，被用于许多大型企业，这些企业依赖它提供正常的商业运作。
 - OpenConnet的架设足够麻烦，如果你不是大型企业，你会用AnyConnect的概率无限趋近于零。
 - 支持自定义路由表，因此是Shadowsocks比较好的替代方案。

当然啦，有这么多有点，那么缺点也比较明显了：配置麻烦复杂。

###配置方法

有关如何配置的问题，这里有一个博主测试可用的还算比较干净的脚本，大大简化了配置的流程：[https://github.com/fanyueciyuan/eazy-for-ss/tree/master/ocservauto](https://github.com/fanyueciyuan/eazy-for-ss/tree/master/ocservauto)

先运行博主的脚本，配置好PPTP，在运行此脚本配置好OpenConnect Server。博主测试这两种服务器是可以同时运行的。

###连接方法
虽然OpenConnect Server兼容Cisco的Anyconnect，但是博主在[windows](https://software.cisco.com/download/navigator.html?mdfid=278875403&flowid=17001)以及[Android平台](https://play.google.com/store/apps/details?id=com.cisco.anyconnect.vpn.android.avf)使用Cisco官方的客户端均出现可以连接但是无法访问网络的问题。改用OpenConnect Server官方推荐的客户端就正常了。以下给出OpenConnect Server官方客户端链接：

 - [Windows OpenConnect-GUI](https://github.com/openconnect/openconnect-gui/releases)
 - [MacOSX OpenConnect-GUI](https://github.com/openconnect/openconnect-gui/wiki/MacOSX)
 - [Android OpenConnect-GUI](https://play.google.com/store/apps/details?id=app.openconnect)

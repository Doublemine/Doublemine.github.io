---
title: Docker非Root运行
tags:
  - Docker
date: 2017-07-11 10:56:01
---


{% note info %}Docker Engine的Deamon进程是以root权限运行的，如果是普通用户要与之交互，需要使用`sudo`命令来提权与之交互。之前使用Docker官方的安装脚本安装完成之后，会给出一个提示将当前非root用户添加到doker组之中，以避免每次都需要输入`sudo`的麻烦。{%endnote%}

然而随着Docker版本的迭代和官网的安装方式的更改，现在官方给出的安装方式是添加仓库源地址，然后使用默认的`apt`或者`yum`包管理工具来完成后安装。并不再提示用户添加非root用户到组。


默认情况下，完成Docker Engine的安装之后，Docker将会自动创建一个名为`docker`的用户组，所以`root`用户和在`docker`组中的用户都可以免去`sudo`来与Docker Engine交互。知道原理之后就简单了：

```shell
sudo usermod -aG docker ${whoami} #添加当前用户到docker组
```


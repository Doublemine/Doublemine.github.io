---
title: MySQL API Drivers安装小记
date: 2017-05-20 02:12:46
tags: 
 - Python
---

#### 前言

最近需要使用Django写点东西，由于自己的macbook上没有也不打算安装MySQL而是以Docker的MySQL镜像替代，Django文档提供了三种MySQL驱动供选择，官方推荐的是[`mysqlclient`](https://pypi.python.org/pypi/mysqlclient),由于我本地没有安装MySQL，所以是没有Native Driver的以至于在安装MySQL驱动的时候遇到了点小问题，在此记录下。

--------

<!--more-->

#### 过程

安装`mysqlclient`：

```she
pip install mysqlclient
```

然而得到错误信息如下：

```Shell
Collecting mysqlclient
  Using cached mysqlclient-1.3.10.tar.gz
    Complete output from command python setup.py egg_info:
    /bin/sh: mysql_config: command not found
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/private/var/folders/ll/d2yyxp555ss1gm613y33hfy80000gn/T/pip-build-mcd2q8ly/mysqlclient/setup.py", line 17, in <module>
        metadata, options = get_config()
      File "/private/var/folders/ll/d2yyxp555ss1gm613y33hfy80000gn/T/pip-build-mcd2q8ly/mysqlclient/setup_posix.py", line 44, in get_config
        libs = mysql_config("libs_r")
      File "/private/var/folders/ll/d2yyxp555ss1gm613y33hfy80000gn/T/pip-build-mcd2q8ly/mysqlclient/setup_posix.py", line 26, in mysql_config
        raise EnvironmentError("%s not found" % (mysql_config.path,))
    OSError: mysql_config not found

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /private/var/folders/ll/d2yyxp555ss1gm613y33hfy80000gn/T/pip-build-mcd2q8ly/mysqlclient/
```

因为没有安装MySQL，所以在安装`mysqlclient`之前还需要安装Connector，如下：

```shell
brew install mysql-connector-c
```

之后安装再安装`mysqlclient`：

```Shell
pip install mysqlclient
```

然后又就报错了，错误信息如下：

```Shell
Collecting mysqlclient
  Using cached mysqlclient-1.3.10.tar.gz
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/private/var/folders/ll/d2yyxp555ss1gm613y33hfy80000gn/T/pip-build-smwmu1qn/mysqlclient/setup.py", line 17, in <module>
        metadata, options = get_config()
      File "/private/var/folders/ll/d2yyxp555ss1gm613y33hfy80000gn/T/pip-build-smwmu1qn/mysqlclient/setup_posix.py", line 54, in get_config
        libraries = [dequote(i[2:]) for i in libs if i.startswith('-l')]
      File "/private/var/folders/ll/d2yyxp555ss1gm613y33hfy80000gn/T/pip-build-smwmu1qn/mysqlclient/setup_posix.py", line 54, in <listcomp>
        libraries = [dequote(i[2:]) for i in libs if i.startswith('-l')]
      File "/private/var/folders/ll/d2yyxp555ss1gm613y33hfy80000gn/T/pip-build-smwmu1qn/mysqlclient/setup_posix.py", line 12, in dequote
        if s[0] in "\"'" and s[0] == s[-1]:
    IndexError: string index out of range

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /private/var/folders/ll/d2yyxp555ss1gm613y33hfy80000gn/T/pip-build-smwmu1qn/mysqlclient/
```

------



#### 解决

通过查找资料得出可能的结论是通过brew安装的`mysql-connector-c`配置可能不正确，打开`/usr/local/bin/mysql_config`脚本修改其中的部分内容:

```sh
# Create options
libs="-L$pkglibdir"
libs="$libs  -l"
```

修改为：

```shell
# Create options
libs="-L$pkglibdir"
libs="$libs  -lmysqlclient -lssl -lcrypto"
```

保存，再次安装`mysqlclient`应该就会正常安装了。接着就可以使用Django和运行在Docker中的MySQL愉快的Coding了～


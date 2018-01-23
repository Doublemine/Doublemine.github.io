title: "Linux 学习笔记-软件包管理"
date: 2015-04-24 10:06:02
categories: 学习笔记
tags: 
- linux
- rpm
- apt
- yum
---



# RPM软件包管理

### rpm包管理查询
参数选项:
- `-a` 查询所有已安装的软件包
- `-f` 查询文件所属软件包
- `-p` 查询软件包
- `-i` 显示软件包信息
- `-l` 显示软件包中的文件列表
- `-d` 显示被标注为文档的文件列表
- `-c` 显示被标注为配置文件的文件列表

<!--more-->

>rpm查询应用案列:


 0. 查看软件是否被安装:

>`rpm -q xxxx软件包名`将显示指定的软件包名是否被安装。
>以下示例将显示`sudo`命令是否被安装:

```bash
# rpm -q sudo
```

>`-qa`将会显示所有被安装的软件包，从中过滤指定的软件包，可以使用管道组合命令:
>` rpm -qa | grep xxxx软件包名`来查看指定软件包是否被安装。

以下示例将查询`sudo`命令是否被安装:

```bash
# rpm -qa | grep sudo
```


 1. 查询文件隶属的软件包: 
>`rpm --qf xxxx软件包名` 其中`f`代表file的意思。

以下示例为查询`/etc/services`文件所属的软件包:

```bash
# rpm -qf /etc/services 
```
 2. 查询软件包信息:
>`rpm -qi xxx软件包名`查询已安装包信息
>`rpm -qip xxxx.rpm`查询指定rpm软件包信息（未安装包信息）

以下示例为查询`sudo`命令软件包的信息:

```bash
# rpm -qi sudo 
```
 3. 查询软件包安装文件：
>`rpm -ql xxx软件包名`查询已安装软件包安装了哪些文件
>`rpm -qlp xxxx.rpm`查询指定的rpm软件包将会安装哪些文件（未安装包安装将会安装的文件） 

以下示例为显示`sudo`命令软件包安装的文件:

```bash
# rpm -ql sudo
```
 4. 查询软件包帮助文档:
>`rpm -qd xxx软件包名` 查询已安装的软件包安装的帮助文档
>`rpm -qdp xxx.rpm` 查询指定rpm软件包将会安装的帮助文档（未安装软件包将会安装的帮助文档）

以下示例将会显示`sudo`命令安装的帮助文档:

```bash
# rpm -qd sudo
```
 5. 查询软件包配置文件： 
>`rpm -qc xxxx软件包名` 查询已安装软件包包含的配置文件
>`rpmm -qcp xxxx.rpm` 查询指定rpm软件包将会安装的配置文件（为安装软件包将会安装的配置文件）

以下示例将会显示`sudo`命令安装的配置文件:

```bash
# rpm -qc sudo
```


学会这些，可以查看linux系统中的文件隶属于那个软件包，再查看该软件包的信息，以确定该软件包是否需要来确定软件包的开启与否。来达到服务器的精简不必要的软件运行以降低负载。

### rpm软件包的安装、卸载

#### 卸载软件包:
```bash
#  rpm -e xxx软件包名
```
>其中`xxx软件包名`为软件包名称。
>注意:如果其它软件包有依赖关系，卸载时会产生提示信息。可以使用--nodeps强行卸载。

以下为强制卸载`samba`软件包 示例:

```bash
# rpm -e --nodeps samba
```

#### 安装本地rpm软件包:
```bash
# rpm -ivh xxxxxxxxx.rpm
```
>其中`-ivh`代表的意思为:
>- i :安装（install）
>- v :显示安装过程中的详细信息
>- h :用`#`号显示安装进度（hash）
>- U :用大写的U来表示升级安装（update）

升级安装示例:
```bash
# rpm -Uvh xxxxx.rpm
```

如果想忽略进度和安装信息，直接安装可使用:

```bsah
# rpm -i xxxxxxxxx.rpm
```
#### 安装本地软件包附带参数说明:

`--excludedocs`
>不安装软件包中的文档文件(如果你已经足够熟悉这个软件包并且服务器空间紧张的话）

以下为示例:

```bash
# rpm -ivh --excludedocs xxxxx.rpm
```

`--prefix PATH`
>将软件包安装到由`PATH`指定的路径下,通常不建议这样做或者说这样指定无效果。因为大部分rpm软件包在打包的时候开发人员指定了安装时软件包文件存放的位置了。

以下为示例:

```bash
# rpm -ivh --prefix=/usr/xiamo/ xxxx.rpm
```

`--test`
>只对安装进行测试，并不实际安装。实际环境中比较有用的一个参数，可以测试安装软件包需要哪些依赖需要是否存在安装兼容性问题等等。

以下为示例:

```bash
# rpm -ivh --test xxxx.rpm
```

`--replacepkgs`
>覆盖安装
>软件包已经被安装,如果要覆盖安装该软件包，例如某个软件包的某个文件丢失，就可以覆盖安装。可以再命令行上使用该参数.

以下为示例:

```bash
# rpm -ivh --replacepkgs xxxx.rpm
```

`--replacefiles`
>文件冲突
>如果要安装的软件包中有一个文件已在安装其它软件包时安装，会出现错误信息，要想让rpm忽略该错误信息，请使用命令行选项。

以下为示例:

```bash
# rpm -ivh --replacefiles xxxx.rpm
```

`--nodeps`
>未解决依赖关系
>rpm软件包可能依赖于其它软件包，在安装了特定的软件包之后才能安装该软件包。你必须安装完所有依赖的软件包，才能解决这个问题。强制安装使用该选项。事实上，这样做安装好的软件包十有八九是无法运行的。这也是rpm比较头疼的地方。还好yum解决了这个问题:)

以下为示例:

```bash
# rpm -ivh --nodeps xxx.rpm
```
### rpm软件包校验

>在linux日常应用中，可能其它管理员误操作将某个软件包的文件更改了，这个时候，为了查看哪些文件被更改，就必须校验软件包是否被更改。
>`rpm -V 软件名称`将会校验指定的已安装的软件包。
>- 如果软件包没有被做过任何修改，那么执行该命令将不会有任何提示信息。
>- 如果软件包被改动过，那么将会出现提示信息。

以下为提示信息代表的含义:

- `5` 文件的md5校验值
- `S` 文件大小
- `L` 链接文件
- `T` 文件的创建时间
- `D` 设备文件按
- `U` 文件的用户
- `G` 文件的用户组
- `M` 文件的权限

### rpm软件包文件提取

- 解压所有文件到当前目录
```bash
# rpm2cpio xxxxxx.rpm | cpio -idv
```

- 解压指定文件到当前目录

>以下命令将指定的rpm解包，并将其中的的`/etc/inittab`文件提取到当前目录得`etc`目录下。（当前目录的`etc`目录下将会有一个`inittab`文件。)
```bash
# rpm2cpio xxxx.rpm | cpio -idv ./etc/inittab
```


## yum软件包管理工具

应用yum包管理工具的好处:

> - 自动解决软件包依赖关系
> - 方便的软件包升级
> - 软件包可信任

缺点:
> -  比rpm慢（需要读取yum源）
> -  需要联网

### yum软件包常用选项:

- 安装
>`yum install xxx软件包名`将会在yum源查找并安装该软件包。

以下为安装`sudo`命令示例:
```bash
# yum install sudo
```
- 检测升级 
>`yum check-update xxxx软件包名`将会在yum源查询指定软件是否有更新可用。(不会安装更新）

以下为检测`sudo`命令是否有更新可用示例:
```bash
# yum check-update sudo
```
><font color=red>如果不指定软件包名，将检测升级所有软件包。（只检测，不进行升级操作)</font>

以下为检测所有安装的软件包是否有更新可用:
```bash
# yum check-update
```
- 升级
>`yum update xxx软件包名`将会更新升级指定的软件包（如果有更新可用的话），因此次命令最好配合`yum check-update`使用。

以下为升级安装`sudo`软件包示例：
```bash
# yum update sudo
```
- 软件包查询
>`yum list`列出yum源上所有的软件包,并标注哪些软件包已安装，哪些软件包可更新。（此命令不时列出系统安装的软件包哦，而是yum源上的。所有一般不建议直接这样执行。可以使用管道配合`more`命令来分页查看。

以下为使用示例:
```bash
# yum list
# yum list | more
```
列出yum源上指定的软件包可以管道配合`grep`，以下为示例:

```bash
# yum list | grep sudo
```
- 软件包信息
>`yum info xxx软件包名` 列出软件包的详细信息以及软件包的作用介绍。

以下示例将会列出`sudo`命令的详细说明,以及该软件包的作用介绍:
```bash
# yum info sudo
```
- 卸载
>`yum remove xxxx软件名` 卸载指定的软件包

以下示例将会卸载`sudo`命令:
```bash
# yum remove sudo
```
- 帮助
>查看更多的关系yum的帮助信息,来更好的了解yum软件包管理机制并且灵活的使用。

示例:
```bash
# yum --help
# man yum
```

## 源代码包安装

>源码安装不同于二进制包安装，它有着很高的自由度。可以个性化定制。但是也往往存在着缺少编译器而无法正常编译安装以及安装之后配置文件的管理问题。

应用举例:
>此处用`proftpd`举例。

```bash
# tar -zxvf proftpd-1.3.3d.tar.gz (解压源码包）
# cd proftpf-1.3.3d （进入源码包目录）
# ./config --prefix=/usr/local/proftpd （配置安装路径）
# make (编译）
# make install  （安装）
```
>软件下载地址: [www.proftpd.org](http://www.proftpd.org)

其中，配置安装路径是非常必要的，因为源码安装不同于二进制安装。通常没有统一的卸载命令。存在管理不方便以及卸载不干净等问题。因此需要统一指定安装目录。习惯性的，将`/usr/local`设置为软件的安装目录，后面再跟上软件包的名称建立安装文件夹。


## 脚本安装

>过于简单，就不解释了。。。

## APT软件包管理

- 搜索软件包
>`apt-cache search xxx软件包名`将会搜索指定的软件包，并列出其所有的相关软件的信息。类似于`yum list`

以下示例将会列出`sudo`命令的软件包:
```bash
# apt-cache search sudo
```
- 软件包信息
>`apt-cache show xxx软件包名`将会显示软件包的详细信息，并列出其作用介绍。类比于`yum info`。

以下示例将会显示`sudo`软件有关详细信息:
```bash
# apt-cache show sudo
```
- 安装
>` apt-get install xxx软件包名`将会安装制定的软件包。
>- `reinstall`参数替换install将会覆盖安装制定软件包。
>- `-f`参数将会修复安装制定软件包。

以下示例将会安装、覆盖安装、修复安装`sudo`：
```bash
# apt-get install sudo (安装）
# apt-get reinstall sudo （覆盖安装）
# apt-get install -f sudo （修复安装）
```
- 删除卸载
>`apt-get remove xxx软件包名`将会卸载指定的软件包。
>- `autoremove`将会自动卸载依赖关系软件包（连同主程序的依赖一起卸载）
>- `--purge`连同软件的配置文件一起删除（不保留配置文件）。

以下示例将会卸载、连同依赖一起卸载、不保留配置文件卸载`sudo`：
```bash
# apt-get remove sudo (卸载）
# apt-get autoremove sudo （连同卸载sudo的依赖）
# apt-get remove --purge （卸载sudo以及配置文件）
```
- 更新软件源
>`apt-get update`更新软件源

示例:
```bash
# apt-get update
```
- 更新已安装软件包
>` apt-get upgrade`将会对已经安装的软件包进行更新。

示例:
```bash
# apt-get upgrade
```
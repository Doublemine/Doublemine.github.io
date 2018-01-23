title: 基于Debian Linux搭建Git服务器
date: 2015-07-18 17:47:37
isupdate: true
updatetime: 2015-07-20
tags: 
- Git
- Linux
categories: Linux
---

&ensp;&ensp;&ensp;&ensp;因为一些众所周知的原因，某些项目不适合开源（代码写的烂我会乱说？(╯﹏╰)b ）。又因为博主不是壕，买不起Github的私有空间，所以只能利用手头上的Debian服务器搭建Git服务器咯~



-----

### 安装Git

    要搭建Git服务器，第一步当然是要安装Git了，键入以下命令来完成Git的安装：

```bash
sudo apt-get install git
```

### 新建Git用户

&ensp;&ensp;&ensp;&ensp;出于安全的原因考虑，我们肯定是不会使用`root`或者其它具有完整的shell执行权限的用户来运行Git的。因此，我们需要创建一个`git`用户来运行`git`服务。

键入以下命令来完成`git`用户的创建：

```bash
sudo adduser git
```

创建用户的途中会要求输入用户密码，这个密码请务必记住，这个密码的作用我们后面再说。`git`用户的其它配置使用默认值直接回车就行。

### 禁用shell登录

`git`用户创建完成之后，系统默认是为其分配的`bash`的，我们不希望`git`用户拥有shell执行权限，因此我们需要更改`git`用户的默认shell，使其不允许登录shell。

我们可以通过编辑<font color=red>`/etc/passwd`</font>文件来完成对`git`用户shell的更改。

键入以下命令编辑<font color=red>`/etc/passwd`</font>文件：

```bash
vim /etc/passwd
```
<!--more-->

找到类似下面的一行（一般在文件的末尾）：

```bash
git:x:1000:1000:,,,:/home/git:/usr/bin/bash
```
修改为：
```bash
git:x:1000:1000:,,,:/home/git:/usr/bin/git-shell
```
如果你不确定<font color=red>`git-shell`</font>是不是在<font color=red>`/usr/bin/git-shell`</font>这个位置的话，可以使用以下命令来查看`git-shell`的路径：
```bash
which git-shell
```

这样，`git`用户可以正常通过`ssh`使用git，但无法登录shell，因为我们为`git用户`指定的`git-shell`每次一登录就自动退出。

### Git密钥登录

既然是私有Git服务器，当然必要的`push`和`pull`以及`clone`等操作不能让其他未经允许的人使用了。当然，我们也不想每次进行远程仓库操作的时候都输入`git`用户的密码。

因此，通过ssh密钥证书的方式就很有必要了~

- 如同Github导入公钥一样，首先收集授权访问用户的公钥文件，也就是他们的<font color=red>`id_rsa.pub`</font>文件。
- 复制`id_rsa.pub`文件其中全部的内容。
- 将<font color=red>`id_rsa.pub`</font>文件其中全部的内容导入到Git服务器的<font color=red>`/home/git/.ssh/authorized_keys`</font>文件中，每一行导入一个公钥文件。

如果没有<font color=red>`/home/git/.ssh/authorized_keys`</font>文件，请执行以下命令自行创建：
```bash
cd /home/git
mkdir .ssh
cd .ssh
touch authorized_keys
```
**这样，凡是添加了公钥到<font color=red>`/home/git/.ssh/authorized_keys`</font>文件的用户都能够正常的使用远程仓库的常用操作了，如果Git公钥没有被添加到<font color=red>`/home/git/.ssh/authorized_keys`</font>文件中又想执行远程仓库的操作的话，那么就需要用到上面`git`用户的密码了，因为`git`会要求输入Git服务器的`git`用户的密码。这就是上面设置`git`用户密码的作用了~**

### 初始化服务器Git仓库

上面的准备工作做完，就需要在Git服务器上选定一个目录作为Git仓库了。假定我们选择<font color=red>`/gitserver/Demo.git`</font>作为Git仓库，那么我们首先需要执行以下命令创建<font color=red>`/gitserver`</font>目录：

```bash
cd /
mkdir gitserver
cd gitserver
```
然后进入`gitserver`目录，执行以下命令：

```bash
sudo git init --bare demo.git
```
Git就会创建一个裸仓库，裸仓库没有工作区，因为Git服务器上的Git仓库主要是为了共享，所以不让用户直接登录到Git服务器上去修改工作区。

**Git服务器上的Git仓库通常都以<font color=red>`.git`</font>结尾。**

### 修改Git仓库owner

创建好仓库之后，我们需要将仓库的owner设置为`git`用户，不让我们前面为`git`用户所做的配置就没啥意义了~

执行以下命令，修改仓库的owner：
```bash
sudo chown -R git:git demo.git
```

### 克隆远程仓库

现在就可以通过`git clone`命令克隆Git服务器上的`demo`仓库了，在各自的PC上执行以下命令以完成克隆：

```bash
$ git clone ssh://git@server:/gitserver/demo.git
Cloning into 'demo'...
warning: You appear to have cloned an empty repository.
```

其中需要注意的是：

- 上述命令`git clone ssh://git@SERVER:/gitserver/demo.git`中,<font color=red>`SERVER`</font>的值为你的服务器的<font color=red>`ip地址`</font>或者<font color=red>`域名`</font>。
- 如果你的服务器更改了默认的ssh端口号，那么需要在地址中指出，像这样：

```bash
$ git clone ssh://git@server:PORT/gitserver/demo.git
```
其中<font color=red>`PORT`</font>即为更改之后的ssh端口号。

例如，博主对服务器做了A记录的域名解析，也更改了默认端口号为9669，那么执行的克隆地址的命令即为：
```bash
$ git clone ssh://git@server.xiamo.tk:9669/gitserver/demo.git
```

后面的操作不用我多说，大家也都懂的了~
title: "备份Hexo博客源文件"
date: 2015-04-06 22:24:04
categories: Hexo
tags:
- Hexo

---

### 前言
使用Hexo编写博客还是比较Nice的。但是有一个问题就是:Hexo博客的源文件都是放在本地的，如果更换了电脑需要更新博客或者不小心博客源文件丢失，那将是一件非常糟心的事情了。未雨绸缪，现在给出这个问题的一种解决办法吧。
<!-- more -->
### 备份方案
想到的解决办法无非是:
- 将博客源文件拷贝到U盘里。

但是这样做的话，同步又比较麻烦。
- 使用网盘的话，据说.git文件不能上传同步。

而且我对国内的网盘也不怎么放心。。

综合起来，我觉得比较可行的方法就是：
- 将博客文件托管到Github。

### 实现方法:

-  在[Github](http://github.com)下创建一个新的repository，取名为`HEXO`。(与本地的Hexo源码文件夹同名即可)
  -  进入本地的`Hexo`文件夹，执行以下命令创建仓库:

```bash
     git init
```
-  设置远程仓库地址，并更新:

```bash
    git remote add origin git@github.com:smilexiamo/hexo.git
    git pull origin master
```
-  修改`.gitignore`文件（如果没有请手动创建一个），在里面加入`
   *.log` 和 `public/` 以及`.deploy*/`。因为每次执行`hexo generate`命令时，上述目录都会被重写更新。因此忽略这两个目录下的文件更新，加快push速度。
   -  执行命令以下命令，完成Hexo源码在本地的提交。

```bash
    git add .
    git commit -m "添加hexo源码文件作为备份"
```
-  执行以下命令，将本地的仓库文件推送到Github。

```
    git push origin master
```
-  现在在任何一台电脑上，只需要`git clone git@github.com:smilexiamo/hexo.git`,即可完成将Hexo源文件复制到本地。（请将后面的`git@github.com:smilexiamo/hexo.git`替换为自己相应的仓库地址。否则，克隆的将是博主的博客源码:)）
  -  在本地编写完博客时，顺次执行以下命令，即可完成Hexo博客源文件的更新同步，保持Github上的hexo源码为最新版本。
```bash
    git add .
    git commit -m "更新hexo源文件"
    git push origin master
```
-  当远程仓库有更新时，执行以下命令，即可同步hexo源文件到本地。

```
    git pull origin master
```

-  至此，Hexo源代码文件就完成了同步和更新了。
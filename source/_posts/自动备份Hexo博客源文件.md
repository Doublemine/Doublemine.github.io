title: 自动备份Hexo博客源文件
date: 2015-07-06 12:49:01
tags:
- Hexo
categories: Hexo
---

&ensp;&ensp;用Hexo写博客是一件比较享受的事情，无奈如果换电脑的话，备份博客就是一件比较闹心的事情。



### 前言

&ensp;&ensp;我曾经给出过通过Git备份Hexo博客源文件的方式，这种方式虽然能够备份Hexo博客的源文件，但是对于博主这种懒人，每次更新博文都需要输入两三行重复的Git命令真是一件麻烦的事情。况且指不定哪天就搞忘push到github上了。

<!--more-->

### 原理

前两天博主刚刚编写过关于**[Hexo添加文章时自动打开编辑器](https://notes.wanghao.work/2015-06-29-Hexo%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E6%97%B6%E8%87%AA%E5%8A%A8%E6%89%93%E5%BC%80%E7%BC%96%E8%BE%91%E5%99%A8.html)**的相关文章，其原理就是利用`NodeJS`的事件监听机制实现监听Hexo的`new`事件来启动编辑器，完成自动启动编辑器的操作。

那么可不可以通过通过监听Hexo的其它事件来完成自动执行Git命令完成自动备份呢？通过查阅[Hexo文档](https://hexo.io/zh-cn/api/events.html)，找到了Hexo的主要事件，见下表：



| 事件名 | 事件发生时间 |
|--------|--------|
|deployBefore|在部署完成前发布
|deployAfter|在部署成功后发布
|exit|在 Hexo 结束前发布
|generateBefore|在静态文件生成前发布
|generateAfter|在静态文件生成后发布
|new|在文章文件建立后发布

于是我们就可以通过监听Hexo的`deployAfter`事件，待上传完成之后自动运行Git备份命令，从而达到自动备份的目的。

### 实现


#### 将Hexo目录加入Git仓库

本脚本需要<font color=red>`提前`</font>将Hexo加入Git仓库并与Github或者Gitcafe远程仓库绑定之后，才能正常工作。如果你不知道怎么操作，请参考这篇博文：

- **[备份Hexo博客源文件](https://notes.wanghao.work/2015-04-06-%E5%A4%87%E4%BB%BDHexo%E5%8D%9A%E5%AE%A2%E6%BA%90%E6%96%87%E4%BB%B6.html)**




#### 安装`shelljs`模块

要实现这个自动备份功能，需要依赖NodeJs的一个`shelljs`模块,该模块重新包装了`child_process`,调用系统命令更加的方便。（其实就是因为博主懒( ╯▽╰)）该模块需要安装后使用。

在命令中键入以下命令，完成`shelljs`模块的安装：

```bash
npm install --save shelljs
```

#### 编写自动备份脚本

待到模块安装完成，在`Hexo`根目录的`scripts`文件夹下新建一个js文件，文件名随意取。

**如果没有`scripts`目录，请新建一个。**

然后在脚本中，写入以下内容：

```javascript
require('shelljs/global');

try {
	hexo.on('deployAfter', function() {//当deploy完成后执行备份
		run();
	});
} catch (e) {
	console.log("产生了一个错误<(￣3￣)> !，错误详情为：" + e.toString());
}

function run() {
	if (!which('git')) {
		echo('Sorry, this script requires git');
		exit(1);
	} else {
		echo("======================Auto Backup Begin===========================");
		cd('D:/hexo');    //此处修改为Hexo根目录路径
		if (exec('git add --all').code !== 0) {
			echo('Error: Git add failed');
			exit(1);

		}
		if (exec('git commit -am "Form auto backup script\'s commit"').code !== 0) {
			echo('Error: Git commit failed');
			exit(1);

		}
		if (exec('git push origin master').code !== 0) {
			echo('Error: Git push failed');
			exit(1);

		}
		echo("==================Auto Backup Complete============================")
	}
}
```

- **其中，需要修改第<font color=red>`17`</font>行的<font color=red>`D:/hexo`</font>路径为Hexo的根目录路径。（脚本中的路径为博主的Hexo路径）**

- **如果你的Git远程仓库名称不为<font color=red>`origin`</font>的话，还需要修改第<font color=red>`28`</font>行执行的push命令，修改成自己的远程仓库名和相应的分支名。**



保存脚本并退出，然后执行`hexo deploy`命令，将会得到类似以下结果:

```bash
INFO  Deploying: git>
INFO  Clearing .deploy folder...
INFO  Copying files from public folder...
[master 3020788] Site updated: 2015-07-06 15:08:06
 5 files changed, 160 insertions(+), 58 deletions(-)
Branch master set up to track remote branch gh-pages from git@github.com:smilexi
amo/notes.git.
To git@github.com:smilexiamo/notes.git
   02adbe4..3020788  master -> gh-pages
On branch master
nothing to commit, working directory clean
Branch master set up to track remote branch gitcafe-pages from git@gitcafe.com:s
milexiamo/smilexiamo.git.
To git@gitcafe.com:smilexiamo/smilexiamo.git
   02adbe4..3020788  master -> gitcafe-pages
INFO  Deploy done: git
======================Auto Backup Begin===========================
[master f044360] Form auto backup script's commit
 2 files changed, 35 insertions(+), 2 deletions(-)
 rewrite db.json (100%)
To git@github.com:smilexiamo/hexo.git
   8f2b4b4..f044360  master -> master
==================Auto Backup Complete============================
```
这样子，每次更新博文并`deploy`到服务器上之后，备份就自动启动并完成备份啦~是不是很方便呢？

Enjoy it！





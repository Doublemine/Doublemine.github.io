title: Hexo添加文章时自动打开编辑器
date: 2015-06-29 17:39:55
tags: Hexo
isupdate: true
updatetime: 2015-10-19
categories: Hexo

---

[![](http://7xkj6q.com1.z0.glb.clouddn.com/blog/image/jpg/hexo_logo.jpg "阅读全文")](https://notes.wanghao.work/2015-06-29-Hexo%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E6%97%B6%E8%87%AA%E5%8A%A8%E6%89%93%E5%BC%80%E7%BC%96%E8%BE%91%E5%99%A8.html)

<!--more-->

在`Hexo`中新建一篇博文非常简单，只需要在命令行中键入以下命令然后回车即可：

```javascript
hexo new "The title of your blog"
```
此后Hexo便会在Hexo的根目录的`source`文件夹下的`_posts`目录下自动帮你创建相应的md文件。然后我们打开该目录，找到刚刚Hexo自动生成的文件打开编辑即可。

但是当我们的博文比较多，这样我们就需要在成堆的Markdown文件中找到刚才自动生成的文件，这样做显然是一件比较痛苦的事情。

------

在访问[Hexo的Github项目](https://github.com/hexojs/hexo)时，发现有类似的[issue](https://github.com/hexojs/hexo/issues/1007)，Hexo作者也给出来解决办法，以下为作者原文：

>ou can try to listen to the new event. For example:

```javascript
var spawn = require('child_process').exec;

// Hexo 2.x
hexo.on('new', function(path){
  exec('vi', [path]);
});

// Hexo 3
hexo.on('new', function(data){
  exec('vi', [data.path]);
});
```

根据作者给出的示例，一番折腾过后博主终于在自己的机器上实验成功了，下面给出操作步骤：

- 首先在Hexo目录下的`scripts`目录中创建一个JavaScript脚本文件。
- 如果没有这个`scripts`目录，则新建一个。
- `scripts`目录新建的JavaScript脚本文件可以任意取名。

通过这个脚本，我们用其来监听`hexo new`这个动作，并在检测到`hexo new`之后，执行编辑器打开的命令。

如果你是<font color=red>windows平台的Hexo用户</font>，则将下列内容写入你的脚本：

```javascript

var spawn = require('child_process').exec;

// Hexo 2.x 用户复制这段
hexo.on('new', function(path){
  spawn('start  "markdown编辑器绝对路径.exe" ' + path);
});

// Hexo 3 用户复制这段
hexo.on('new', function(data){
  spawn('start  "markdown编辑器绝对路径.exe" ' + data.path);
});
```

如果你是<font color=red>Mac平台Hexo用户</font>，则将下列内容写入你的脚本：

```javascript
var exec = require('child_process').exec;

// Hexo 2.x 用户复制这段
hexo.on('new', function(path){
    exec('open -a "markdown编辑器绝对路径.app" ' + path);
});
// Hexo 3 用户复制这段
hexo.on('new', function(data){
    exec('open -a "markdown编辑器绝对路径.app" ' + data.path);
});
```

保存并退出脚本之后，在命令行中键入：

```javascript
hexo new "auto open editor test"
```
是不是就顺利的自动打开了自动生成的md文件啦~

Enjoy it！
---
title: 解决Git refusing to merge unrelated histories问题
date: 2017-06-28 10:40:36
tags: Git
---

最近在使用brew更新了git之后，发现在与Github上的新创建的`repo`建立关联的之后，进行`pull`操作会出现类似于下面的这种错误：

```shell
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
fatal: refusing to merge unrelated histories
```

通过查阅资料显示，GIt从版本`2.9.0`开始，预设行为不允许合并没有共同祖先的分支，需要加上`--allow-unrelated-histories`选项进行pull操作才不会出现此类错误信息：

```she
git pull origin master --allow-unrelated-histories
```



相关参考：

- [Git merge (2.9.0)](https://git-scm.com/docs/git-merge/2.9.0)
- [StackoverFlow: Git refusing to merge unrelated histories](https://stackoverflow.com/questions/37937984/git-refusing-to-merge-unrelated-histories)
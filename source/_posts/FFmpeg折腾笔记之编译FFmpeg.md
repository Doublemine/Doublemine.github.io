---
title: FFmpeg折腾笔记之编译FFmpeg
date: 2017-01-13 21:58:04
tags: 
     - FFmpeg
     - NDK
     - Android
---

​        又有很久没有写新的文章了，感觉再不写点什么东西博客草都长满了。于是打算折腾下FFmpeg，记录下折腾的过程，以熟悉Android NDK开发的基本操作。过程中有地方有错误欢迎指出，如果你对这个方面有所了解，欢迎讨论指教。

#### 编译环境

- [FFmpeg 2.8.11 "Feynman"](https://ffmpeg.org/releases/ffmpeg-2.8.11.tar.bz2)
- Mac OS X 
- NDK  `14.0.3770861`



下载完成FFmpeg源码之后，先对源码根目录中的`configure`文件进行修改以适应Android平台。因为默认编译出来的动态库文件版本号在`.so`之后，例如：`libavcodec.so.56.60.100`。Android平台对这种格式不能很好的识别（如果你不介意一个一个修改文件名的话）。通过`Vim`或者其他文本编辑器打开`configure`文件的第`2934`行（如果你下载的FFmpeg版本和我的一样的话）将:



```shell
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
```



修改为:

```shell
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'  
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'  
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'  
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

后保存。
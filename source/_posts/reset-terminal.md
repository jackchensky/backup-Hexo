---
title: 关于Mac OS X中Terminal终端如何恢复默认
date: 2017-03-27 17:45:15
categories: MAC
tags: MAC
---
![如何恢复Terminal默认样式](https://img.jackchen.cn/image/jpg/reset_terminal.jpeg)
## 原因
因为换了电脑，所以最近一直在完善各种软件和内核的配置。基本这几天都在填坑中度过。以前各种配置都很随意的设置，现在要自己一个个的重新恢复成老样子就不容易了。这也是一种工作流程上的坏习惯！因为Hexo也要重新部署到新电脑，配置部署填了一整天都没有搞定。还莫名其妙的把系统终端`Terminal`弄成了个奇怪的样式，而且在界面设置里怎么都恢复不过来。如下图的样子~
<!--more-->
![乱掉的样式](https://img.jackchen.cn/image/jpg/reset_terminal_02.jpeg)

## 解决方法
网上找了好一阵子各种修改，大多数都没有能够解决。最后找到了几篇文章可以完美解决，具体方法总结如下

在终端`Terminal`中输入

```
echo $PS1
```

这时会显示类似\h:\W \u\$这样的一段信息，这些信息就是用来定义提示符的显示方式，具体的细节下边会列出。

继续在`Terminal`里输入

```
cd~
open -e .bash_profile
```

这个时候会打开`TextEdit`，按照你的样式要求，可以在最后加上下面这句

```
export PS1="\u \w$"
```

或者是`Ubuntu`的那种风格

```
export PS1="\u@\h:\w $"
```

然后保存退出。

如果没有`.bash_profile`文件的话就新创建一个，在终端中输入如下

```
cd~
touch .bash_profile
```

然后重启后就发现`Terminal`终端可以恢复默认的样式啦....


## 相关资料
https://www.zhihu.com/question/29650903
http://equation85.github.io/blog/customize-terminal-on-mac/
http://osxdaily.com/2006/12/11/how-to-customize-your-terminal-prompt/

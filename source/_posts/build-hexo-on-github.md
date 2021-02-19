---
title: 在GitHub上架设Hexo静态站
date: 2016-09-12 23:11:14
categories: Hexo
tags: Hexo
---
![GitHub上架设Hexo](http://img.jackchen.cn/pic-02.jpg)
## 1.前言
我想重新架设一个站来记录学习中遇到的事件，以便自己以后的回溯和总结。于是就有了这篇文章:D

## 2.准备
首先你得要有一个[GitHub](http://www.github.com)账号。直接去官网注册一下就好。整个网站架设好了就和我的这个站：[jackchensky.github.io](http://jackchensky.github.io)，感觉是一样的。你可以自己先打开来瞅一眼！是不是感觉自己像个coding高手一样。

在下面的架设过程中你需要对下面几个工具熟悉起来。因为以后会一直使用
> * 命令行工具的使用
> * Atom的使用
> * 如何写出优雅的Markdown文档

OK，Let's do it!
<!--more-->

## 3.前期架设
### 3.1.在GitHub上创建一个 Repo
在游览器中登录[GitHub](http://www.github.com)，创建一个 Repo，名称格式为 `yourname.github.io`。比如我的个人GitHub账户用户名是`jackchensky`，所以，我的这个Repo名称就是 `jackchensky.github.io`。

### 3.2.确认本地已经安装好 git 和 npm
在终端或者iTerm里输入一下命令确认git与npm已经成功安装
``` bash
git --version
npm --version
```
### 3.3.安装 Hexo
在终端或者iTerm 中继续输入
``` bash
npm install hexo -g
npm install hexo-cli -g
```
### 3.4.本地创建工作目录
现在去你的GitHub把你的Repo的git地址拷贝出来
> * 我的是 [https://github.com/jackchensky/jackchensky.github.io.git](https://github.com/jackchensky/jackchensky.github.io.git)
> * 你的是[https://github.com/yourname/yourname.github.io.git](https://github.com/jackchensky/jackchensky.github.io.git) (请把yourname换成你的GitHub用户名)。

然后在终端或iTerm中输入
``` bash
cd ~/Public
git clone https://github.com/yourname/yourname.github.io.git
```
### 3.5.让你本地网站初始化
``` bash
hexo init yourname.github.io
cd yourname.github.io
npm install hexo-deployer-git --save
hexo generate
hexo server
```
完成以上步骤之后，你可以打开游览器，在地址栏里输入： `localhost:4000`,看看初始默认网页的样子！

### 3.6.现在部署到GitHub上去
在终端或者iTerm中进入到jackchensky.github.io目录下，输入
``` bash
atom .
```
😝 ...是不是看到Atom自动打开 `yourname.github.io`这个目录下了吧！是不是很神奇。在计算机字符术语中 `.` 代表“所有的”，所以，`atom .`就是说“用Atom 这个程序打开当前目录内的所有文件”... 现在将Atom的左侧面板中找到 `_config.yml`文件，搜索到 deploy ，改成如下：
``` bash
deploy:
  type: git
  repo: httPs://github.com/yourname/yourname.github.io.git
```
PS: 记得要把`yourname`改成你的GitHub用户名！
现在就剩下最后一步了，在终端或者iTerm中输入：
``` bash
hexo deploy
open http://yourname.github.io
```
### 3.7.Hexo基本命令
Hexo通常基本命令就四个，而且还可以使用组合命令。基本命令如下
``` bash
hexo g = hexo generate  #生成
hexo s = hexo server    #启动本地预览
hexo d = hexo deploy    #远程部署
---
hexo n "文章标题" = hexo new "文章标题"    #新建一篇文章
```
通常为了提高效率，选择用组合命令，如下
``` bash
hexo s -g   #等同于输入 hexo g,再输入 hexo s
hexo d -g   #等同于输入 hexo g,再输入 hexo d
```
### 3.8.总结
现在你可以自己来创建文章，并进行体验编辑带来的乐趣和优雅！

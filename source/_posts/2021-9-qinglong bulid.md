---
title: 青龙面板搭建踩坑指南
date: 2021-10-02 02:06:39
categories: Docker
tags:
- Docker
- qinglong
- 京豆
- 薅羊毛
---

![正确薅羊毛的架构方式](https://img.jackchen.cn/moniz-wallpaper-1920.jpg)

## 前言

> 本来对薅羊毛这种事情没太在意，直到有一天发现身边的人都在薅羊毛，瞬间感觉自己少赚了“几个亿”的感觉。然后就研究了下在GitHub上自动化撸羊毛，注册了个GitHub马甲号，才薅了一天账户就直接被封停了😂 说明薅羊毛这种事情已经是一个事件了，不是小众人群了。然后又发现了大牛都在用青龙面板来自动化薅羊毛。手头正好有一天服务器里有装了Docker，上一篇文章我正好有分享给大家[`如何在服务器上配置Docker环境来安装WordPress`](https://me.jackchen.cn/2019/09/03/2019-8-Docker%20bulid%20wordpress/)的教程，（刚看了下，发现上一篇文章还是2019年写的，整个2020年没写一篇文章，我也是服了我自己了~ 😄 ）

## 1.安装配置青龙面板

> 前置要求需要服务器已经安装了Docker-ce，选装了Docker-compose

<!--more-->

符合上面的前置要求后可以按照下面的安装方式来安装青龙面板

> 1. 新建一个文件夹，用于存放相关数据
> 2. 下载本仓库中的`docker-compose.yml`至本地，或是复制文件内容后在本地自行建立并粘贴内容
> 3. 使用docker-compose启动
> 4. 浏览器输入ip:5700即可进入面板

通过ssh连接到服务器后具体的执行方式如下：

```plain
# 新建数据文件夹
mkdir qinglong
cd qinglong
# 下载docker-compose.yml文件
wget https://raw.githubusercontent.com/whyour/qinglong/develop/docker-compose.yml
# 启动
docker-compose up -d
```

## 2.登录

打开浏览器访问宿主机ip的5700端口即可

例如 `http://192.168.100.123:5700即ip:5700`

> 如果页面无法打开，记得看下服务器的 5700 端口有没有打开，我配置完后也无法打开，然后本准备秀一把直接在服务器里用 `firewall-cmd` 命令来打开 5700 端口的，最后发现 5700 端口没能打开，连Docker上其他网站都打不开了，于是就乖乖登录阿里云后台安全策略中添加了5700端口。

具体配置方法可以参考下图：

![端口配置1](https://img.jackchen.cn/%E7%AB%AF%E5%8F%A3%E8%AE%BE%E7%BD%AE-01.png)

![端口配置2](https://img.jackchen.cn/%E7%AB%AF%E5%8F%A3%E8%AE%BE%E7%BD%AE-02.png)

![端口配置3](https://img.jackchen.cn/%E7%AB%AF%E5%8F%A3%E8%AE%BE%E7%BD%AE-03.png)

> 首次登录
>
> 账号:admin 密码:admin
>
> 会生成`auth.json`

在ssh输入

```plain
1.docker exec it qinglong bash
2.cat /ql/config/auth.json
```

cat查看之后返回的结果类似如下字段

```json
{"username":"admin","password":"<your-password>"}
# admin即为登录名;<your-password>为登录密码
```

输入此处记录的`username`及`password`，即可成功登陆qinglong面板，登陆后可以把密码修改成自己熟悉的。

## 3.拉取脚本

可以直接ssh里拉取，方式如下：

```plain
ql repo https://github.com/xxx.git #拉取仓库
ql raw https://raw.githubusercontent.com/xxx #拉取单个脚本
```

也可以登录面板点击右上角`添加定时`，方式如下：

![添加拉取仓库](https://img.jackchen.cn/%E6%8B%89%E5%8F%96%E4%BB%93%E5%BA%93.png)

别忘记在面板界面左侧菜单中的`环境变量`中添加你自己的JD COOKIE

名称一定要填写`JD_COOKIE`，值填写上一步复制的内容，备注可以填写这是谁的账号，可以添加多个账号，但名称都必须是`JD_COOKIE`。添加多个账号的另一个方法：集合在一个环境变量下，多个 cookie 用 & 隔开。(获取不同账号cookie时，要用不同浏览器)

添加了账号和脚本了，可以运行一个脚本试试。随便点击一个脚本右侧的运行，然后点击运行右侧的日志，查看运行情况：

可以看到你绑定的京东账号已经识别了，已经开始执行脚本了，但是有些京东活动是需要去京东app手动开启活动的。这就需要你多关注这些执行日志，配合去app进行某些操作，才能最大化获取京豆。如果你想最大化获取京豆，可以在运行2天之后，进入日志中查看每个脚本的运行日志，看那些脚本需要你手动打开app中的某些服务，然后你配合操作就行，剩下的就交给脚本吧。

ps:新手推荐使用Faker集合仓库，集合仓库内包含本文内所有作者可用脚本，并不断更新 →_→ [`查看大佬的库`](https://www.notion.so/QL-pannel-Pull-GitHub-Repository-Code-da5099068a434e29920cdcb3e7af4410)

pss:如果在面板里拉库没反应的话可以把Docker重启下再看看，本人就是定时添加后一直不成功，研究了半天，几乎放弃的时候重启下Docker居然脚本都好了。。。

## 4.备份

所有数据都将保存在`docker-compose.yml`所在的同级目录的`data`文件夹中，如需要备份，请直接备份`docker-compose.yml`及`data`文件夹即可

```plain
root@jack:/opt/qinglong# ls -lah
总用量 8.0K
drwxr-xr-x 3 root root 4.0K  8月 30 01:29 .
drwxr-xr-x 4 root root 4.0K  8月 30 00:51 ..
drwxr-xr-x 8 root root 4.0K  8月 30 01:30 data
-rw-r--r-- 1 root root  386  8月 30 01:29 docker-compose.yml
```

## 5.更新

在面板执行”更新面板”任务即可

或者

```plain
cd qinglong
docker-compose down
docker pull whyour/qinglong:latest
docker-compose up -d
```

## 6.总结

现在你就阔以安心的让脚本帮你跑所有的程序来薅羊毛了😁 后期有时间再研究下扫码自动添加CK和如何在手机小组件上显示前一天的羊毛量 😁 👏🏻

```plain
* 参考文章:
https://github.com/whyour/qinglong/blob/develop/INSTALL.md#安装方式1
https://github.com/shufflewzc/faker2
https://www.notion.so/QL-pannel-Pull-GitHub-Repository-Code-da5099068a434e29920cdcb3e7af4410
https://v2raytech.com/jd-script-to-get-rich/
https://blog.csdn.net/chinassj/article/details/119871281
https://blog.csdn.net/xiaozhazhazhazha/article/details/119519247?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1.no_search_link&spm=1001.2101.3001.4242
https://blog.csdn.net/weixin_42565036/article/details/117569495
```

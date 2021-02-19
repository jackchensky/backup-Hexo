---
title: 修复阿里云服务器报“wordpress IP验证不当”漏洞
date: 2019-03-24 14:00:13
categories: wordpress
tags:
- wordpress
- 阿里云
---
![修复wordpress IP验证不当](http://img.jackchen.cn/pic-5.jpg)
## 前言
> 嗯，真的好久没有更新了。上一篇文章居然还是2017年的，时间过的真快！最近在阿里云香港服务器上用Docker架设了一个wordpress网站。然后各种折腾好了之后发现阿里云后台总是显示一个漏洞。然后还是高危级别的~强迫症受不了老是看到那个小红标。决定盘它~

## 准备
首先阿里云的后台显示的漏洞名称为`wordpress IP验证不当漏洞`，听起来是WordPress的一个版本Bug。然后点看具体的看了说明里给出的详细说明如下：
> wordpress /wp-includes/http.php文件中的wp_http_validate_url函数对输入IP验证不当，导致黑客可构造类似于012.10.10.10这样的畸形IP绕过验证，进行SSRF。

<!--more-->
## 查询分析
#### 解决办法一：
可以升级你的阿里云后台云盾到企业版（前提是你真的非常有钱），一键自动修复（没试过，好不好用不清楚）
#### 解决办法二：
如果你的网站后台不复杂且经常性的升级wordpress到最新版本，可以尝试升级你的wordpress到官方最新版本（不确定升级后漏洞就会消失，升级前记得要给网站备份，备份，备份！）
#### 解决办法三：
其实很多问题遇到了之后要学会用关键字去Google找答案，嗯，没错~ 是谷歌不是某度！！某度搜出来的都是坑和广告。网上转了一圈后发现问题在`http.php`这个文件上。嗯，目标锁定就开始盘吧~

## 问题解决
#### 连接服务器
我是用SSH连接的服务器，如果你有ftp可以直接找到WordPress安装目录下的`wp-includes/http.php`文件，下载到本地用记事本打开后修改。如果你是用SSH连接，就在终端或者iTerm 中输入
``` bash
ssh root@你的服务器IP地址或域名
```
然后输入密码登录

#### 修改准备
如果你的服务器没有使用Docker容器，那直接找到`wp-includes/http.php`文件，然后vim进入修改就可以了。如果你是用Docker容器安装的。先找到放置容器配置文件的目录下，然后在终端或者iTerm 中输入
``` bash
docker-compose exec WordPress bash
```
这样你就可以进入到这个容器的包文件中，然后找到`wp-includes/http.php`文件，进入`vi http.php`修改。如果提示无法编辑。可能是因为没有安装vi。直接输入`apt-get install vim`来安装Vi。如果提示：Unable to locate package vim,则需要输入`apt-get update`, 等更新完毕以后再敲命令：`apt-get install vim`就OK啦~ 下面开始修改`http.php`这个文件。

#### 修改文件(记得提前备份一下要修改的文件)
1、大概在533行（不同的WordPress版本可能行数不同，你可以查找关键词进行查找）：
``` bash
$same_host = strtolower( $parsed_home['host'] ) === strtolower( $parsed_url['host'] );
/*修改为*/
$same_host = (  strtolower( $parsed_home['host'] ) === strtolower( $parsed_url['host'] ) || 'localhost' == strtolower($parsed_url['host']));
```
2、在http.php文件的549行（不同的WordPress版本可能行数不同，你可以查找关键词进行查找）：
``` bash
if ( 127 === $parts[0] || 10 === $parts[0] || 0 === $parts[0]
/*修改为：*/
if ( 127 === $parts[0] || 10 === $parts[0] || 0 === $parts[0] || 0 === $parts[0]
```
#### 去验证
修改完以上内容，然后再到阿里云盾控制台重新验证一下漏洞，就会发现漏洞已经不存在了。以为这样就解决了，但是第二天打开后台，这个漏洞又冒出来~ 气人！！

#### 第三处修改
各种找了各种尝试后，发现还有一个地方要修改，还是http.php文件的540行
``` bash
preg_match('#^(([1-9]?\d|1\d\d|25[0-5]|2[0-4]\d)\.){3}([1-9]?\d|1\d\d|25[0-5]|2[0-4]\d)$#', $host)
/*修改为：*/
preg_match('#^(([1-9]?\d|1\d\d|25[0-5]|2[0-4]\d|0+\d+)\.){3}([1-9]?\d|1\d\d|25[0-5]|2[0-4]\d)$#', $host)
```
好像是为了既增加对0开头的012.10.10.10这样的IP进行验证！修改完，然后再到阿里云盾控制台重新验证一下漏洞，就会发现漏洞已经不存在了。

## 总结
其实我的个人博客站[设计创意1984](http://www.jackchen.cn/blog)也是wordpress架构的，wordpress大大简化了个人建站甚至企业建站的上限要求。对于很多设计师和前端来说都可以尝试的去学习下wordpress建站的方法。可以快速有效的搭建出各种类型的网站。确实不错！！

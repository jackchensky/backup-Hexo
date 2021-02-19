---
title: 解决Hexo本地生成后出现的错误问题
date: 2016-09-22 17:07:25
categories: Hexo
tags: Hexo
---
![Hexo本地生成后的错误](http://img.jackchen.cn/hexo-error-01.jpg)
## 1.问题
在本地架设好Hexo后一切调试的都挺顺利的，唯一的问题就是每次在执行 `hexo -d` 等命令的时候总是会出现如下的一堆错误提示
```
{ [Error: Cannot find module './build/Release/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/default/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/Debug/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/Release/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/default/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/Debug/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
```

百度谷歌了几次都没能找到好的解决方案，包括 Hexo 官方对于这个错误的解决方法 执行→  `$ npm install hexo --no-optional` 也木有用！
<!--more-->
## 2.发现
今天无意中发现了一个比较笨的解决方法。用一段代码彻底将 `Node` 删除干净了！然后重装一下就OK了哦！👌
棒棒哒，Let's do it!

## 3.解决
创建一个 `shell` 文件，在终端或者 iTerm 中输入如下
```
nano  uninstall_node.sh
```
然后输入下面一段代码后保存退出→
```
#!/bin/bash
lsbom -f -l -s -pf /var/db/receipts/org.nodejs.pkg.bom \
| while read i; do
sudo rm /usr/local/${i}
done
sudo rm -rf /usr/local/lib/node \
/usr/local/lib/node_modules \
/var/db/receipts/org.nodejs.*
```
然后给文件赋权限
```
chmod  777  uninstall_node.sh
```
最后运行脚本,彻底清除node.js
```
sh  uninstall_node.sh
```
现在可以重新安装node.js啦！
```
brew install node
```
最后还要安装一下Hexo,执行如下命令就好，其他配置不需要改变！
```
npm install -g hexo-cli
```
## 4.总结
不管做什么事情问题总是会有的，解决不了不要早早放弃，也不要抓着不放，学会换个方法，换个思路和角度，或许问题就能瞬间消失哦 ！

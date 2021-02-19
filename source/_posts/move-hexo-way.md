---
title: 关于Hexo换电脑后的恢复和配置问题
date: 2017-03-28 09:01:20
categories: Hexo
tags: Hexo
---
![关于Hexo换电脑后的恢复和配置问题](https://img.jackchen.cn/image/jpg/move-hexo-way.jpg)
## 前言
>用了八年的MacBookPro突然当机无法启动了，自己拆了修了两天也没有能够救活，只能换买了新电脑。新电脑收到后各种安装和环境配置真心累。还有恢复Hexo的工作想起来似乎很easy，但是做起来却遇到了各种大小坑。今天记录一下，留着给需要的小伙伴们。

<!--more-->
## 安装需要的开发环境

#### XCode
这个东东安装以前安装很费劲的，因为有好几个G，所以下载等待通常是很漫长的，不过现在可以在终端中直接用命令行来安装了：

```
xcode-select --install
```

#### 安装 Homebrew
`Homebrew` 是一个软件包管理系统。Mac OS X是基于Unix的操作系统，可以安装大部分为Unix/Linux开发的软件。然而，如果只是以使用为目的，对每个软件都进行手工编译不是很方便，也不利于管理已安装的软件，于是出现了类似于Linux中APT、Yum等类似的软件包管理系统，其中最著名的有MacPorts、Fink、Homebrew等。

还是在命令行工具中拷贝粘贴以下代码，而后按回车键
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

随后再次在命令行工具中拷贝粘贴以下代码，而后按回车键

```
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bash_profile
```

#### 安装 Git 工具
`Git` 是一个版本控制工具，它有点像时间机器，可以让你回到从前。使用 `Git` 管理你的项目，对项目做出修改以后，提交一下，写一条信息，说明一下为啥做出这样的修改。在未来你可以恢复到之前提交的那个点上。不管你是设计师还是开发者，都应该学会使用 `Git`。后面我会教大家用 `Git` 工具来备份自己的 `Hexo` 到 `GitHub` 中。

还是在命令行工具中拷贝粘贴以下代码，而后按回车键

```
brew install git
```

#### 安装 rvm 与 Ruby 2.3.1
分别输入下面三行命令
```
\curl -sSL https://get.rvm.io | bash -s stable
rvm install 2.3.1
rvm use 2.3.1
```
`rvm` 是 `Ruby` 的版本管理工具，其作用是在系统中安装若干个不同版本的 `Ruby`，且不让它们之间发生冲突。你可以安装很多个版本的 `Ruby`，比如，刚刚安装了 `2.3.1`，随后你还可以安装 `1.9.2`

```
rvm install 1.9.2
```
当你需要使用 `1.9.2` 版本的 `Ruby` 的时候，就可以用下面命令

```
rvm use 1.9.2
```
当然你也可以随时切换回之前的版本，如果想要卸载哪个版本的 `Ruby`，你可以用下面命令
```
rvm uninstall 1.9.2
```
#### 安装 nvm 和 node
>这边要重点说一下，因为在这里遇到过一个坑，耗了不少时间，因为网上有各种安装方法。一些已经不太官方，所以在安装后调试Hexo环境总是出现各种错误。最后搞得电脑里面乱乱的，最后强迫症的作用下，找了方法彻底清理后重新安装。

###### 思路如下：######
* 用brew来安装nvm，并管理nvm的升级
* 用nvm来安装node.js，并管理node.js的升级

卸载掉老版本的 Node 和 nvm

###### 卸载Node ######
* 如果是从brew安装的, 运行brew uninstall node
* 删除~/目录下所有node和node_modules
* 删除/usr/local/lib中的所有node和node_modules
* 删除/usr/local/lib中的所有node和node_modules的文件夹
* 在/usr/local/bin中, 删除所有node的可执行文件(node和npm)

###### 命令行整理后如下：######
```
sudo rm -rf ~/.npm
sudo rm -rf ~/node_modules
sudo rm -rf ~/.node-gyp
sudo rm /usr/local/bin/node
sudo rm /usr/local/bin/npm
sudo rm /usr/local/lib/dtrace/node.d
```
扩展参考：[*如何删除node.js*](https://www.zhihu.com/question/27389115/answer/36434788)

###### 卸载.nvm ######
手动安装的nvm，删除下面这三个就可以了
```
rm -rf ~/.nvm
rm -rf ~/.npm
rm -rf ~/.bower
```
还需要删除这个文件中的一段配置 `.bash_profile` (用brew安装后还需要重新加上的哦)

```
# vim .bash_profile
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```
扩展参考：[*How to uninstall nvm? #298*](https://github.com/creationix/nvm/issues/298)

###### 确认是否清理干净 ######
重启终端后，分别输入下面几个命令应该都是找不到，才算是正确的
```
nvm
node
npm
```
###### 重新安装 ######
```
# 使用brew安装nvm
brew install nvm
# vim .bash_profile后增加下面这两行
export NVM_DIR="$HOME/.nvm"
source $(brew --prefix nvm)/nvm.sh
# 使用nvm安装node.js
nvm install node
# 校验
$ nvm --version
0.31.0
$ node -v
v5.7.1
$ npm -v
3.6.0
$ nvm list
->       v5.7.1
default -> node (-> v5.7.1)
node -> stable (-> v5.7.1) (default)
stable -> 5.7 (-> v5.7.1) (default)
iojs -> N/A (default)
```
这样就完成了 `node` 和 `nvm` 的安装

#### 安装 Hexo ####
Hexo 是基于 Node.js 的第三方模块，所以我们需要对其进行单独安装。Mac 用户打开 Terminal，输入下面代码
```
sudo npm install -g hexo
```
（注：NPM的全称是Node Package Manager，是一个Node.js包管理和分发工具，已经成为了非官方的发布Node模块（包）的标准。）

#### 创建文件夹 ####
在你想要创建的磁盘目录下创建文件夹，比如我之前在 `github` 同步的文件夹是 `jackchensky.github.io`,你也可以自己新建个(不要用中文)，然后在终端下输入下面命令
```
cd ~/Public/jackchensky.github.io  /** 到你的Public文件夹里，jackchensky.github.io目录下 **/
hexo init   /** 创建一个Hexo新框架 **/
```
这个时候你会发现 `Hexo` 文件夹下出现了一些文件和文件夹

继续输入下面代码
```
npm install /** 安装相应的 node modules （node 模块）**/
hexo g /** 生成一套静态网页，功能和 hexo generate 相似 **/
hexo server /** 在本地建立 Server ，提供访问和浏览 **/
```
输入完毕后，会提示你使用浏览器进入 http://localhost:4000/

这样你就在本地创建好了一个`Hexo`博客，如果你是新建的博客，可以看我之前的教程，看看如何写文章和生成文章等 [* 在GitHub上架设Hexo静态站*](http://localhost:4000/2016/09/12/build-hexo-on-github/)
如果你是迁移博客的继续往下看

#### 将原来的文件拷贝到新电脑中 ####
这里要注意哪些文件是必须的，哪些文件是可以删除的~
* 讨论下哪些文件是必须拷贝的：首先是之前自己修改的文件，像站点配置_config.yml，theme文件夹里面的主题，以及source里面自己写的博客文件，这些肯定要拷贝的。除此之外，还有三个文件需要有，就是scaffolds文件夹（文章的模板）、package.json（说明使用哪些包）和.gitignore（限定在提交的时候哪些文件可以忽略）。其实，这三个文件不是我们修改的，所以即使丢失了，也没有关系，我们可以建立一个新的文件夹，然后在里面执行hexo init，就会生成这三个文件，我们只需要将它们拷贝过来使用即可。总结：` _config.yml，theme/，source/，scaffolds/，package.json，.gitignore `，是需要拷贝的。
* 再讨论下哪些文件是不必拷贝的，或者说可以删除的：首先是.git文件，无论是在站点根目录下，还是主题目录下的.git文件，都可以删掉。然后是文件夹node_modules（在用npm install会重新生成），public（这个在用hexo g时会重新生成），.deploy_git文件夹（在使用hexo d时也会重新生成），db.json文件。其实上面这些文件也就是是.gitignore文件里面记载的可以忽略的内容。总结：` .git/，node_modules/，public/，.deploy_git/，db.json ` 文件需要删除。
* 在git bash中切换目录到新拷贝的文件夹里，使用 ` npm install ` 命令，进行模块安装。很明显我们这里没用` hexo init `初始化，因为有的文件我们已经拷贝生成过来了，所以不必用` hexo init `去整体初始化，如果不慎在此时用了` hexo init `，则站点的配置文件 `_config.yml` 里面内容会被清空使用默认值，所以这一步一定要慎重，不要用` hexo init `。
>插件安装后，有的需要对配置文件_config.yml进行配置，具体怎么配置，可以参考上面插件在github主页上的具体说明

最后使用` hexo g `，然后使用` hexo d `进行部署，如果都没有出错，就转移成功了！

#### 在Github上创建新仓库做备份 ####

#### 本地项目Git监管 ####
去你电脑的` Hexo `目录，比如我在` Public `中创建的 jackchensky.github.io 目录，想要让 Git 去监管这个项目目录，需要先去初始化一下
```
git init
```
现在， 就成功的创建了一个 repository（仓库），下面去查看一下它的状态
```
git status
```
提示我们现在正处在 master（主） 这个 branch（分支）上，然后有一堆还没有跟踪的文件，想让 Git 跟踪这些文件，需要把它添加到 Staging（工作） 区域，然后再去 commit（提交）一下。
```
git add .
git commit -m '第一次提交'
```
返回信息将看到我们提交的相关信息

确认一下我们的工作，可以使用 ` log `命令
```
git log
```
再查看一下状态会发现返回信息会告诉我们，木有啥可以提交的 ...

#### 在Github上创建一个新的远程仓库 ####
使用 `github` 提供的服务，我们可以把项目文件推送到 `github` 提供的远程仓库里面，这样，你就不用担心意外删除掉在本地电脑上的项目文件了，因为你在远程的仓库里面，还有一个备份。你可以先去注册一个 [*github*](https://github.com) 帐号，登录以后，去创建一个远程仓库：

* 点击页面右上角的 + 号，Repository name 这里，输入仓库的名称
* Description 是可以选填的描述信息
* Public 公开或 Private 私密，如果你不想让别人看到你的项目的代码，应该选择 Private
* 点击 Create repository

创建完成以后，` github ` 会给你提示，你可以在本地去创建一个新的仓库，也可以把本地上已有的仓库推送到创建的这个 ` github ` 的远程仓库里面。下面，我们把之前本地创建的` jackchensky.github.io `项目推送到刚刚在 ` github ` 上新建的 ` backup-blog ` 项目中去
```
git remote add origin https://github.com/jackchensky/backup-blog.git
git push -u origin master
```
上面这两行命令是，先去为项目添加一个远程的仓库，告诉 ` git ` 这个远程仓库的地址是啥，这个仓库是由 ` github ` 提供的，然后使用 ` git push ` ，去把 ` master ` 这个分支推送到这个远程的仓库里面。

可能这会提示你输入你在 github 上的用户名还有密码~

完成以后，你可以打开在 ` github ` 上的你的项目，你会发现项目里的文件已经显示出来了。在本地上，你可以继续使用 ` git ` 去提交修改，创建分支，然后想要把修改的东西推送到 ` github ` 上，可以使用命令：
```
git push
```

如果` git `提示这个 `fatal: remote origin already exists `是因为你可能之前有添加过远程` git `库，先删除掉远程库

```
git remote rm origin
```
然后用上面的方法重新添加就可以了

OK 👌 .. 这样你就不用害怕你的` Hexo `博客文件哪天突然丢失啦~

---
title: 在Docker上运行一个wordpress网站
date: 2019-09-03 02:06:39
categories: wordpress
tags:
- wordpress
- Docker
---

![在Docker上运行wordpress](http://img.jackchen.cn/virtual_reality_travel-wallpaper-1920x1080.jpg)
## 前言
> 一直对Docker很感兴趣，很想用Docker来配置一个网站来体验一下它的高效和便捷。用 Docker Compose 配置好网站需要的服务，这样不管到哪都可以一键启动网站，不需要重复配置网站环境，可以让很多对服务器运维不了解的人快速的架构一个网站。最近实在太忙，这篇文章断断续续写了一个月，在配置的过程遇到过很多问题，大部分都通过Google一一化解。现在把整个过程整理了出来，希望需要的童鞋看了可以少走弯路:) ，下面就给大家说说如何配置架构网站吧~ 👏

## 1.安装配置桌面版Docker
首先需要登录Docker官方网站去下载桌面版本，Docker有MAC版本和Win版本，我这里下载的是MAC版本。PS:下载之前你需要先去申请一个`docker hub账号`然后登录一下，之后才可以看到下载按钮。如果你要下载MAC版本的可以[`点击这里下载`](https://hub.docker.com/editions/community/docker-ce-desktop-mac)。
<!--more-->
下载完成后直接打开文件，然后在打开的界面里把`Docker`拖到`Applications`(就是你的应用程序目录)中。然后你就可以在你的启动台中找到`Docker desktop`。第一次打开会提示应用需要一些管理权限，点击OK输入你系统的密码。这样你会发现你电脑的右上角会出现一个Docker小图标，点开它会有一个下拉菜单，然后找到`Sign in/Create Docker ID`去登录一下（账号密码就是你之前在官网上申请好了的），登录成功后你能在下拉菜单中看到你的用户名。然后在下拉菜单中点开`Preferences偏好设置`，在弹出的窗口中选择`Daemon图标`来配置添加一下镜像加速地址，这样在国内下载镜像也会很快。

添加的地址可以在你的阿里云账号里找到——登录你的阿里云账号，在左上角控制台下面打开产品与服务，然后在右侧划出的界面搜索里搜索关键字`容器`，你会在下面看到`容易镜像服务`，点击后可以看到侧边菜单栏里有一个`容器加速器`，点击后`复制加速器的地址`，然后回到`Docker desktop`的`Daemon`界面，把地址粘贴在`Registry mirrors`下面，然后点击下面`Apply & Restart`来应用并重启下。之后`Docker desktop`就会应用我们做的配置。

## 2.配置需要的服务
用Docker Compose来配置运行WordPress网站需要的服务，我们需要把这些配置需求写在一个yml后缀的文件里。先在你电脑里新建一个存放网站的目录——比如说桌面上。在终端里输入如下代码：
```
cd ~/desktop     #在终端定位到桌面目录下
mkdir design-web #新建一个网站文件夹design-web
cd design-web    #进入新建的这个目录
atom ./          #如果你电脑里有转Atom编辑器，可以直接在编辑器里打开这个目录
```
然后在这个目录下创建一个`docker-compose.yml`文件，然后我们可以在这个目录下面去配置WordPress网站需要的环境。输入如下配置：

```
version: '3'
services:
  wordpress:
    image: wordpress:5.0.2
    restart: always
    depends_on:
      - db
    ports:
      - "8008:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: 'designweb'
      WORDPRESS_DB_PASSWORD: 'password'
      WORDPRESS_DB_NAME: 'designweb'
    volumes:
      - ./app/wp-content:/var/www/html/wp-content/
      - ./app/config/php-uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: 'designweb'
      MYSQL_USER: 'designweb'
      MYSQL_PASSWORD: 'password'
      MYSQL_ROOT_PASSWORD: 'password'
    volumes:
      - ./app/db:/var/lib/mysql
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    restart: always
    depends_on:
      - db
    ports:
      - "8800:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: 'password'

```
好了，这样我们就在`Docker compose`里定义了所有我们需要的服务。

## 3.运行配置的服务
首先我们要确定一下`Docker desktop`已经在你的系统中运行了，然后我们只需要打开终端，再到项目的目录里（就是`Docker-compose.yml`这个文件的目录下）执行下面命令：
```
docker-compose up -d
```
这样，你就启动了在`Docker compose`里定义的这些服务。如果你是第一次运行，Docker会先下载服务需要的镜像，比如WordPress镜像和MySQL镜像，以及PHPMYadmin镜像，然后基于镜像去创建需要的容器。下载这些镜像的时候可能需要等待一段时间，如果你之前已经有这些镜像，那下一次在执行`docker-compose up`命令的时候就不会等待了，会非常的快（因为docker只需要去创建容器，不需要再去下载镜像了）。最后你会看到运行完后会出现——
```
Creating designweb_db_1 ... done
Creating designweb_wordpress_1 ... done
Creating designweb_phpmyadmin_1 ... done
```
然后在终端中执行一下：
```
docker ps
```
来看一下系统中正在运行的容器，这里你会看见3个正在运行的容器，然后打开你电脑里的游览器，输入`localhost:8008`。因为在定义wordpress服务的时候我们设置了主机上面的8008端口映射了容器里面的80端口，所以访问的时候打开的就是`wordpress服务`（如果打不开可以稍等一下，然后再刷新一下页面就可以打开了）。如果打开的页面有问题可能是你设置的端口被占用了，可以换一个端口试试。如果还是不行可能是你的配置有问题可以[点击这里查看一下Docker官方的WordPress配置文档](https://docs.docker.com/compose/wordpress/#shutdown-and-cleanup)。

一切OK后，你可以看到游览器打开了wordpress的安装界面，然后在网页里的语言中，选择`简体中文`，然后点击`继续`。填写你网站的标题名城，后台管理员的用户名、密码和电子邮箱，然后点击`安装WordPress`。在跳出的安装成功的网页中点击`登录`按钮，输入你刚刚设置的用户名和密码，点击`登录`按钮。这样你就成功的登录了WordPress网站的管理后台。

这样你就成功在本地电脑中创建了一个WordPress网站 😄

## 4.WordPress网站的源代码管理
现在我们要给这个网站添加一个代码仓库。首先我们要看一下我们项目目录，你会看见项目文件夹中有一个`app`目录，`app`目录下面有一个`db`目录。这个db目录就是我们项目中数据库相关的文件，正常我们不会将这个目录放在源代码管理中。还有一个`wp-content`目录，这个目录放的是WordPress网站的一些资源，如果你上传过文件，你会发现`wp-content`目录下会多了一个`uploads`目录。这个目录就是我们平时上传网站上的文件存放的目录。这个文件也不需要放在代码管理中。所以我们现在要在配置里说明一下。

在你的项目目录下，比如我这里的`designweb`目录下创建一个`.gitignore`文件来写清楚不需要源代码管理的文件，比如下面这样——
```
.DS_Store
app/db/
app/wp-content/uploads/
```
然后打开终端，在项目的根目录下做git版本控制
首先初始化
```
git init
```
然后在终端中添加所有的文件
```
git add .
```
然后保存到仓库的历史记录中
```
git commit -m 'init'
```
然后你在终端中输入
```
git log
```
会看到一条我们刚刚提交的记录。

## 5.创建项目的阿里云远程仓库
远程仓库我们用的是阿里云，首先打开游览器输入https://code.aliyun.com ，登陆你的阿里云账号。然后点击`新项目`来创建一个新的私有项目。这个项目的项目路径是`code.aliyun.com+你的用户名+项目名`（比如design-web），然后下面`可见的等级`选择`Private`表式私有项目，确认后点击下面的`创建项目`。

然后复制`SSH`类型的仓库地址，再回到你电脑的终端来添加这个远程地址。你项目的目录下输入`git remote add origin+刚刚复制的SSH地址`，比如下面这样——
```
git remote add origin git@code.aliyun.com:jackchensky/design-web.git
```
在把我们项目推送到刚添加的origin阿里云的远程仓库中，输入——
```
git push origin master
```
如果你的电脑之前没有配置SSK可能会提示没有推送的权限，不急我们可以配置一下SSHK来验证下权限。执行一下——
```
cat ~/.ssh/id_rsa.pub
```
这样就输出了现在用户目录下面的这个.ssh里面的公钥内容

如果你没有这个文件，可以创建一个——
```
ssh-keygen去创建一个秘钥
```
然后复制一下上面公钥的内容，回到刚刚阿里云的远程仓库，会看到网页上方有一个提示栏，点击`增加SSH秘钥`，打开的就是一个增加公钥的界面，然后粘贴进去，然后在下面设置一个标题，比如`MAC`，再点击下面`增加秘钥`按钮。就要就OK啦！

如果增加后，上面显示`Fingerprint has already been taken`，那说明你的这个公钥在其他阿里云账号里已经使用过啦，这个时候你需要用另一个方式来验证一下。点击左侧菜单`首页`，点击我们刚刚创建的这个项目，在左侧菜单里点击`成员`，添加你之前在其他阿里云中使用过的`用户`，在`角色`选择一个角色，比如选择`Master`，然后点击`增加用户到项目`，这样就OK啦！

然后我们再回到终端，再次输入——
```
git push origin master
```
你会发现项目成功的开始推送到远程了☝️。然后点击`code.aliyun.com`左侧的`项目`再点击下面的`文件`菜单，就可以看到刚刚推送上来的master分支包含的文件啦 😊。

当然作为一个☝️优秀的程序员👩‍💻(码农)，记得要在后面添加个注释，要有这个良好的习惯，防止以后忘记了~

## 6.准备好域名
现在需要将整个网站克隆到你的服务器上，所以你要提前做好添加DNS记录让网站的域名指向服务器。如果你的域名是在阿里云购买的，还可以顺便为域名申请一个免费的SSL证书。对你网站以后的扩展有帮助哦~ 比如说你要把你的网站内容做成一个微信小程序！！

## 7.克隆项目仓库到阿里云服务器
因为我的服务器是在阿里云上，所以就以阿里云服务器来做流程说明，其他平台大家可以参考，应该大同小异。首先我用的是阿里云的ECS服务器，操作系统是CentOS7。用终端工具连接后可以先对服务器做一些小配置。

### 7.1 安装epel-release，一个软件仓库，在终端输入如下代码(如果你当前用的不是root，需要在下面的命令前面加上`sudo` 来获取管理员权限)

```
yum install epel-release -y
```
现在用yum安装的东西会多很多！

### 7.2 安装一个社区的软件仓库，在终端输入如下代码

```
yum install https://centos7.iuscommunity.org/ius-release.rpm
```

### 7.3 安装Git工具，在终端输入如下代码

```
yum install git2u -y
```
上面代码最后的`-y`是表示确认安装。

### 7.4 在安装下nginx，在终端输入如下代码

```
yum install nginx -y
```

### 7.5 从远程项目地址clone到你的服务器
进入你的服务器，到`mnt`目录下，新建一个`app`的目录，可以把网站项目放在这个目录下面。现在就可以把之前远程仓库里的项目克隆到服务器新建的`app`目录下了。打开 https://code.aliyun.com ，登录后点击左侧的项目，然后在右边找到SSH类型的仓库地址，复制一下。然后回到终端，在新建的`app`目录下输入如下代码

```
git clone + 刚刚复制的地址
```
回车执行一下，如果有询问是否连接之类的，输入 `yes`，如果显示 `Permission denied (publickey)` 可能是因为项目是私有的项目，需要有权限才可以克隆。没关系，按照下面的步骤来给远程仓库添加个服务器上的权限Key就可以了。

首先在终端 `app`目录下输入 `ls -la ~/.ssh` 来查看当前主目录下.ssh里面的文件，找到`id_rsa.pub`，终端中输入 `cat ~/.ssh/id_rsa.pub`，然后回车，复制下里面的内容，回到 code.aliyun.com 点击左边的首页>设置>SSH公钥，然后在右边点击 `增加 SSH 秘钥` ，然后将刚刚复制的内容粘贴到公钥后面的空格里，在下面标题后面输入一个标题，比如 aliyun，再点击下面的增加秘钥就完成了。

再回到终端，再执行一下之前的克隆命令

```
git clone + 远程复制过来的SSH地址
```
回车后就可以看到项目被成功的克隆到你服务器里了。

## 8.在阿里云CentOS服务器上安装Docker

### 8.1 先安装两个依赖的软件
在终端中连接到你的阿里云服务器，然后输入如下代码

```
yum install yum-utils device-mapper-persistent-data lvm2 -y
```
然后再执行一下下面的代码
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
### 8.2 安装Docker
继续在终端中输入如下代码

```
yum install docker-ce -y
```
回车后就会开始安装，安装完成后输入下面代码来启动看看
```
systemctl start docker
systemctl enable docker
```
然后可以继续查看一下docker的状态，输入下面代码
```
systemctl status docker
```
你可以在跳出的状态文字中看到 `active(running)`表示正在运行中...

### 8.3 安装 Docker Compose
在终端中连接服务器后继续输入如下代码

```
yum install docker-compose -y
```
这种安装方法是从yum仓库里安装的，版本可能会比较低一些，如果你想安装新的版本，可以去Docker Compose在github上的远程仓库，[https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)，这个页面里有最新版本的Docker Compose，复制页面里的安装方法，代码如下，粘贴到你的终端中

```
curl -L https://github.com/docker/compose/releases/download/1.25.0-rc1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
执行后再复制页面里的执行权限代码，粘贴到终端中

```
chmod +x /usr/local/bin/docker-compose
```
回车执行一下，然后就可以使用Docker Compose啦！！

## 9.运行Docker Compose项目

### 9.1 运行项目
经过上面的一些步骤，终于可以在服务器上用Docker Compose来运行项目啦 😺
首先，打开终端连接到服务器，找到项目的位置，比如我的项目位置为 `cd /mnt/app/designweb`，在项目`designweb`目录下可以看到`app`文件夹目录和`docker-compose.yml`配置文件。执行一下这个配置文件，代码如下

```
docker-compose up -d
```
这样就可以创建和运行在配置文件里定义的服务，第一次运行需要去下载服务需要的镜像，所以需要一些时间。有了镜像后再创建容器就会很快了。创建好了之后执行一下 `docker ps` 会看到正在运行的服务。

### 9.2 创建 Nginx代理
找到你服务器上Nginx配置文件所在的目录，一般目录在这个位置 `/etc/nginx/conf.d`，在这个目录下创建一个配置文件，在终端中输入如下命令

```
vi designweb.cn.conf  // 创建的文件名可以用你的域名+.conf
```
回车后进入新建的.conf文件，然后按键盘上的字母 `i` 进入编辑模式，然后输入下面的配置代码，注意域名修改成你自己的，还有如果你域名有申请SSL证书，需要把证书放对目录位置并在下面配置文件里指向到对应的目录位置。

```
upstream designweb {
  server 127.0.0.1:8000;
}

server {
  listen        80;
  server_name   www.designweb.cn designweb.cn;
  return 301    https://$host$request_uri;
}

server {
  listen       443 ssl http2;
  server_name  designweb.cn;
  ssl          on;
  index        index.html;

  ssl_certificate           /etc/nginx/ssl/designweb.cn/designweb-ca-bundle.crt;
  ssl_certificate_key       /etc/nginx/ssl/designweb.cn/designweb.key;
  ssl_session_timeout       5m;
  ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers               AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
  ssl_prefer_server_ciphers on;

  location / {
    proxy_set_header  X-Forwarded-Host $host;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_set_header  X-Real-IP  $remote_addr;
    proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header  Host $http_host;
    proxy_redirect    off;
    expires           off;
    sendfile          off;
    proxy_pass        http://designweb;
  }
}
```
编辑完成后，按一下键盘左上角的 `esc`，然后在输入 `:wq` 回车来保存退出编辑模式！然后在终端里执行一下 `nginx -t` 来测试一下配置有没有问题！如果显示 `test is successful`就没问题啦~ 现在再重启一下Nginx服务器，输入 `systemctl reload nginx`。👌现在打开你电脑的游览器，访问你绑定在服务器上的域名，然后页面会跳转到一个wordpress的安装页面，然后在安装页面输入你网站的名字、管理员的用户名、管理员密码和一个邮箱地址，输入完成后点击下面的 Install Wordpress来安装WordPress网站，安装完成后，输入你刚刚设置的用户名和密码就可以登录网站的管理后台啦！！现在你就可以在后台设置里对网站做一些修改和配置，并开始发布你的文章啦！

### 9.3 修改WordPress上传文件大小限制
WordPress后台默认上传的文件大小为2M，如果要修改这个限制，需要在配置文件里说明一下，大家可以看到上面我的docker-compose.yml的配置文件中有一行这个代码

```
- ./app/config/php-uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
```
这个就是为了修改上传文件大小的，但是上面我没有创建 `php-uploads.ini` 这个配置文件，在你项目目录下的`app`目录里新建一个 `config`目录，在这个目录里新建一个 `php-uploads.ini`配置文件，在这个文件里添加一些和文件上传相关的配置，代码如下

```
file_upload = On
upload_max_filesize = 200M
post_max_size = 200M
max_execution_time = 600
```
这样你的WordPress上传文件大小的限制就修改成了200M。

### 9.4 Docker在服务器上频繁重启
网站配置完成后很开心，但是发现网站很不稳定，查看服务器发现Docker在频繁的重启。在查阅了很多资料后还不是很确定具体是哪种可能，最后问了做技术的朋友，知道是服务器内存小了，我初始服务器配置的内存是512M，所以在阿里云上把服务器内存升级到1G后问题完美解决 👏

## 10.总结
现在你就可以用Docker来运行一个WordPress网站啦！还是很不错的 :D

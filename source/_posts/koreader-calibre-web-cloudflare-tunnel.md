---
title: KOReader 无法通过 OPDS 访问 NAS 上的 Calibre-Web？我最后用 Cloudflare Tunnel 解决了
date: 2026-09-19 16:19:06
categories: NAS
tags:
- NAS
- KOReader
- Calibre-Web
- OPDS
- Cloudflare Tunnel
---

![KOReader、电子阅读器与 Cloudflare Tunnel](/images/cloudflare-tunnel-koreader/koreader-cloudflare-tunnel-cover.jpg)

> 本文记录一次真实折腾过程：NAS 内网里的 Calibre-Web，本地访问正常，手机 5G 也能打开域名端口，但 Kindle Oasis 3 上的 KOReader OPDS 怎么都访问不稳定。最后的稳定方案是：用 Cloudflare Tunnel 给 Calibre-Web 做一个 HTTPS 公网入口。

文中涉及的域名、邮箱、内网 IP、Tunnel ID 等都已脱敏，截图也做了局部打码。

<!--more-->

## 背景

我在 NAS 上部署了 Calibre-Web，主要用来管理电子书，并通过 KOReader 自带的 OPDS 功能在 Kindle 上浏览和下载书籍。

原本的访问方式大概是：

```text
http://nas.example.com:8083/opds
```

在局域网里访问没问题，手机 5G 下也能打开。但是 Kindle Oasis 3 + KOReader 访问 OPDS 时经常失败。

排查后发现，这类问题很可能跟下面几个因素有关：

- 家宽公网环境不稳定，可能是 IPv6-only 或 NAT 环境
- Kindle / KOReader 对 IPv6、非标准端口、HTTP 明文连接的兼容性不如手机浏览器
- 直接暴露 NAS 端口到公网也不够安全
- DDNS 能解决“域名指向哪里”，但不能保证 Kindle 那边一定能顺利连通

所以最终目标变成：

```text
KOReader -> HTTPS 域名 -> Cloudflare Tunnel -> NAS 内网 Calibre-Web
```

## 最终方案

最终我使用了：

- Cloudflare 免费版
- Cloudflare Zero Trust Free
- Cloudflare Tunnel
- NAS 上运行 `cloudflared`
- 一个单独子域名，例如：

```text
https://books.example.com/opds
```

Cloudflare Tunnel 的好处是：NAS 不需要开放公网端口，NAS 主动连接 Cloudflare，外部访问 Cloudflare 的 HTTPS 域名即可。

## 方案结构

```text
Kindle KOReader
    |
    | HTTPS
    v
books.example.com
    |
    | Cloudflare Tunnel
    v
NAS 内网
    |
    v
Calibre-Web: http://192.168.x.x:8083
```

Calibre-Web 自己继续保留账号密码认证，不额外开启 Cloudflare Access。

注意：不要给这个地址加 Cloudflare Access 登录保护，因为 KOReader 处理不了网页式 SSO、验证码、OTP 之类的登录流程。

## 第一步：准备一个可以接入 Cloudflare 的域名

我这里没有直接使用正在跑其他服务的主域名，而是换了一个已经不用的域名。

如果你的域名还在承载网站、邮箱或其他服务，千万不要直接改 NS。改之前至少要检查：

- `A` / `AAAA` 记录
- `CNAME` 记录
- `MX` 邮箱记录
- `TXT` 记录，例如 SPF、DKIM、DMARC
- 是否还有 DDNS 在更新这个域名

如果域名已经不用，操作就简单很多：直接把整个域名接入 Cloudflare。

在 Cloudflare 添加域名时，套餐选择 Free 即可。

![Cloudflare 自动导入 DNS 记录后，需要检查哪些记录被代理、哪些记录保持 DNS only](/images/cloudflare-tunnel-koreader/01-review-dns-records-redacted.png)

上图这一步要重点看两件事：网站类记录可以走橙云代理，邮箱相关记录不要走橙云代理。尤其是 `mail`、`smtp`、`pop3` 这类记录，建议保持 `DNS only`。

## 第二步：接入 Cloudflare DNS

Cloudflare 会要求把域名的 Nameserver 改成它提供的两条，例如：

```text
aaa.ns.cloudflare.com
bbb.ns.cloudflare.com
```

原来的 Nameserver 可能类似：

```text
dns9.hichina.com
dns10.hichina.com
```

需要去域名注册商后台修改 DNS 服务器，而不是在“域名解析”页面里改 A 记录。

如果是阿里云，入口一般在：

```text
域名控制台 -> 域名管理 -> DNS 修改 / DNS服务器修改
```

不要去“云解析 DNS”页面找，那里只能改解析记录，不负责修改 Nameserver。

![Cloudflare 要求把域名的 Nameserver 换成它分配的两条](/images/cloudflare-tunnel-koreader/02-update-nameservers-redacted.png)

### DNSSEC 注意事项

Cloudflare 会提醒确认 DNSSEC 是否关闭。

如果原 DNS 服务商那里 DNSSEC 没开，就不用管。如果已开启，建议先关闭 DNSSEC，再改 Nameserver，否则可能出现解析异常。

## 第三步：Cloudflare Zero Trust Free

进入 Cloudflare Zero Trust：

```text
https://one.dash.cloudflare.com/
```

![第一次进入 Cloudflare Zero Trust，点击 Get started 开始配置](/images/cloudflare-tunnel-koreader/03-zero-trust-get-started.png)

首次使用会要求开通 Zero Trust Free。它是免费的，但可能会要求绑定付款方式。

开通后建议马上设置一个低额度账单提醒：

```text
Manage Account -> Billing -> Billable usage -> Billable usage notification
```

可以添加：

```text
Billing Budget Alert
Budget: 1 USD
```

这样如果真的产生可计费使用量，会有邮件提醒。

正常只用 Cloudflare Tunnel 给个人 OPDS 使用，不应该产生费用。

## 第四步：创建 Cloudflare Tunnel

在 Zero Trust 控制台进入：

```text
Networks -> Tunnels & Mesh
```

![在 Zero Trust 的 Networks 里进入 Tunnels & Mesh，创建一个 Cloudflared Tunnel](/images/cloudflare-tunnel-koreader/04-tunnels-page-redacted.png)

点击：

```text
Create a tunnel
```

类型选择：

```text
Cloudflared
```

Tunnel 名字可以写：

```text
calibre-web-nas
```

接下来选择 Docker 安装方式。Cloudflare 会给一条命令，类似：

```bash
docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <YOUR_TOKEN>
```

这里的 token 是敏感信息，不要发到公开场合，不要写进博客，不要提交到 Git。

## 第五步：在 NAS 上运行 cloudflared

理想情况下，可以直接在 NAS 上跑：

```bash
docker run -d \
  --name cloudflared-calibre-web \
  --restart unless-stopped \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate run --token <YOUR_TOKEN>
```

但我实际遇到的第一个坑就是：NAS 拉 Docker Hub 镜像超时。

报错类似：

```text
Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection
```

### 解决 Docker Hub 拉不动的问题

我最后没有继续死磕 Docker Hub，而是改用 Cloudflare 官方发布的 `cloudflared` 二进制文件。

NAS 是 x86_64，所以使用 Linux amd64 版本：

```text
cloudflared-linux-amd64
```

官方下载地址：

```text
https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
```

如果下载慢，可以用断点续传：

```bash
curl -L -C - \
  -o cloudflared \
  https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
```

下载完成后，给执行权限：

```bash
chmod 755 cloudflared
./cloudflared --version
```

我为了让它仍然由 Docker 管理，最后把这个二进制封成了一个 NAS 本地 Docker 镜像，再用 Docker 运行。这样可以继续使用：

```text
--restart unless-stopped
```

也就是 NAS 重启后自动恢复。

容器运行成功后，在 Cloudflare Tunnel 页面能看到 Connector 状态变成：

```text
Connected / Healthy
```

![Tunnel 状态变成 Healthy，Connector 显示 Connected，说明 NAS 已经连上 Cloudflare](/images/cloudflare-tunnel-koreader/05-tunnel-healthy-redacted.png)

容器日志里也会看到类似：

```text
Registered tunnel connection
```

这就说明 NAS 已经成功连上 Cloudflare。

## 第六步：配置 Public Hostname

这是另一个容易点错的地方。

不要配置：

```text
Hostname routes
```

这个是给 Cloudflare Gateway / 私有网络路由用的，不适合 KOReader 访问公网 OPDS。

![这个 Hostname routes 页面不是我们要用的，看到 Gateway 提示就说明点错了](/images/cloudflare-tunnel-koreader/06-wrong-hostname-routes-page.png)

应该配置：

```text
Published application routes
```

或者界面里叫：

```text
Public Hostname
```

新增一条公开主机名：

```text
Subdomain: books
Domain: example.com
Path: 留空
Type: HTTP
URL: 192.168.x.x:8083
```

最终访问地址就是：

```text
https://books.example.com/opds
```

这里 `192.168.x.x:8083` 是 NAS 内网里 Calibre-Web 的地址。因为 `cloudflared` 跑在 NAS 或同一局域网内，所以它可以访问这个内网地址。

## 第七步：测试

浏览器里先测试：

```text
https://books.example.com
```

如果能看到 Calibre-Web 页面，说明基本通了。

再测试 OPDS：

```text
https://books.example.com/opds
```

如果 Calibre-Web 开了登录认证，浏览器或 OPDS 客户端会要求输入 Calibre-Web 的用户名密码。

最后在 KOReader 里添加 OPDS：

```text
名称：Calibre-Web
地址：https://books.example.com/opds
用户名：Calibre-Web 用户名
密码：Calibre-Web 密码
```

保存后测试浏览目录和下载书籍。

## 我踩到的坑

### 1. 不要直接动正在使用的主域名

如果域名下面还有网站、邮箱、DDNS 或其他服务，直接改 Nameserver 可能影响现有业务。

最好用一个闲置域名，或者完整备份并迁移所有 DNS 记录。

### 2. 邮箱相关记录不要开橙云代理

如果 Cloudflare 自动导入了 `mail`、`smtp`、`pop3` 等记录，有时会默认开橙云代理。

邮件相关记录应该保持：

```text
DNS only
```

不要 Proxied。

MX 记录本身通常只能是 DNS only。

### 3. Cloudflare Access 不要开

Cloudflare Access 很适合保护网页后台，但不适合 KOReader 的 OPDS。

KOReader 无法处理交互式网页登录、SSO、邮箱验证码等流程。

这个场景下，认证交给 Calibre-Web 自己处理即可。

### 4. Hostname routes 不是 Public Hostname

Cloudflare 新界面里有：

```text
Hostname routes
Published application routes
```

OPDS 这种公网访问应用，应该用 Published application routes / Public Hostname。

Hostname routes 会提示需要 Cloudflare Gateway，不是我们要的。

### 5. Docker Hub 拉不动很正常

NAS 环境里拉 Docker Hub 经常超时。

可以考虑：

- 换镜像源
- 本机下载镜像后传到 NAS
- 下载官方二进制再运行
- 封成本地 Docker 镜像运行

这次我用的是官方二进制 + 本地 Docker 镜像。

### 6. Token 不要泄露

Cloudflare 给的 Docker 命令里包含 Tunnel token。

这个 token 等同于授权凭据，千万不要：

- 发到群里
- 写进博客
- 提交到 Git
- 截图完整公开

## 回滚方式

如果以后不用了，可以这样回滚：

1. 在 Cloudflare 删除 Public Hostname
2. 停掉 NAS 上的 cloudflared 容器

```bash
docker stop cloudflared-calibre-web
docker rm cloudflared-calibre-web
```

3. 如果整个域名不再使用 Cloudflare，可以把 Nameserver 改回原 DNS 服务商
4. 如果只是不用 OPDS Tunnel，域名仍然可以继续留在 Cloudflare

## 总结

这套方案的核心价值是：

- 不需要在路由器上开放 NAS 端口
- 不依赖家宽公网 IPv4
- 对 Kindle / KOReader 更友好，因为最终是标准 HTTPS 地址
- Calibre-Web 仍然保留自己的账号密码
- Cloudflare Tunnel 免费方案足够个人使用

最终 KOReader 里只需要填：

```text
https://books.example.com/opds
```

相比 DDNS + 端口转发，这个方案更稳，也更适合不想把 NAS 直接暴露到公网的人。

---

头图摄影：[Linh Quach](https://unsplash.com/@linhquach)，来源：[Unsplash](https://unsplash.com/photos/e-reader-with-glass-of-water-and-coaster-gaTwia11ewI)。

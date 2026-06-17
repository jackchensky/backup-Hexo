---
title: 把 Codex 额度显示到 Touch Bar 上的折腾记录
date: 2026-06-17 16:20:00
categories: MAC
tags:
- MAC
- Codex
- Touch Bar
- Swift
---

![TouchBarCodexToken 一处刷新，四处同步](/images/touchbar-codex-token/touchbar-codex-token-overview.jpg)

## 前言

> 最近使用 Codex 的频率明显变高之后，我遇到一个很现实的问题：不是不知道额度会消耗，而是不知道它什么时候快消耗完。每次都要切回 Codex 里看，或者等到提示出来才发现额度紧张，这个体验就有点被动。刚好手上还有一台带 Touch Bar 的 MacBook Pro，于是就想着能不能做一个小工具，把 Codex 的 5 小时额度和周额度直接显示在 Touch Bar、菜单栏和桌面小浮窗里。
>
> 这个小工具最后叫 `TouchBarCodexToken`。它不是为了做一个很复杂的软件，更多是一次把自己日常痛点产品化的小实验：让额度一直可见，让心里有数。

<!--more-->

## 1.先确定读取额度的方式

最开始我以为这个事情可能要抓网页，或者从某个配置文件里找状态。但这样很不稳定，也不太优雅。后来发现 Codex 本机应用里自带了一个 `app-server`，可以通过标准输入输出走 JSON-RPC。

应用启动时实际调用的是：

```bash
/Applications/Codex.app/Contents/Resources/codex app-server --listen stdio://
```

然后先做一次 `initialize`，再调用：

```plain
account/rateLimits/read
```

这样就可以拿到账号额度信息。这个方案有几个好处：

> 1. 不需要填写 API Key。
> 2. 不保存账号、密码、授权码。
> 3. 不抓网页，不依赖页面结构。
> 4. 数据来自本机 Codex app-server，逻辑上更稳定。

Swift 里这部分主要由 `CodexAppServerClient` 负责。它用 `Process` 启动 Codex app-server，然后用 `Pipe` 写入一行 JSON 请求，再从标准输出里一行一行读 JSON 响应。

核心思路大概就是：

```swift
process.executableURL = codexURL
process.arguments = ["app-server", "--listen", "stdio://"]
process.standardInput = inputPipe
process.standardOutput = outputPipe
```

请求时给每个 JSON-RPC 请求分配一个 id，返回时再根据 id 找回对应的回调。这里比较像自己手写一个很轻量的客户端。

## 2.把额度整理成能显示的数据

拿到原始额度之后，不能直接塞到界面里。因为界面真正需要的是：

> 1. 5 小时额度剩余多少。
> 2. 周额度剩余多少。
> 3. 什么时候重置。
> 4. 读取失败时应该显示什么。

项目里用 `RateLimitStore` 做中间层。它会把 Codex 返回的窗口数据整理成两个 `LimitMeter`：一个是 `5 小时`，一个是 `周限额`。

判断方式也比较直接：

```swift
return abs(duration - 300) < 30
```

5 小时就是大约 300 分钟的窗口；周额度则按 7 天左右的窗口来识别。这样即使返回数据里主次顺序有变化，也能尽量归类到正确的显示位置。

这里还有一个小细节：刷新失败时不能直接把旧数据清空。因为网络或者 app-server 短暂异常时，如果界面突然变成空白，反而会让人误判。现在的逻辑是保留旧数据，只在状态里记录错误信息。

## 3.一份状态，同步到三个地方

这个工具的界面主要有三层：

> 1. 菜单栏状态。
> 2. 桌面置顶 HUD。
> 3. Touch Bar 额度条。

菜单栏适合一直挂着，HUD 适合扫一眼，Touch Bar 则是最有仪式感的地方。

HUD 默认是一个小胶囊浮窗，类似这样：

```plain
● 5h 85%   ● 7d 80%   ↻   ×
```

`↻` 用来手动刷新，`×` 用来退出。状态点按剩余额度显示不同颜色。菜单栏里还可以设置浮窗颜色和透明度，这些设置保存在 `UserDefaults`，下次启动继续生效。

AppKit 里菜单栏入口由 `NSStatusBar.system.statusItem` 创建：

```swift
private let statusItem = NSStatusBar.system.statusItem(withLength: 118)
```

而 HUD 使用一个自定义的 `NSPanel`，保持在桌面上方但不抢主应用焦点。这样写文章或者用 Codex 的时候，它不会像普通窗口一样挡住工作流。

## 4.Touch Bar 是最想做、也最受限制的地方

![Touch Bar 额度显示细节](/images/touchbar-codex-token/touchbar-codex-token-detail.jpg)

Touch Bar 这块是整个项目最有意思的部分。最终显示的内容包括：

> 1. Codex 图标。
> 2. 5 小时额度分段电量条。
> 3. 周额度分段电量条。
> 4. 剩余百分比。
> 5. 重置时间。
> 6. 本地 token 使用统计。

我没有用简单的进度条，而是做了一个分段电量条 `SegmentedBatteryBar`。这样在 Touch Bar 这种又窄又长的区域里，可读性会比普通 progress bar 好一些。

Touch Bar 的布局空间很紧，后面连续调整了好几次间距，比如图标大小、重置时间宽度、两行之间的距离。提交记录里能看到一串类似这样的改动：

```plain
Tighten Touch Bar remaining spacing
Tighten Touch Bar reset spacing
Tighten Touch Bar row spacing
Enlarge Touch Bar icon
```

这些看起来都是小改动，但 Touch Bar 这种地方就是这样：多 4 个像素嫌挤，少 4 个像素又显得散。

不过这里也有一个系统限制：macOS 公开的 Touch Bar API 和当前前台 App、first responder 绑定得比较紧。也就是说，当你切回 Codex 输入时，Touch Bar 可能会被 Codex 自己接管。现在的处理方式是点击 HUD 主体区域时，应用会尝试激活自己的 Touch Bar 内容，但不能强行长期霸占 Touch Bar。

这个限制绕不过去，只能接受它。

## 5.顺手把本地 token 使用量也放进去

做到后面又冒出一个想法：既然都在显示额度了，那能不能顺便显示昨天和累计 token 用量？

Codex 本机会在 `~/.codex/sessions` 下保存 session 的 jsonl 文件，里面有 `token_count` 事件。项目里用 `LocalTokenUsageReader` 去读这些文件：

```swift
private static let sessionDirectory = FileManager.default.homeDirectoryForCurrentUser
    .appendingPathComponent(".codex/sessions", isDirectory: true)
```

读取逻辑大概是：

> 1. 遍历所有 `.jsonl` 文件。
> 2. 找到包含 `token_count` 的行。
> 3. 解析 `last_token_usage` 和 `total_token_usage`。
> 4. 统计昨日用量和累计用量。

一开始这个统计如果放在主线程，刷新时 HUD 和菜单栏会有短暂卡顿。后来改成后台队列读取：

```swift
private let tokenUsageQueue = DispatchQueue(
    label: "TouchBarCodexToken.LocalTokenUsageReader",
    qos: .utility
)
```

这类小工具最怕的不是功能少，而是为了显示一个数字把系统体验拖慢。后台读取之后，主界面就顺滑多了。

## 6.让它跟着 Codex 自动启动

如果每次打开 Codex 还要手动打开额度条，那这个工具就不够“日常”。所以后面加了一个 LaunchAgent。

首次运行 app 时，会在用户目录下写入：

```plain
~/Library/LaunchAgents/com.jackchen.TouchBarCodexToken.CodexLauncher.plist
```

它每 5 秒检查一次 Codex 是否正在运行。如果 Codex 已经启动，而额度条没有运行，就自动打开 `TouchBarCodexToken.app`。

这里还有一个小细节：如果我在 Codex 还开着的时候手动退出额度条，启动器不应该立刻又把它拉起来。否则就变成“你越关它越开”的尴尬状态。所以项目里加了一个手动退出锁：

```plain
~/Library/Application Support/TouchBarCodexToken/manual-quit.lock
```

本轮 Codex 会话里手动退出后就不再自动拉起；等 Codex 完全退出，再清掉这个状态。

## 7.构建和打包

项目本身是一个 Swift Package：

```swift
// swift-tools-version: 5.8

let package = Package(
    name: "TouchBarCodexToken",
    platforms: [
        .macOS(.v11)
    ]
)
```

开发期可以直接跑：

```bash
swift run
```

正式构建走：

```bash
scripts/build-app.sh
```

这个脚本会先尝试：

```bash
swift build -c release
```

如果 SwiftPM 的 SDK 探测失败，就 fallback 到 `swiftc -sdk` 直接编译。最后组装出：

```plain
build/TouchBarCodexToken.app
```

为了方便分享，还加了 DMG 打包脚本：

```bash
scripts/package-dmg.sh
```

打包出来类似：

```plain
dist/TouchBarCodexToken-0.1.4.dmg
```

目前项目没有 Apple Developer 签名和公证，所以第一次打开时 macOS 可能会提示无法验证开发者。自己用问题不大，分享给别人时就需要说明一下右键打开。

## 8.这次迭代里踩过的几个点

### 8.1 不要用网页状态做数据源

一开始最容易想到的是抓网页或者找某个缓存文件。但只要页面结构一改，工具就废了。最后选择本机 app-server，虽然要自己处理 JSON-RPC，但稳定性明显更好。

### 8.2 Touch Bar 的可控性有限

Touch Bar 不是一个可以随便常驻的副屏。它和当前前台 App 绑定很紧，这点在设计时必须接受。最后的策略是：能显示就显示，不能强行抢。

### 8.3 小 UI 的像素很值钱

HUD 和 Touch Bar 都是小空间 UI。这里不是把信息堆上去就行，而是要不停调间距、字号、图标大小、颜色对比。否则看起来会很“工程味”，但用起来不舒服。

### 8.4 自动启动要尊重手动退出

自动启动是为了省心，不是为了和用户对着干。加上 `manual-quit.lock` 之后，逻辑就舒服很多。

## 9.当前版本做到了什么

现在 `TouchBarCodexToken` 已经可以做到：

> 1. 读取 Codex 5 小时额度和周额度。
> 2. 在菜单栏显示额度剩余百分比。
> 3. 在桌面 HUD 里显示额度、刷新和退出。
> 4. 点击 HUD 后尝试显示 Touch Bar 额度条。
> 5. 显示本地昨日和累计 token 用量。
> 6. Codex 启动时自动打开，Codex 退出后自动退出。
> 7. 支持构建 app 和打包 DMG。

对我来说，这个小工具最核心的价值不是“它用了什么技术”，而是它把一个很小但高频的焦虑点变成了一个一直可见的状态。用 Codex 的时候，抬眼就知道额度大概还剩多少，心里就稳很多。

## 10.后续可以继续做的事

后面如果继续完善，我可能会优先考虑这几件事：

> 1. 做 Universal Binary，让 Intel 和 Apple Silicon 都原生运行。
> 2. 增加签名和公证，减少首次打开时的安全提示。
> 3. 增加更细的历史趋势，比如最近 7 天 token 使用量。
> 4. 给没有 Touch Bar 的机器做一个更完整的桌面 HUD 模式。

总之，这次算是把一个“我自己想要的小工具”做成了可以日常使用的状态。它不大，但刚好解决了一个真实问题，这种项目做起来还是挺有成就感的。

项目已经放到 GitHub 上了，感兴趣的同学可以去看看源码，也欢迎一起帮忙优化：

```plain
https://github.com/jackchensky/TouchBarCodexToken
```

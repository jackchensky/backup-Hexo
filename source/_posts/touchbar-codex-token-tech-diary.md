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

## 彩蛋：这次小工具背后的提示词流程

最后放一个彩蛋。下面不是逐字聊天记录，而是我把这次做 `TouchBarCodexToken` 的过程整理成的一组可复用提示词。如果你也想用 AI 辅助做一个自己的小工具，可以参考这个节奏：先把需求说清楚，再一轮一轮让它落地、验证、修补。

### 1.先描述真实痛点

```plain
我经常使用 Codex，但不知道 5 小时额度和周额度还剩多少。
我想做一个 macOS 小工具，把 Codex 额度显示在菜单栏、桌面 HUD 和 Touch Bar 上。
要求尽量本地化，不要让我填写 API Key，也不要保存账号密码。
你先帮我分析一下可行方案，看看数据应该从哪里读取。
```

这一轮的重点不是马上写代码，而是先确认方向。后来确定的路线是：不抓网页，不读敏感配置，而是调用本机 Codex app-server。

### 2.确认技术路线

```plain
我们用 Swift / AppKit 写一个 macOS 小应用。
请帮我设计项目结构：
1. 一个客户端负责启动 Codex app-server，并通过 stdio JSON-RPC 读取额度。
2. 一个状态层负责把原始额度整理成 5 小时额度和周额度。
3. UI 同时显示在菜单栏、桌面 HUD、Touch Bar。
4. 失败时保留旧数据，不要把界面清空。
先给出模块划分和关键类名。
```

这一步很重要。小工具看起来简单，但如果一开始没有分层，后面很容易变成所有逻辑都塞在 `AppDelegate` 里。

### 3.实现 Codex app-server 客户端

```plain
请实现一个 CodexAppServerClient。
它需要启动：
/Applications/Codex.app/Contents/Resources/codex app-server --listen stdio://

然后通过 JSON-RPC 调用：
initialize
account/rateLimits/read

要求：
1. 每个请求有自增 id。
2. stdout 按行读取 JSON。
3. 根据 id 匹配响应回调。
4. 支持 account/rateLimits/updated 通知。
5. 不记录任何敏感信息。
```

这一轮解决的是数据入口。只要数据入口稳定，后面的 UI 就只是怎么呈现的问题。

### 4.把原始额度变成可显示状态

```plain
请设计 RateLimitStore 和显示模型。
Codex 返回的额度里可能有 primary 和 secondary 两个窗口。
请根据 windowDurationMins 判断：
1. 约 300 分钟的是 5 小时额度。
2. 约 7 天的是周额度。

输出给 UI 的状态需要包含：
剩余百分比、剩余文字、重置时间、刷新中状态、错误信息。
刷新失败时保留旧额度，只更新错误状态。
```

这个提示词让 AI 不只是“展示 JSON”，而是把数据整理成界面真正需要的状态。

### 5.做菜单栏和桌面 HUD

```plain
请用 AppKit 做一个菜单栏 + 桌面 HUD。
菜单栏显示：
5h xx% 和 W xx%

桌面 HUD 是一个置顶小胶囊，显示：
● 5h xx%   ● 7d xx%   ↻   ×

要求：
1. 点击 ↻ 刷新额度。
2. 点击 × 退出应用。
3. 状态点根据额度高低显示不同颜色。
4. HUD 不要抢主应用焦点。
5. 菜单里可以设置 HUD 颜色和透明度，并保存到 UserDefaults。
```

这一轮主要是把工具变成“能日常挂着用”的形态，而不是一个只会弹窗的 demo。

### 6.做 Touch Bar 两行额度条

```plain
请实现 Touch Bar 显示。
内容包括：
1. Codex 图标。
2. 5 小时额度分段电量条。
3. 周额度分段电量条。
4. 剩余百分比。
5. 重置时间。

Touch Bar 高度很小，请尽量使用紧凑布局：
两行显示，字号小一点，数字使用 monospaced digit。
进度条不要用普通长条，做成分段电量条。
```

Touch Bar 这里后面反复调了很多次。提示词越具体，AI 越容易帮你朝正确方向收敛。

### 7.补上本地 token 使用统计

```plain
Codex 本地 session 在 ~/.codex/sessions 下面有 jsonl 文件。
里面有 token_count 事件。
请增加 LocalTokenUsageReader：
1. 遍历 jsonl。
2. 读取 payload.type == token_count 的事件。
3. 统计昨日 last_token_usage.total_tokens。
4. 统计累计 total_token_usage.total_tokens。
5. 放到后台队列读取，不要阻塞 HUD 和菜单栏刷新。
```

这个功能是后面加的。它的关键点不是“能不能读出来”，而是不能为了读统计把界面卡住。

### 8.让它跟随 Codex 自动启动和退出

```plain
请实现一个 CodexLifecycleMonitor：
1. 监听 Codex 是否启动。
2. Codex 启动时显示 HUD 并开始刷新。
3. Codex 退出时关闭本工具。

再实现一个 LaunchAgent 自动启动器：
1. 首次运行后写入 ~/Library/LaunchAgents。
2. 每 5 秒检查一次 Codex 是否运行。
3. 如果 Codex 在运行但本工具没运行，则自动打开本工具。
4. 如果用户在本轮 Codex 会话中手动退出，不要马上再次拉起。
```

这一轮让工具从“手动打开的软件”变成“跟随工作流自动出现的小助手”。

### 9.打包成可分享的 app 和 DMG

```plain
请写两个脚本：

scripts/build-app.sh
用于：
1. swift build -c release。
2. 如果 SwiftPM SDK 探测失败，就 fallback 到 swiftc -sdk。
3. 组装 build/TouchBarCodexToken.app。
4. 复制 Info.plist、AppIcon.icns、启动器脚本。
5. 尝试 ad-hoc codesign。

scripts/package-dmg.sh
用于：
1. 调用 build-app.sh。
2. 创建 dist/dmg-stage。
3. 放入 app 和 Applications 软链接。
4. 用 hdiutil 生成压缩 DMG。
```

做到这里，小工具就不只是本机源码项目了，而是可以打包分享给别人试用。

### 10.最后让 AI 做一次代码体检

```plain
请从代码审查角度检查这个项目：
1. 有没有会卡主线程的地方。
2. app-server 进程和 Pipe 是否有正确清理。
3. 失败状态是否会误清空旧数据。
4. LaunchAgent 是否可能反复拉起。
5. 有没有保存密码、API Key、授权码等敏感信息。
6. README 是否把兼容性、安装方式和隐私说明讲清楚。
```

我现在越来越喜欢在最后加一轮“代码体检”。很多时候功能已经跑通了，但真正影响日常使用体验的，反而是这些边边角角。

这一整套提示词背后的思路其实很简单：不要一句话让 AI “帮我做个 App”，而是把 App 拆成可以验证的小步骤。每一步都让它产出能运行、能检查、能继续迭代的东西，这样最后更容易得到一个真的能用的小工具。

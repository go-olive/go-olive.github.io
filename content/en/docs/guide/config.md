---
title: "配置文件"
description: ""
lead: ""
date: 2022-10-26T01:50:00+08:00
lastmod: 2022-10-26T01:50:00+08:00
draft: false
images: []
menu:
  docs:
    parent: "guide"
    identifier: "config-c8b2d16a209b228a5704932e195f43c6"
weight: 40
toc: true
---

## 详解

### 最小配置项

只需要填写这几个配置项，就可以开始对一个直播间进行录制了。

```toml
[[Shows]]
# 全局唯一字符串，因为配置修改会实时监控实时更新，故以此 ID 作为标识这一个录制的配置项
ID = 'a'
# 平台名称
Platform = "bilibili"
# 房间号
RoomID = "21852"
# 主播名称
StreamerName = "old-tomato"
```

### 定制文件名称

新增配置项 `OutTmpl`

- 日期: `{{ now | date \"2006-01-02 15-04-05\"}}`

- 主播名称: `{{ .StreamerName }}`

- 直播标题: `{{ .RoomName }}`

- 直播平台: `{{ .SiteName }}`

```toml
[[Shows]]
ID = 'a'
Platform = "bilibili"
RoomID = "21852"
StreamerName = "old-tomato"
# 文件名称将会是 `[2022-04-24 02-02-32][old-tomato][Hi!]`
OutTmpl = "[{{ now | date \"2006-01-02 15-04-05\"}}][{{ .StreamerName }}][{{ .RoomName }}]"
```

### 定制文件保存位置

新增配置项 `SaveDir`

> 文件名称中的`tmpl变量`，如主播名称、直播平台等也可以在这里使用

```toml
[[Shows]]
Platform = "bilibili"
RoomID = "21852"
StreamerName = "old-tomato"
SaveDir = "/Users/luxcgo/Videos"
```

### 定制文件下载器

新增配置项 `Parser`

```toml
[[Shows]]
Platform = "bilibili"
RoomID = "21852"
StreamerName = "old-tomato"
# 使用 `ffmpeg` 作为文件下载器
Parser = "ffmpeg"
```

参考表

| 下载器     | 类型   | 平台                |
| ---------- | ------ | ------------------- |
| streamlink | 第三方 | YouTube/Twitch      |
| yt-dlp     | 第三方 | YouTube             |
| ffmpeg     | 第三方 | 除了 YouTube/Twitch |
| flv        | 原生   | 除了 YouTube/Twitch |

> 如果您需要使用这些第三方下载器，请务必手动将它们下载到您的本地环境中。
>
> 原生下载器已内置到 **olive** ，您无需下载就可畅快使用。

### 录制结束后执行定制化命令

增加 `Shows.PostCmds` 配置项，在任何一个 `Shows` 的下面都可以自定义的配置多个命令。

在录制结束的时候会自动依次执行，若中途有命令执行失败，则提前退出。

**olive** 内部提供了几个已经实现好开箱即用的命令（ 在`[[Shows]]` 的`PostCmds`中增加 `Path` 配置项 ）

- `olivearchive`: 将文件移动到当前路径下的 `archive` 文件夹中。
- `olivetrash`: 将文件删除（不可恢复）。
- `olivebiliup`: 若有配置 `CookieFilepath`、`BiliupEnable`，则会根据配置自动上传至哔哩哔哩，若上传失败会执行`olivearchive`。
- `oliveshell`: 将常规终端指令切分成字符串数组，并配置到 `Args` 中。
  - **olive** 内置了文件路径作为环境变量，并可以通过 `$FILE_PATH` 获取。注意环境变量只有在 shell 环境中才会正确解析，如 `/bin/zsh -c "echo $FILE_PATH"` ，只执行 `echo $FILE_PATH` 则很可能获取不到路径信息。

配置文件样例

```toml
[Config]
# 开启 biliup
BiliupEnable = true
# cookies 文件路径
# cookies 文件生成可在命令行执行 `olive biliup login` 命令
CookieFilepath = '/Users/lucas/github/olive/cookies.json'
Threads = 6
# 每秒最多上传字节数，以下配置是限速 2MB/s
MaxBytesPerSecond = 2097152

[[Shows]]
ID = 'a'
Enable = false
Platform = 'bilibili'
RoomID = '1319379'
StreamerName = 'test1'
Parser = 'flv'
SaveDir = ''
PostCmds = '[{"Path":"oliveshell","Args":["/bin/zsh","-c","echo $FILE_PATH"]},{"Path":"olivebiliup"},{"Path":"olivetrash"}]'
```

模拟执行配置文件

1. 录制结束
2. 执行自定义命令`/bin/sh -c "echo $FILE_PATH"`
3. 若上条命令执行成功，执行内置命令`olivebiliup`
4. 若上条命令执行成功，执行内置命令`olivetrash`

### 切分文件

当以下任意一个条件满足是时， **olive** 会新创建一个文件用于录制。

- 最大视频时长: `Duration`
  - 一个时间段字符串是一个序列，每个片段包含可选的正负号、十进制数、可选的小数部分和单位后缀，如"300ms"、"-1.5h"、"2h45m"。
  - 合法的单位有"ns"、"us" /"µs"、"ms"、"s"、"m"、"h"。
- 最大视频大小(字节): `Filesize`

```toml
[[Shows]]
ID = 'a'
Platform = 'bilibili'
RoomID = '1319379'
SplitRule = '{"FileSize":2000000000,"Duration":"1h"}'
```

## 直播平台

### 原生

| Platform |
| -------- |
| bilibili |
| douyin   |
| huya     |
| kuaishou |
| lang     |
| tiktok   |
| twitch   |
| youtube  |

**olive** 依赖 **[olivetv](https://github.com/go-olive/olive/tree/main/foundation/olivetv)** 来支持上述网站的直播录制。

如果您的不在上述列表中，欢迎在 [discussion](https://github.com/go-olive/olive/discussions/50) 中留下评论或提交 pr 。

### streamlink

更多的一些网站被 streamlink 支持，所以就没有必要重新造轮子了。streamlink 可以作为一个插件在 **olive** 中使用 。只要网站在 streamlink 中被支持，你也可以在 **olive** 中使用它。

例子如下：

```toml
[[Shows]]
Platform = "streamlink"
RoomID = "https://twitcasting.tv/c:aiueo033000"
StreamerName = "c:aiueo033000"
OutTmpl = "[{{ .StreamerName }}][{{ now | date \"2006-01-02 15-04-05\"}}]"

[[Shows]]
Platform = "streamlink"
RoomID = "https://live.nicovideo.jp/watch/lv337855989"
StreamerName = "ラムミ"
OutTmpl = "[{{ .StreamerName }}][{{ now | date \"2006-01-02 15-04-05\"}}]"

[[Shows]]
Platform = "streamlink"
RoomID = "https://play.afreecatv.com/030b1004/241795118"
StreamerName = "덩이￼"
OutTmpl = "[{{ .StreamerName }}][{{ now | date \"2006-01-02 15-04-05\"}}]"
```

## Config.toml

一个包含所有特性的配置文件。

```toml
[Config]
PortalUsername = 'olive'
PortalPassword = 'olive'
# 日志输出目录
LogDir = '/Users/luxcgo/olive'
# 全局 OutTmpl
OutTmpl = "[{{ now | date \"2006-01-02 15-04-05\"}}][{{ .RoomName }}]"
# 全局 SaveDir，绝对路径
SaveDir = "/Users/luxcgo/olive/{{ .SiteName }}/{{ .StreamerName }}/"

# 日志等级 (0~6), 越大日志输出越多
LogLevel = 5
# 直播间状态查询间隔时间（秒）
SnapRestSeconds = 15
# 文件是否满足切割条件检测间隔时间（秒）
SplitRestSeconds = 60
# 直播间录播结束后执行命令的并发执行的个数
CommanderPoolSize = 1
# 解析器工作状态检查间隔时间（秒）
ParserMonitorRestSeconds = 10

# 部分网站需要配置 cookie
DouyinCookie = "__ac_nonce=06245c89100e7ab2dd536; __ac_signature=_02B4Z6wo00f01LjBMSAAAIDBwA.aJ.c4z1C44TWAAEx696;"
KuaishouCookie = "did=web_d86297aa2f579589b8abc2594b0ea985"

# biliup 配置项
BiliupEnable = false
CookieFilepath = '/Users/lucas/github/olive/cookies.json'
Threads = 6
MaxBytesPerSecond = 2097152

[[Shows]]
# 全局唯一字符串，因为配置修改会实时监控实时更新，故以此 ID 作为标识这一个录制的配置项
ID = 'a'
# 平台名称
Platform = "huya"
# 房间号
RoomID = "518512"

[[Shows]]
ID = 'b'
Enable = false
Platform = 'bilibili'
RoomID = '1319379'
StreamerName = 'test1'
Parser = 'flv'
SaveDir = ''
PostCmds = '[{"Path":"oliveshell","Args":["/bin/zsh","-c","echo $FILE_PATH"]},{"Path":"olivebiliup"},{"Path":"olivetrash"}]'
SplitRule = '{"FileSize":2000000000,"Duration":"1h"}'
```

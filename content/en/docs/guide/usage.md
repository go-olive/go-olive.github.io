---
title: "安装使用"
description: ""
lead: ""
date: 2022-10-26T01:32:23+08:00
lastmod: 2022-10-26T01:32:23+08:00
draft: false
images: []
menu:
  docs:
    parent: "guide"
    identifier: "usage-2b18521821154aa3ba5696ad164acfbe"
weight: 20
toc: true
---

## 安装部署

您可以通过以下 3 种方式中的任意一种来安装 **olive**：

- 源码安装

  ```sh
  go install github.com/go-olive/olive@latest
  ```

- [二进制安装](https://github.com/go-olive/olive/releases)

- docker 镜像

  ```sh
  docker pull luxcgo/olive@latest
  ```

## 快速开始

只需要传入直播间网址就可以让 **olive** 开始工作。

```sh
olive run -u https://www.huya.com/518512
```

## 进阶使用

### 命令行版本

通过使用配置文件启动 **olive** , 该文件为您提供了更多的选项。

模板文件参考: [config.toml](https://github.com/go-olive/olive/blob/main/zarf/tmpl/config.toml)

```sh
olive run -f /path/to/config.toml
```

> /path/to/config.toml 指的是`config.toml`配置文件的绝对路径

### docker版本

docker-compose.yaml 例子如下：

```yaml
version: "3.8"

services:
  olive:
    image: luxcgo/olive:latest
    container_name: olive-run
    command:
      [
        "./olive",
        "run",
        "-f",
        "/config/config.toml",
      ]
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
      - ./downloads:/downloads:Z
      - ./config:/config:Z
```

## 帮助功能

如果有任何不懂得命令，都可以执行`help`命令获得帮助，🌰 如下

```sh
olive help
```

```sh
olive help run
```

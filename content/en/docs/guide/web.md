---
title: "网页端"
description: ""
lead: ""
date: 2022-10-26T01:39:54+08:00
lastmod: 2022-10-26T01:39:54+08:00
draft: false
images: []
menu:
  docs:
    parent: "guide"
    identifier: "web-758637e4ebad85c9500062799c12eac3"
weight: 30
toc: true
---

> [demo 网站](https://olive.luxcgo.com/)：只作为样例演示，请务必不要修改账号密码

除了以后端服务的方式运行，您还可以部署网页端的服务。

该服务已打包成`docker 镜像`

```sh
# 保存以下内容为本地文件，文件名`docker-compose.yaml`
# 并执行以下命令
docker-compose up -d olive-db
docker-compose up -d olive-server
docker-compose up -d olive-portal
```

```yaml
version: "3.8"

services:
  olive-db:
    image: postgres:14-alpine
    container_name: olive-db
    restart: always
    ports:
      - 127.0.0.1:5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - ./pgdata:/var/lib/postgresql/data
    networks:
      - olive-network

  olive-server:
    image: luxcgo/olive:latest
    container_name: olive-server
    depends_on:
      - olive-db
    ports:
      - 127.0.0.1:3000:3000
      - 127.0.0.1:4000:4000
    command:
      [
        "./olive",
        "server",
        "--db-host",
        "olive-db:5432",
        "-l",
        "/downloads",
        "-s",
        "/downloads"
      ]
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
      - ./downloads:/downloads
      - ./config:/config
    networks:
      - olive-network

  olive-portal:
    image: luxcgo/olive-portal:latest
    container_name: olive-portal
    depends_on:
      - olive-server
    ports:
      - "8080:8080"
    volumes:
      - ./config/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      - olive-network

networks:
  olive-network:
```

---
title: Install docker-compose on centos
date: 2025-04-28 09:29:17
tags: docker compose
description: Docker Compose是用于定义和运行多容器Docker应用程序的工具。在Compose中，可以使用YAML文件来配置应用程序的服务。 然后运行一条命令，即可从配置中创建并启动所有服务。 使用Compose可在一台主计算机上方便地协调多个容器映像。
---

## 在 Centos 上 安装 docker-compose

### 在线安装

  ```bash
  ## 下载安装docker-compose，占位符version为版本号，可从https://github.com/docker/compose/releases查阅选择
  sudo curl -L "https://github.com/docker/compose/releases/download/${version}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  ## 添加可执行权限
  sudo chmod +x /usr/local/bin/docker-compose
  ## 查看安装是否成功
  docker-compose --version
  ```

### 离线安装
  
* 准备安装包，在可连接外网的机器下载```docker-compose```安装包，然后将安装包```ssh```至目标服务器```/usr/local/bin```目录下，然后为```docker-compose```文件添加可执行权限，最后检查安装是否成功。

### 参考文献

* [docker-compose install](https://docs.docker.com/compose/install/)
* [Docker - 离线安装 docker-compose（以CentOS系统为例）](https://www.hangge.com/blog/cache/detail_2469.html)

---
title: Use docker-compose deploy apisix
date: 2025-04-24 16:57:28
tags: apisix
description: APISIX是一个动态、实时、高性能的云原生AP网关，提供了负载均衡、动态上游、灰度发布、服务熔断、身份认证、可观测性等丰富的流量管理功能。
cover: /img/post_covers/130963205_p0.jpg
---

## 使用 docker-compose 部署 apisix

### 下载源码

  ```bash
  ## 外网
  sudo yum install -y git
  git clone https://github.com/apache/apisix-docker.git
  ## 内网自行下载完 apisix-docker ，拷贝上传至服务器
  ```

### 编排容器

  ```bash
  cd apisix-docker/example
  docker-compose -p docker-apisix up -d
  ## 检查是否成功
  curl "http://127.0.0.1:9080/apisix/admin/services/" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'
  ```

### 参考文献

  * [官方文档](https://apisix.apache.org/zh/docs/apisix/getting-started)
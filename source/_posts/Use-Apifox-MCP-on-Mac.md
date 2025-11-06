---
title: Use Apifox MCP on Mac
date: 2025-11-06 10:13:52
tags: ai
description: 介绍如何在 Mac 上配置和使用 Apifox MCP Server，将 Apifox 项目接口文档接入 Cursor 等 AI IDE，实现根据接口文档自动生成代码、搜索文档内容等功能，提升开发效率。
cover: /img/post_covers/Google_AI_Studio_2025-11-06T02_13_10.206Z.png
---

## 在 Mac 上使用 Apifox MCP

### 介绍

* 可以将```Apifox```项目内的接口文档作为数据源提供给```Cursor```等支持```AI```编程的```IDE```工具，可以帮助我们根据接口文档生成或修改代码、搜索接口文档内容等。

### 安装

* 安装```Nodejs```环境，[推荐使用```nvm```进行安装](../nodejs/Use%20nvm%20to%20manage%20nodejs.md)，要求版本号 >= 18

* 配置```apifox mcp server```，建议每个项目当成一个```server```配置：

  ```bash
  {
    "mcpServers": {
      "<project-name> API 文档": {
        "command": "npx",
        "args": [
          "-y",
          "apifox-mcp-server@latest",
          "--project-id=<project-id>"
        ],
        "env": {
          "APIFOX_ACCESS_TOKEN": "<access-token>"
        }
      }
    }
  }
  ```

  ```<project-name>```为项目名称，```<project-id>```为项目ID（项目设置 -> 基本设置 -> 项目ID复制），```<access-token>```为Apifox的访问令牌（设置 -> ```API```访问令牌 -> 新建）

### 使用

* 获取项目接口信息：

  ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/ai/apifox_mcp1.png)

  ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/ai/apifox_mcp2.png)

* 根据项目接口信息更新代码：

  ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/ai/apifox_mcp3.png)

### 参考文献

* [Apifox MCP Server](https://www.modelscope.cn/mcp/servers/@apifox/apifox-mcp-server)

---
title: "Spring Cloud Alibaba Nacos"
date: 2020-08-10T10:41:54+08:00
categories: 
 - Java
tags: 
 - SpringCloud
 - Nacos
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
 - SpringCloud
 - Nacos
description: 
---
## Spring Cloud Alibaba Nacos

## 介绍

[Nacos](https://nacos.io/zh-cn/index.html) 是一款阿里出品的集服务注册发现、服务配置和管理于一身的开源软件。在国内非常流行 基本上替代了 Spring Cloud Config 和 Spring Cloud Bus

## 技术栈

`Java` ` vsersion 1.8`

`Docker` `version 19.03.5`

`MySQL` ` vsersion 5.7`

## 下载

前提：电脑上安装了 Docker

1. 下载 Nacos

   ```dockerfile
   docker pull nacos/nacos-server:latest
   ```

## 启动

1. 部署 启动 Nacos

   ```dockerfile
   docker run --env MODE=standalone --name nacos -d -p 8848:8848 nacos/nacos-server
   ```

## 访问Nacos

- 访问地址

  http://localhost:8848/nacos/index.html

- 账号密码

  账号：nacos 密码：nacos

## 配置

### 配置持久化

#### 使用MySQL 持久化配置

- 进入容器

  ```bash
  docker exec -it <容器ID> /bin/bash
  ```

- config/application.properties 

- 



- da
- 
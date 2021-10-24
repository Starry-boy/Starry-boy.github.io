---
title: "RabbitMQ 入门"
date: 2020-08-10T10:41:54+08:00
categories: 
 - 消息队列
tags: 
  - RabbitMQ
  - 消息队列
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - RabbitMQ
  - 中间件
  - 消息队列
description: 
---
# RabbitMQ 入门

## 下载RabbitMQ

> rabbitmq:management 这个版本带 web 界面

```bash
docker pull rabbitmq:management
```

## 安装RabbitMQ

- 默认配置

  > username = guest
  >
  > Password = guest

  ```bash
  docker run -d --hostname my-rabbit --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq:management
  ```

- 指定账号密码 

  > username  = user
  >
  > Password = password

  ```bash
  docker run -d --hostname my-rabbit --name rabbit -e RABBITMQ_DEFAULT_USER=ratel -e RABBITMQ_DEFAULT_PASS=ratel -p 15672:15672 -p 5672:5672 rabbitmq:management
  ```

  ## 访问Web界面

  > http://localhost:15672

  



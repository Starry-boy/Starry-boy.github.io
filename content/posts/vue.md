---
title: "从零开始搭建 vue环境"
date: 2020-08-10T10:41:54+08:00
categories: 
 - JavaScript
tags: 
  - JavaScript
  - Vue
  - NodeJS
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - JavaScript
  - Vue
  - NodeJS
description: 
---
# 从零开始搭建 vue环境

## 一、安装node环境

1. 下载地址为：https://nodejs.org/en/

2. 查看是否安装成功

   ```shell
   node -v
   ```
   ![node v.png](http://q6r376k5l.bkt.clouddn.com/node%20-v_1583463943796.png)

3. 初始化 packag.json 文件

   ```shell
   npm init -y	
   ```
    ![npm init y.png](http://q6r376k5l.bkt.clouddn.com/npm%20init%20-y_1583463964369.png)

4. 切换淘宝的镜像仓库：http://npm.taobao.org/		

   ```shell
   npm install -g cnpm --registry=https://registry.npm.taobao.org
   ```


## 二、搭建vue项目环境

1. 全局安装vue-cli

   ```shell
   npm install --global vue-cli
   ```

   ![npm install.png](http://q6r376k5l.bkt.clouddn.com/npm%20install_1583464002031.png)![npm install](/Users/ratel/Documents/npm install.png)

2. 进入项目目录，创建一个基于 webpack 模板的新项目

   ```shell
   vue init webpack 项目名
   ```

### 三、安装 element-ui 项目环境

1. 安装 element-ui 环境

   ```shell
   npm i element-ui -S
   ```

   ![install elementui.png](http://q6r376k5l.bkt.clouddn.com/install%20element-ui_1583464017766.png)![install element-ui](/Users/ratel/Documents/install element-ui.png)

2. Element-ui 快速上后 `官方文档`

   https://element.eleme.cn/#/zh-CN/component/quickstart
## 一、安装node环境

1. 下载地址为：https://nodejs.org/en/

2. 查看是否安装成功

   ```shell
   node -v
   ```
   
3. 初始化 packag.json 文件

   ```shell
   npm init -y	
   ```
   
4. 切换淘宝的镜像仓库：http://npm.taobao.org/		

   ```shell
   npm install -g cnpm --registry=https://registry.npm.taobao.org
   ```


## 二、搭建vue项目环境

1. 全局安装vue-cli

   ```shell
   npm install --global @vue/cli
   ```

2. 进入项目目录，创建一个基于 webpack 模板的新项目

   ```shell
   vue init webpack 项目名
   ```
   
   或
   
   ```
   vue ui
   ```
   
   

### 三、安装 element-ui 项目环境

1. 安装 element-ui 环境

   ```shell
   npm i element-ui -S
   ```

2. Element-ui 快速上后 `官方文档`

   https://element.eleme.cn/#/zh-CN/component/quickstart

### 四、安装依赖

```bash
# 建议不要直接使用 cnpm 安装以来，会有各种诡异的 bug。可以通过如下操作解决 npm 下载速度慢的问题
npm install --registry=https://registry.npm.taobao.org
```





External-dubbo / web

Order-dubbo / job


10.10.15.32 job跟前端

10.10.15.30 部署web

10.10.15.31 部署dubbo

查看日志权限：dev   lsmy@dev

cat localhost_access_log.2020-09-23.txt | grep '/api/reservation'





```
set user root
set host 10.10.14.135
set password "lsmy@2020"
set port 22
set timeout 10
spawn ssh -p $port $user@$host
expect "*password:*"
send "$password\r"
interact
#expect eof
```


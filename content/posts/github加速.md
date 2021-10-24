---
title: "Github加速"
subtitle: ""
date: 2021-10-24T16:01:05+08:00
lastmod: 2021-10-24T16:01:05+08:00
draft: false
toc:
  enable: true
weight: false
categories: ["default"]
tags: ["Github","奇技淫巧"]
---

# Github加速

## 1. GitHub 镜像访问

这里提供两个最常用的镜像地址：

- https://github.com.cnpmjs.org
- https://hub.fastgit.org

也就是说上面的镜像就是一个克隆版的Github，你可以访问上面的镜像网站，网站的内容跟Github是完整同步的镜像，然后在这个网站里面进行下载克隆等操作。

## 2. GitHub文件加速

利用 Cloudflare Workers 对`github release` 、`archive` 以及项目文件进行加速，部署无需服务器且自带CDN.

```
https://gh.api.99988866.xyz` `https://g.ioiox.com
```

以上网站为演示站点，如无法打开可以查看开源项目：gh-proxy-GitHub 文件加速自行部署。

## 3. Github 加速下载

只需要复制当前 GitHub 地址粘贴到输入框中就可以代理加速下载！

地址：http://toolwa.com/github/

![img](.image/640.jpeg)



## 4. 加速你的 Github

https://github.zhlh6.cn

输入 Github 仓库地址，使用生成的地址进行 git ssh 操作即可

## 5. 谷歌浏览器GitHub加速插件(推荐)

谷歌浏览器Github加速插件.crx 下载

百度网盘: https://pan.baidu.com/s/1qGiIUzqNlN1ZczTNFbPg0A,提取码：stsv

如果可以直接访问谷歌商店，可以访问GitHub 加速谷歌商店安装。

![img](.image/640-20200817104028585.jpeg)



## 6. GitHub raw 加速

GitHub raw 域名并非 github.com 而是 `raw.githubusercontent.com`，上方的 GitHub 加速如果不能加速这个域名，那么可以使用 Static CDN 提供的反代服务。

将 `raw.githubusercontent.com` 替换为 `raw.staticdn.net` 即可加速。

## 7. GitHub + Jsdelivr

jsdelivr 唯一美中不足的就是它不能获取 exe 文件以及 Release 处附加的 exe 和 dmg 文件。

也就是说如果 exe 文件是附加在 Release 处但是没有在 code 里面的话是无法获取的。所以只能当作静态文件 cdn 用途，而不能作为 Release 加速下载的用途。

## 8. 通过Gitee中转fork仓库下载

网上有很多相关的教程，这里简要的说明下操作。

1.访问gitee网站：https://gitee.com/ 并登录，在顶部选择“从GitHub/GitLab导入仓库” 如下：

![img](.image/640-20200817104028578.jpeg)

2.在导入页面中粘贴你的Github仓库地址，点击导入即可：

![img](.image/640-20200817104028607.jpeg)

3.等待导入操作完成，然后在导入的仓库中下载浏览对应的该GitHub仓库代码，你也可以点击仓库顶部的“刷新”按钮进行Github代码仓库的同步。

![img](.image/640-20200817104028696.jpeg)

## 9. 通过修改HOSTS文件进行加速

### 为什么github下载速度这么慢？

GitHub 我们都知道是世界上最大的开源及私有软件项目的托管平台，全世界每天有海量优秀的开源软件在这里产生，而 GitHub 在国内很多时候获取到的下载链接是亚马逊的服务器。

中国因为不可言说的原因，经常抽疯或龟速。想要加快 GitHub 下载速度就需要用到 GitHub 国内加速服务，对于有条件的可以使用代理加快访问速度，而没有条件的就可以用到网上热心人士维护的加速服务了。

### 如何提高github的下载速度？

手动把cdn和ip地址绑定。

### 第一步：获取github的global.ssl.fastly地址

访问：http://github.global.ssl.fastly.net.ipaddress.com/#ipinfo 获取cdn和ip域名：![img](.image/640-20200817104028601.jpeg)

得到：199.232.69.194 https://github.global.ssl.fastly.net

### 第二步：获取github.com地址

访问：https://github.com.ipaddress.com/#ipinfo 获取cdn和ip：![img](.image/640-20200817104028551.jpeg)

得到：140.82.114.4 http://github.com

### 第三步：修改host文件

windows系统：

1、修改`C:\Windows\System32\drivers\etc\hosts`文件的权限，指定可写入：

右击->hosts->属性->安全->编辑->点击Users->在Users的权限“写入”后面打勾。如下：![img](.image/640-20200817104028564.jpeg)

然后点击确定。

2、右击->hosts->打开方式->选定记事本（或者你喜欢的编辑器）->在末尾处添加以下内容：

```
199.232.69.194 https://github.global.ssl.fastly.net

140.82.114.4 https://github.com
```
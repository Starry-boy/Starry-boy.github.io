---
title: "Git 入门命令"
subtitle: ""
date: 2021-10-24T16:01:05+08:00
lastmod: 2021-10-24T16:01:05+08:00
draft: false
toc:
  enable: true
weight: false
categories: ["documentation"]
tags: ["Git"]
---



## Git 入门命令

1. 初始仓库

   `git init`

2. clone 代码

   `git clone <url>`

3. 添加文件到暂存区

   `git add <filename>`  OR  `git add .`

   > git add . 表示将所有改动或新增的文件都添加到暂存区

4. 将暂存区内容添加到仓库中

   `git commit `

5. 上传代码

   `git push`

   > git push 表示 推送代码到远程仓库，并且合并

6. 更新代码

   `git pull` OR `git pull --rebase`  OR `git fetch`

   > git pull 表示更新代码并合并,如果主分支会有合并记录
   >
   > git pull --rebase 表示更新代码并合并，但是主分支上不会有合并记录
   >
   > git fetch 表示跟新代码

7. 查看日志

   `git log`

   > 按提交时间 倒序 显示提交记录

8. 查看暂存区

   `git status`

   > 可以看到暂存区的文件，和已经提交 但是 未push的记录数量

9. 查看本地仓库文件

   `git show <CommitID>`

   > 可以看到本地仓库未push的文件

10. 比较文件不同

    `git diff`

11. 回退版本

    `git reset head`

    > 回退到最后一次提交的版本，丢弃当前修改的内容

12. 查看指定文件历史修改内容

    `git blame <file>`

    > 以列表形式查看指定文件的历史修改记录

13. 将工作区修改的内容保存到暂存区

    `git stash`

14. 将暂存区的内容恢复出来

    `git stash pop`
    
15. 忽略已在版本控制中的文件

```
git update-index --assume-unchanged
```



## Git 高级命令





## 更新

1. `git pull`

   作用：拉取线上最新代码到本地

   优点：

   缺点：

2. `git pull --rebase`

   作用：拉取线上最新代码到本地

   优点：

   缺点：

## 提交

## 合并

## 暂存

## 分支

## 别名

```git
git config --global alias.pr 'pull --rebase'
```

表示用 将 `git pull --rebase` 简化成 `git pr`

---
layout: post
title: 更新开源项目 master 分支的最新提交到本地项目
date: 2021-02-16T20:36:16+08:00
tags: [Git, Emacs, GitHub]
categories: [Develop Tools]
---
当从 GitHub fork 一个开源的项目进行本地开发时，项目的 master 分支会不断加入新的代码，如何能够方便地将master 分支的代码合并到自己的项目呢？

下面以 [purcell](https://github.com/purcell) 的 [Emacs 配置项目](https://github.com/purcell/emacs.d)为例，操作步骤如下：
### 1. Fork 开源项目仓库，并下载克隆的项目进行开发
``` bash
git clone https://github.com/Eason0210/emacs.d.git
```
### 2. 切到需要 merge 的本地 fork master
``` bash
git checkout master
```
### 3. 获取开源项目的 master
``` bash
git remote add main https://github.com/purcell/emacs.d.git
```
### 4. 获取开源 master 最新代码
``` bash
git fetch main
```
### 5. merge 开源项目 master 到本地的 fork master
``` bash
git merge main/master
``` 
### 6. push到 fork master
``` bash
git push -u origin master
```
### 7. 查看本地 fork 的分支和是否有原始 maser 分支
``` bash
git remote -v
```
### 8. 查看所有的分支
``` bash
git branch -av
```
### 9. Merge 任何分支
同理，按照上述步骤，可以合并任何两个分支，按照步骤3-4获取额外的合并分支 branch1，步骤5-6把 branch1 合并到新分支，则新分支就有了 branch1 的所有 commit 。

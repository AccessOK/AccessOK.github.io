---
title: git使用
date: 2023-12-15 10:27:28
description: Linux git使用
type: "tags"
comments: true
categories:
- Learning Tools
- Linux
- Git
tags:
- Linux
- Git
---
# 远程配置

# 本地配置

优先级：local > global > system

```shell
$ git config user.name <name>
$ git config user.email <email>
$ git config --local ：local只对仓库有效
$ git config --global ：global对登录用户所有仓库有效
$ git config --system ：system对系统的所有用户有效
```

# 操作本地仓库

```shell
$ git checkout <old-branch>
$ git checkout -b <new-branch>
```

# 合并远程分支

```shell
$ git push origin <local-branch>:<cloud-branch>
```

# 删除本地分支

```shell
$ git branch -d <local-branch>
```
# 删除远程分支

```shell
$ git push origin -d <cloud-branch>
```
拉取远程代码到本地分支
```plain
$ git pull remote local-branch
```
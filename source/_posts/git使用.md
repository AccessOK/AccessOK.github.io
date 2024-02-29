---
title: git使用
date: 2023-12-15 10:27:28
description: Linux git使用
type: "tags"
comments: true
categories:
- Linux
- Tools
- Git
tags:
- Linux
- Git
---
# 远程配置
"待补充···"

# 本地配置

优先级：local > global > system

```bash
git config user.name <name>
git config user.email <email>
# --local ：local只对仓库有效
# --global ：global对登录用户所有仓库有效
# --system ：system对系统的所有用户有效
```

# 本地仓库切换分支

```bash
git checkout <old-branch>
git checkout -b <new-branch>
```

# 合并远程分支

```bash
git push origin <local-branch>:<cloud-branch>
```

# 删除本地分支

```bash
git branch -d <local-branch>
```
# 删除远程分支

```bash
git push origin -d <cloud-branch>
```
# 拉取远程代码到本地分支

```bash
git pull origin <local-branch>
```
# 创建远程分支

```bash
# 将本地分支推送到远程分支，若远程分支不存在则会自动新建新远程分支
git push origin <local-branch>:<cloud-branch>
# 将本地空分支推送到远程分支上时，则会删除该远程分支
git push origin :<cloud-branch>
```

# 合并commit

```bash
# 本地通常会有无数次 commit ，可以合并“相同功能”的多个 commit，以保持历史的简洁。

```
---
title: 基于devstack部署openstack开发环境
date: 2024-12-15 10:27:28
description: 基于devstack部署openstack开发环境
type: "tags"
comments: true
categories:
- Openstack
- Deploy
tags:
- openstack
---

# 环境配置

系统：openEuler-2403-LTS-SP1
架构：x86_64(虚拟机)
配置：8c-16GB-200GB
网卡：

# 部署步骤

DevStack 是一系列可扩展的脚本，用于基于 git master 中所有内容的最新版本快速搭建完整的 OpenStack 环境。它可用作交互式开发环境，并作为 OpenStack 项目大部分功能测试的基础。

从干净且最小化的 Linux 系统安装开始。DevStack 尝试支持 Ubuntu 的两个最新 LTS 版本：Rocky Linux 9 和 openEuler。如果您没有偏好，Ubuntu 24.04 (Noble) 是经过最多测试的版本，并且运行速度可能最流畅。

# 问题记录

执行./stack.sh

- 报错提示
[ERROR] ./stack.sh:238 If you wish to run this script anyway run with FORCE=yes
/opt/stack/devstack/functions-common: line 331: /opt/stack/logs/error.log: No such file or directory

- 解决方案

```bash
FORCE=yes ./stack.sh
```

- 报错提示

Error on exit
World dumping... see /opt/stack/logs/worlddump-2025-07-11-061851.txt for details
sudo: iptables: command not found

- 解决方案

```bash
sudo yum install iptables-services
```

- 报错提示

/opt/stack/devstack/lib/tls: line 43: hostname: command not found

- 解决方案

```bash
yum install hostname
```

- 报错提示

× Cannot uninstall pip 23.3.1 ╰─> The package's contents are unknown: (能够下载pip高版本，但是无法卸载低版本，因为低版本是由rpm安装的)

- 解决方案
  
```bash
sudo pip install --upgrade pip
```

- 报错提示

++functions:get_extra_file:68               wget --progress=dot:giga -t 2 -c https://github.com/etcd-io/etcd/releases/download/v3.4.27/etcd-v3.4.27-linux-amd64.tar.gz -O /opt/stack/devstack/files/etcd-v3.4.27-linux-amd64.tar.gz
/opt/stack/devstack/functions: line 68: wget: command not found

- 解决方案

```bash
yum install wget
```

- 报错提示

+lib/etcd3:install_etcd3:120               tar xzvf /opt/stack/devstack/files/etcd-v3.4.27-linux-amd64.tar.gz -C /opt/stack/devstack/files
/opt/stack/devstack/lib/etcd3: line 120: tar: command not found

- 解决方案

```bash
yum install tar
```

- 报错提示

Exception: you need a C compiler to build uWSGI

- 解决方案

```bash
yum install gcc
```

- 报错提示

plugins/python/uwsgi_python.h:4:10: fatal error: Python.h: No such file or directory

- 解决方案

```bash
yum install python3-devel
```

+lib/apache:install_apache_uwsgi:112       sudo apxs -i -c mod_proxy_uwsgi.c
sudo: apxs: command not found

```bash
 yum install httpd-devel libxml2-devel libxslt-devel
 ```

 [ERROR] /opt/stack/devstack/functions-common:713 git call failed: [git clone --no-checkout https://github.com/novnc/novnc.git /opt/stack/novnc]

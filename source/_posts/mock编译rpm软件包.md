---
title: mock编译rpm软件包
date: 2023-12-15 10:27:28
description: Linux mock编译rpm软件包
type: "tags"
comments: true
categories:
- Learning Tools
- Linux
- Rpm
tags:
- Linux
- Rpm
---
# mock 环境编译

1、yum install mock

2、新建或修改 /etc/mock/example.cfg

```shell
config_opts['basedir'] = '/home/'
config_opts['root'] = 'example'
config_opts['target_arch'] = 'x86_64'
config_opts['legal_host_arches'] = ('x86_64',)
config_opts['chroot_setup_cmd'] = 'install tar gcc-c++   which xz sed make bzip2 gzip gcc coreutils unzip shadow-utils diffutils cpio bash gawk rpm-build info patch util-linux findutils grep vim automake yum wget git'

config_opts['releasever'] = '8'

config_opts['yum.conf'] = """
[main]
keepcache=1
debuglevel=2
reposdir=/dev/null
logfile=/var/log/yum.log
retries=20
obsoletes=1
gpgcheck=0
assumeyes=1
syslog_ident=mock
syslog_device=
mdpolicy=group:primary

# repos
[base]
name=base
baseurl = url
enabled=1
gpgcheck=0
module_hotfixes=true
"""

```
3、可使用root直接运行，也可创建mock用户并添加到mockbuild组中在编译
```bash
$ mock -r example --rebuild *src.rpm
```

4、使用root用户进入mock环境

```bash
$ cd /var/lib/mock/**
$ chroot .
```

# rpm-build编译

```bash
#解压src包
$ rpm -ivh -D "_topdir `pwd`"  /path/to/*.src.rpm
#根据spec下载依赖
$ yum-builddep /path/to/*.spec
# -D "_topdir `pwd`" 指定编译目录，否则会在默认目录下编译
$ rpmbuild -ba -D "_topdir `pwd`" /path/to/*.spec
```
# koji提交

1 .安装koji

2.修改koji配置文件

3.提交rpm包

# rpm module开启

在repo的配置文件中添加,即可开启module模块下载
```bash
module_hotfixes = true
```
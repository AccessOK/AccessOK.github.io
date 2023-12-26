---
title: Python安装配置
date: 2023-12-15 10:27:28
description: Linux Python环境配置
type: "tags"
comments: true
categories:
- Linux
- Python
tags:
- Linux
- Python
---
# 安装Python
1. 下载python
```bash
$ wget https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tgz
# 解压到opt
$ tar -zxvf Python-3.8.0.tgz -C /opt
```
2. 编译安装python3.8.0
```bash
#进入到python-3.8.0文件夹
$ cd /opt/Python-3.8.0 
#检查以确保系统满足程序的最低要求
$ sudo ./configure 
#执行编译脚本
$ sudo make 
#直接安装
$ sudo make install
#替换原版本安装
$ sudo make altinstall
```
3. 修改python默认版本
```bash
#查询python3.8位置
$ whereis python3.8
python3.9: /usr/local/bin/python3.8 /usr/local/lib/python3.8
#删除当前软连接
$ sudo rm /usr/bin/python
#为新版python创建软链接
$ sudo ln -s /usr/bin/python3.8 /usr/bin/python
```
4. 配置环境变量
修改用户主目录下名为.bash_profile的文件，配置PATH环境变量并使其生效。
```bash
$ cd ~
$ vim .bash_profile
# ... 此处省略上面的代码 ...
export PATH=$PATH:/usr/local/python38/bin
# ... 此处省略下面的代码 ...
```
5. 激活环境变量
```bash
$ source .bash_profile
```
# 修改Python默认版本
1. 查询当前版本
```bash
$ python --version
Python 2.7.16
```
2. 删除当前软链接
```bash
$ sudo rm /usr/bin/python
```
3. 为新版python创建软链接
```bash
$ sudo ln -s /usr/bin/python3.7 /usr/bin/python
```
4. 校验结果
```bash
$ python --version
Python 3.7.3
```
# pip源配置
pip配置信息保存路径：~/.config/pip/pip.conf
```bash
#配置清华源
$ pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
#删除配置
$ pip config unset global.index-url
```
# venv环境配置
1.创建独立的python运行环境
```bash
#创建独立目录
$ mkdir venv
$ cd venv/
#初始化python运行环境
$ python* -m venv .
#查看当前目录，发现生成lib，include,bin，pyvenv.cfg
$ ls
bin  include  lib  pyvenv.cfg
#进入bin目录，激活该venv环境
$ cd bin
$ source activate
#退出当前的proj101env环境
$ deactivate
```
# Ipython编程工具
1. 安装Ipython
```plain
$ pip install ipython
```
2. 启动IPython
```bash
$ ipython
```
3. 退出ipython
```bash
$ ctl+D
```
turtle图形绘制工具
1. 安装依赖
```bash
# For Ubuntu or other distros with Apt:
$ sudo apt-get install python3-tk
# For Fedora:
$ sudo dnf install python3-tkinter
```
2. 绘制图形
```plain
import turtle

turtle.pensize(4)
turtle.pencolor('red')

turtle.forward(100)
turtle.right(90)
turtle.forward(100)
turtle.right(90)
turtle.forward(100)
turtle.right(90)
turtle.forward(100)

turtle.mainloop()
```



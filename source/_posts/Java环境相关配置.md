---
title: java安装配置
date: 2023-12-15 10:27:28
description: Linux java环境配置
type: "tags"
comments: true
categories:
- Linux
- Java
tags:
- Linux
- Java
---

# Java-环境安装配置
Link： <https://www.oracle.com/java/technologies/downloads/>

下载tar包，解压到指定的位置

修改环境变量(Linux)

```plain
vim /etc/profile
export JAVA_HOME=/opt/java/jdk1.8.0_381
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JRE_HOME=$JAVA_HOME/jre
```
# maven配置
Link：<https://maven.apache.org/>

```plain
export MAVEN_HOME=/opt/maven/apache-maven-3.9.4
export PATH=${PATH}:${MAVEN_HOME}/bin
```
# 运行jar包
```bash
java -jar ***.jar
```

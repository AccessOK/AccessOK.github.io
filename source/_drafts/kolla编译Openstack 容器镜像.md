---
title: kolla编译Openstack容器镜像
date: 2023-12-15 10:27:28
description: kolla编译Openstack容器镜像
type: "tags"
comments: true
categories:
- Openstack
- Deploy
tags:
- openstack
---

kolla提供编译镜像的功能，kolla-ansible具有部署openstack的功能。
### 系统配置
#### 关闭防火墙
```bash
systemctl disable --now firewalld
```
#### 配置域名
```bash
#增加域名解析
echo "10.30.38.116 harbor.chinauos.com" >> /etc/hosts
```
注：
“registry.uniontech.com”和“harbor.chinauos.com” 是两个容器镜像仓库。
其中“harbor.chinauos.com”是对外仓库，“registry.uniontech.com”是研发仓库。
### 搭建编译环境
#### 安装配置docker
安装docker和openstack-kolla包
```bash
yum install moby-engine  git -y python3-devel
```
配置docker
```bash
systemctl enable --now  docker.service
```
修改docker配置文件:/etc/docker/daemon.json
```bash
{
    "insecure-registries": [
        "registry.uniontech.com", "harbor.chinauos.com"
    ]
}
```
注：配置docker拉取容器镜像的仓库，配置此域名之后要配置相应的域名。
“registry.uniontech.com”账户和密码：
“harbor.chinauos.com”账户和密码：
为docker配置不安全仓库之后重新启动docker服务。
```bash
systemctl daemon-reload 
systemctl restart docker.service 
```
#### 安装openstack-kolla
```bash
#拉起openstack-kolla源码
git clone -b victoria-source \
"http://gerrit-dev.uniontech.com/openstack/openstack-kolla"
#切换到响应的分支安装
#使用pip3安装/卸载源码
pip3 install openstack-kolla/
pip3 uninstall openstack-kolla/
```
注：

1. 安装后所有容器镜像的Dockerfile都在/usr/local/share/kolla/docker对应名称目录下。
2. 若需要修改容器镜像找到对应的目录，更改模板文件即可。
3. 更新sql文件，请将sql文件重新命名为ustack.sql.j2。
#### 登录harbor仓库
```bash
docker login harbor.chinauos.com
```
Username: ustack
Password: Ustack12#$
参考：[https://ivanzz1001.github.io/records/post/docker/2018/04/11/docker-harbor-uage](https://ivanzz1001.github.io/records/post/docker/2018/04/11/docker-harbor-uage)
harbor是镜像管理平台，登录用户之后，则可根据用户角色权限操作镜像。
### 编译镜像
```bash
kolla-build \
--base-image harbor.chinauos.com/ren-test/uniontechos-server-20-1060a-x86  \
--config-file kolla-build.conf \
-t source \
--base uniontechos \
--tag victoria \
nova
```
参数说明: 
--base-image 使用指定的基础镜像
--base 构建uniontechos镜像
--tag 构建镜像生成镜像的tag
--base-image 根据不架构修改-x86/-arm 
--config-file 指定特定的源码地址构建镜像。
--template-only 不制作镜像，仅仅生成 Dockerfile文件。
注：openstack上搭建环境时，建议采用外部网络直连的方式配置虚拟机网络，采用xlan网络配置浮动ip时，在构建容器镜像时会导致安装依赖失败，拉取不到容器里的依赖，亲测有效。
注：部分参数可以使用kolla-build --help查询，image和tag等变量信息可以登录harbor进行查看。例如当前镜像在harbor.chinauos.com域名下的ren-test项目下的uniontechos-server-20-1060a-x86镜像。点击镜像即可查询tag名称。
kolla-build.conf 格式参考如下：
```bash
[nova-base]
type = git
location = http://gerrit-dev.uniontech.com/openstack/openstack-nova
reference = victoria-source

[cinder-base]
type = git
location = http://gerrit-dev.uniontech.com/openstack/openstack-cinder
reference = victoria-source
```
容器镜像仓库配置。可根据自身想要的安装包，配置源地址，此处的源地址是指构建容器镜像时安装依赖的rpm包的仓库地址。
构建镜像 x86的源为:/usr/local/share/kolla/docker/base/UniontechOS.repo
```bash
[UniontechOS-$releasever-AppStream]
name = UniontechOS $releasever AppStream
#baseurl = https://cdimage.uniontech.com/server-dev/1060/a/release/compose/AppStream/x86_64/os/
baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/AppStream/x86_64/os/
enabled = 1
gpgcheck = 0
module_hotfixes=true

[UniontechOS-$releasever-BaseOS]
name = UniontechOS $releasever BaseOS
#baseurl = http://10.30.38.131/kojifiles/repos/kongzi-build/latest/x86_64/
baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/BaseOS/x86_64/os/
#baseurl = https://cdimage.uniontech.com/server-dev/1060/a/release/compose/BaseOS/x86_64/os/
enabled = 1
gpgcheck = 0
module_hotfixes=true

[UnionTechOS-$releasever-openstack]
name = UnionTechOS $releasever openstack
baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/OpenStack-Victoria/x86_64/os/
#baseurl = http://10.30.38.131/kojifiles/repos/kongzi-openstack-victoria-build/latest/x86_64/
enabled = 1
gpgcheck = 0

[ceph]
name=ceph
baseurl=https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/Storage/x86_64/
gpgcheck=0
enabled=0
module_hotfixes=true

[Tools]
name = Tools
baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/PowerTools/x86_64/os/
enabled = 0
gpgcheck = 0

[PLUS]
name = plus
baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/Plus/x86_64/os/
enabled = 0
gpgcheck = 0
```
构建镜像 arm的源为:/usr/local/share/kolla/docker/base/Ustack_aarch64.repo
```bash
[UniontechOS-$releasever-AppStream]
name = UniontechOS $releasever AppStream
#https://cdimage.uniontech.com/server-dev/1060/e/release-0606/arm64/OS/
baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/AppStream/aarch64/os/
#baseurl = http://pools.uniontech.com/server-enterprise-c/kongzi/1020/AppStream/x86_64/
enabled = 1
gpgcheck = 0
#module_hotfixes=true

[UniontechOS-$releasever-BaseOS]
name = UniontechOS $releasever BaseOS
#baseurl = https://cdimage.uniontech.com/server-dev/1060/e/release-0606/arm64/everything/
baserurl = http://10.30.38.131/kojifiles/repos/kongzi-build/latest/aarch64/
#baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/BaseOS/aarch64/os/
enabled = 1
gpgcheck = 0
#module_hotfixes=true

[UnionTechOS-$releasever-openstack]
name = UnionTechOS $releasever openstack
#baseurl = https://cdimage.uniontech.com/server-dev/1060/e/release-0606/arm64/OpenStack-V/
baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/OpenStack-Victoria/aarch64/os/
enabled = 1
gpgcheck = 0

[ceph]
name=ceph
baseurl=https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/Storage/aarch64/
gpgcheck=0
enabled=1
module_hotfixes=true

[Tools]
name = Tools
baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/PowerTools/aarch64/os/
enabled = 1
gpgcheck = 0

[PLUS]
name = plus
baseurl = https://cdimage.uniontech.com/server-dev/1060/a/RC/RC3/compose/Plus/aarch64/os/
enabled = 1
gpgcheck = 0
```
注：制作ustack-web需要修改start.sh 如下:/usr/local/share/kolla/docker/base/start.sh
![image.png](https://cdn.nlark.com/yuque/0/2023/png/26098815/1695277271879-efa44824-2de4-48af-86b1-8d348c42d10c.png#averageHue=%2314172a&clientId=u4d0409ea-d20d-4&from=paste&height=74&id=u8dd3bc39&originHeight=93&originWidth=591&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=9779&status=done&style=none&taskId=u8ca6c926-8137-456f-acc7-eeef95147c0&title=&width=472.8)
### 推送镜像到harbor
执行如下脚本前请先根据操作环境修改变量。
```bash
set -o errexit

TAG=victoria
KOLLA_NAMESPECE=kolla
UOS_REGISTRY=harbor.chinauos.com
UOS_NAMESPACE=kolla-ustack-v-x86
#ren-test项目中的镜像为基础镜像，kolla-ustack-v-x86项目中的镜像为持续更新的研发镜像。

KOLLA_SOURCE=$(docker images | awk /kolla/'{print $1}'| xargs -I {} echo -e {}':'${TAG})
for i in ${KOLLA_SOURCE}; do
        DOCKER_IMAGE_LINE=$(echo $i | tr ' ' '\n')
        UOS_TEST=${DOCKER_IMAGE_LINE#${KOLLA_NAMESPECE}}
        UOS_TAG=${UOS_REGISTRY}/${UOS_NAMESPACE}${UOS_TEST}
        echo ${UOS_TAG}
        docker tag ${DOCKER_IMAGE_LINE} ${UOS_TAG}
        docker push ${UOS_TAG}
        docker rmi -f ${UOS_TAG}
done
```

---
title: Openstack开发环境部署
date: 2023-12-15 10:27:28
description: Openstack开发环境部署
type: "tags"
comments: true
categories:
- Openstack
- Deploy
tags:
- openstack
---
kolla-ansible
参考文档：
https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html
https://docs.openstack.org/kolla-ansible/latest/user/troubleshooting.html
https://docs.openstack.org/kolla-ansible/latest/contributor/kolla-for-openstack-development.html

Gateway
pdb
查看容器挂载情况
docker inspect container_name | grep Mounts -A 20
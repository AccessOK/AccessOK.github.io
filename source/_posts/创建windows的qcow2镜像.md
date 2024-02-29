---
title: 创建windows的qcow2镜像
date: 2024-01-10 10:27:28
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

创建qcow2
```bash
qemu-img create -f qcow2 /path/to/win-iso/windows_server_2019.qcow2 50G
```

安装windows镜像到qcow2

```bash
virt-install --virt-type=kvm --name win-2019 --cpu=host --memory 3072 --vcpus=2 --os-type=windows --os-variant=windows --disk=//path/to/win-iso/cn_windows_server_2019_x64_dvd_4de40f33.iso,device=cdrom --disk=/path/to//win-iso/virtio-win-0.1.172.iso,device=cdrom --network=default,model=virtio --graphics vnc --disk=/path/to/win-iso/windows_server_2019.qcow2,size=50,bus=virtio,format=qcow2 --boot cdrom --check all=off
```

压缩镜像

```bash
qemu-img convert -O qcow2 windows_server_2019.qcow2 new-windows_server_2019.qcow2
```


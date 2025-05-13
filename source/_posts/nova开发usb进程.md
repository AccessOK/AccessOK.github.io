---
title: openstack usb 需求开发
date: 2024-07-07 10:27:28
description: openstack usb透传功能开发
type: "tags"
comments: true
categories:
- Openstack
- Program
- Status
tags:
- openstack
---
# nova目录分析
## nova组件：

- nova-api: nova-api主要接受外部nova外部api调用。
- nova-conductor:nova-conductor主要操作数据库。
- nova-comoute：主要通过libvirt等下层组件操作实例。
## nova目录详情：

![](https://cdn.nlark.com/yuque/0/2023/jpeg/26098815/1697079908226-8056d8e6-9aa7-4765-a7a3-44364d039db6.jpeg)

# WSGI
# sqlalchemy

# 创建数据库

- nova 早期的OpenStack只有nova一个数据库，里面存放了所有的关于虚拟机的表。如instance表：存放每一个主机主机信息（后面会介绍到）；quotas表：项目配额信息 ；fixed_ips表；块存储设备表等。
- nova_api 从nova数据库中移除的一部分全局数据表组成的数据库，如flavors、key_pairs、quotas等。noav_api的出现是为了解决大规模时消息队列和数据库瓶颈问题。
- nova_cell0 nova_cell0数据中存放了所有创建失败的instance。虚拟机创建失败后不属于任何一个cell，那么就记录在nova_cell0中。

参考：[https://hellowac.github.io/technology/python/sqlalchemy/](https://hellowac.github.io/technology/python/sqlalchemy/)
## nova-manager ： 
查看当前数据库版本：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/26098815/1694767694502-66030176-97af-4937-a435-c5cf10c24c4c.png#averageHue=%23242424&clientId=u2abe46d4-9561-4&from=paste&height=48&id=uc1d572e7&originHeight=60&originWidth=597&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=10895&status=done&style=none&taskId=uf0ad4f28-44e0-4bda-a19a-1f139f01310&title=&width=477.6)
注：当前数据库版本对应：nova/db/sqlalchemy/migrate_repo/versions/中文件版本
![image.png](https://cdn.nlark.com/yuque/0/2023/png/26098815/1694767901297-19b51841-3ed4-42d7-bf3e-f40a35adac8b.png#averageHue=%23e1e4d9&clientId=u2abe46d4-9561-4&from=paste&height=57&id=u8c277543&originHeight=71&originWidth=626&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=13762&status=done&style=none&taskId=ud0c0f397-4ba8-46af-908c-535569b2c09&title=&width=500.8)
可以使用：nova-manage db sync 同步数据库
![image.png](https://cdn.nlark.com/yuque/0/2023/png/26098815/1694768037496-897dff96-cbf0-456b-8b07-add31b402d3a.png#averageHue=%23242424&clientId=u2abe46d4-9561-4&from=paste&height=220&id=uc0d53f1f&originHeight=275&originWidth=1870&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=70814&status=done&style=none&taskId=ud35e5ad4-0cfe-48eb-8962-9718c0d3b4c&title=&width=1496)
同步失败，报错信息：
ERROR: Could not access cell0.
Has the nova_api database been created?
Has the nova_cell0 database been created?
Has "nova-manage api_db sync" been run?
Has "nova-manage cell_v2 map_cell0" been run?
Is [api_database]/connection set in nova.conf?
Is the cell0 database connection URL correct?
Error: Foreign key associated with column 'instance_actions.instance_uuid' could not find table 'instances' with which to generate a foreign key to target column 'uuid。
参考：[https://docs.openstack.org/nova/rocky/cli/nova-manage.html](https://docs.openstack.org/nova/rocky/cli/nova-manage.html)
分析问题：其他环境运行正常，删除新增的实体类文件之后，运行正常。
解决问题：学习nova创建实体类并同步创建数据库。
参考：[https://blog.csdn.net/chengqiuming/article/details/79672973](https://blog.csdn.net/chengqiuming/article/details/79672973)
参考：[https://stackoverflow.com/questions/28047027/sqlalchemy-not-find-table-for-creating-foreign-key](https://stackoverflow.com/questions/28047027/sqlalchemy-not-find-table-for-creating-foreign-key)
在实体类文件中添加meta.reflect()函数之后，执行nova-manage db sync ${version}成功。
```
nova-manage db sync 413
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/26098815/1695020664736-595464c3-6758-4c32-88c0-58eab95b44e7.png#averageHue=%23e6dba7&clientId=u4b9cd771-95ab-4&from=paste&height=864&id=ueca8c230&originHeight=1080&originWidth=1870&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=337902&status=done&style=none&taskId=uffc198c6-a123-471b-b581-a2d8af1a65a&title=&width=1496)
在nova_cell0数据库创建devices数据表成功。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/26098815/1695020575978-1a79ae14-767e-4f63-8afb-3f27f7548208.png#averageHue=%23f9f9f8&clientId=u4b9cd771-95ab-4&from=paste&height=864&id=uf2b38cc2&originHeight=1080&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=146635&status=done&style=none&taskId=u1c3ea37e-3f4f-4b42-b526-5204d17ef58&title=&width=1536)
注：nova_cell0 是不适用，用于保存失败数据。
分析原因：nova_api和nova_conductorl中的/etc/nova/nova.conf地址不一样。为什么不一样？下回分析。
解决方案：操作数据库的时候单独配置数据库链接数据
```
 nova-manage db sync \
    --database_connection mysql+pymysql://root:secretmysql@dbserver/nova_cell0?charset=utf8
```
# 开发查询接口
报错如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/26098815/1696926488107-53fe9b2f-b46c-478d-a67b-a900224e5c45.png#averageHue=%232c2e20&clientId=ube672ce4-aec2-4&from=paste&height=864&id=u8892f3a1&originHeight=1080&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=501052&status=done&style=none&taskId=u70a770cd-2815-4b1e-a73b-d80da7b7951&title=&width=1536)
引入定义装饰器的类

# 定时任务
重启nova-compute立即执行定时任务。

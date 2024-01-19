# 基于Nova开发Usb

## 初识Nova

​	Nova（OpenStack Compute Service）是 OpenStack 最核心的服务，负责维护和管理云环境的计算资源，同时管理虚拟机生命周期。

- Nova-Api服务

  主要用于接收和响应外部请求。

  - nova-api组件实现了RESTful API功能，是外部访问Nova的唯一途径。
  - 接收外部的请求并通过Message Queue将请求发送给其他的服务组件，同时也兼容EC2 API，所以也可以用EC2的管理工具对nova进行日常管理。

- Nova-Cert服务

  是Nova的证书管理服务，用来为EC2服务提供身份验证，仅仅是在EC2 API的请求中使用。

- Nova-Scheduler服务

  用于Nova的调度工作，在创建虚拟机时，由它选择将虚拟机创建在哪台计算节点上。

- Nova-Conductor服务

  这个是服务是计算节点访问数据库时的一个中间层，它出现的作用是防止计算节点的Nova-Compute服务直接访问数据库。

- Nova-Console服务

  Nova增强了它的控制台服务。控制台服务允许用户可以通过代理服务器访问虚拟化实例。这就涉及了一对新的守护进程（nova-console和nova-consoleauth).

- Nova-Consoleauth服务

  主要用于控制台的用户访问授权

- Nova-Novncproxy服务

  用于为用户访问虚拟机提供了一个VNC的代理。通过VNC协议，可以支持基于浏览器的novnc客户端，后面你在网页打开的console界面就是依靠这个服务提供的。

- Nova-Compute

  Nova-compute是Nova最重要的组件之一。

  - nova-compute 一般运行在计算节点上，通过Message Queue接收并管理VM的生命周期。
  - Nova-compute 通过Libvirt管理KVM，通过XenAPI管理Xen等。

## nova源码目录结构

```
accelerator/    # Cyborg 加速器
api/            # Nova API 服务
	__init__.py
    auth.py             # 身份认证中间件
    compute_req_id.py   # x-compute-request-id 中间件（oslo_middleware）
    metadata/           # Metadata API
    openstack/          # Nova v2.1 API
        __init__.py
        api_version_request.py  # 版本验证
        auth.py                 # noauth 中间件
        common.py               # 信息查询的工具函数
        compute/                # 每个 API 的入口点
        	from nova.api.openstack.compute.routes import APIRouterV21
        	routes.py			# 路由文件
        identity.py             # 验证项目是否存在
        requestlog.py           # 请求日志中间件
        urlmap.py               # url 映射
        versioned_method.py     # 版本信息
        wsgi.py                 # WSGI 相关抽象类
        wsgi_app.py             # WSGI 应用程序初始化方法
    validation/         # 请求体验证
    wsgi.py             # WSGI 原语（请求、应用、中间件、路由、加载器）
    
cmd/            # 各个 Nova 服务的入口程序
compute/        # Nova Compute 服务
conductor/      # Nova Conductor 服务 *** 处理需要协调的请求（构建/调整）、充当数据库代理或处理对象转换。***
conf/           # 所有的配置选项
console/        # nova-console 服务
db/             # 封装数据库操作
hacking/        # 编码规范检查
image/          # 封装镜像操作，Glance 接口抽象
keymgr/         # 密钥管理器实现
locale/         # 国际化相关文件
network/        # nova-network 服务
notifications/  # 通知相关功能
objects/        # 封装实体对象的 CURD 操作
pci/            # PCI/SR-IOV 支持
policies/       # 所有 Policy 的默认规则
privsep/        # oslo_privsep 相关
scheduler/      # Nova Scheduler 服务
servicegroup/   # 成员服务（membership），服务组
storage/        # Ceph 存储支持
tests/          # 单元测试
virt/           # 支持的 hypervisor 驱动
volume/         # 封装卷访问接口，Cinder 接口抽象

# 文件
__init__.py
availability_zones.py   # 区域设置的工具函数
baserpc.py              # 基础 RPC 客户端/服务端实现
block_device.py         # 块设备映射
cache_utils.py          # oslo_cache 封装
config.py               # 解析命令行参数
context.py              # 贯穿 Nova 的所有请求的上下文
crypto.py               # 包装标准加密数据元素
debugger.py             # pydev 调试
exception.py            # 基础异常类
exception_wrapper.py    # 封装异常类
filters.py              # 基础过滤器
i18n.py                 # 集成 oslo_i18n
loadables.py            # 可加载类
manager.py              # 基础 Manager 类
middleware.py           # 更新 oslo_middleware 的默认配置选项
monkey_patch.py         # eventlet 猴子补丁
policy.py               # 策略引擎
profiler.py             # 调用 OSProfiler
quota.py                # 每个项目的资源配额
rpc.py                  # RPC 操作相关的工具函数
safe_utils.py           # 不会导致循环导入的工具函数
service.py              # 通用节点基类，用于在主机上运行的所有工作者
service_auth.py         # 身份认证插件
test.py                 # 单元测试基础类
utils.py                # 工具函数
version.py              # 版本号管理
weights.py              # 权重插件
wsgi.py                 # 管理 WSGI 应用的服务器类
```

## rpci

https://jckling.github.io/2021/05/23/OpenStack/OpenStack%20Nova/index.html



## 数据库

https://xcodest.me/nova-cell-v2.html

![image-20231113201456185](/home/wang/.config/Typora/typora-user-images/image-20231113201456185.png)



nova主要有三个数据库，分别时nova，nova_api,nova_cell0。Nova Cell 模块以树型结构为基础，主要包括 API-Cell（根 Cell）与 Child-Cell 两种形式。API-Cell 运行 nova-api 服务，每个 Child-Cell 运行除 nova-api 外的所有 nova-*服务，且每个 Child-Cell 运行自己的消息队列、数据库及 nova-cells 服务。

cell的两种架构模式及工作原理
单cell部署 架构模式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/202012311730081.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xpaHVpaHVpMDA2,size_16,color_FFFFFF,t_70#pic_center)

多cell部署 架构模式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201231173031550.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xpaHVpaHVpMDA2,size_16,color_FFFFFF,t_70#pic_center)

下图整个有三部分组成，cell0， , cell1. cell2 位于最上层的cell0，也就是api-cell， 而下层的cell1与cell2则是平行对等的关系，他们之间无交互，相互独立，还可以继续增加cell3,cell4 。 而上层的api cell主要包括了
Nova API, Nova Scheduler, Nova Conductor 这3个 Nova 服务 ,同时在 API Cell 中还需要 MQ 提供组件内的通信服务。API Cell 中的 DB 包含两个数据库，分别是 api数据库 和 cell数据库，api 数据库保存了全局数据，比如 flavor 信息。此外 api 数据库中还有一部分表是用于 placement 服务的；而 cell数据库则是用于保存创建失败且还没有确定位于哪个 cell 的虚机数据，比如当虚拟机调度失败时，该虚拟机数据就会被保存到cell数据库中。也就是cell0数据库中。



在每个 Cell 中，都有自己独立使用的数据库、消息队列和 Nova Conductor 服务，当前 Cell 中的所有计算节点，全部将数据发送到当前 Cell 中的消息队列，由 Nova Conductor 服务获取后，保存至当前 Cell 的 Nova 数据库中。整个过程都不会涉及到 API Cell 中的消息队列。因此通过对计算节点进行 Cell 划分，可以有效降低 API Cell 中消息队列和数据库的压力。假如一个 MQ 能支持200个计算节点，则在划分 Cell 以后，每个 Cell 都可以支持200个计算节点，有 N 个 Cell 就可以支持 N X 200 个计算节点，因此可以极大提升单个 OpenStack 的集群管理规模。

3 ， Cell v2实现的原理
在大致了解了 Cell V2 架构的基本组成后，接下来介绍一下在 Nova 组件中，究竟是如何实现 Cell 划分的。多 Cell 的实现涉及 nova_api 数据库中的3个表，分别是 cell_mappings, host_mappings, instance_mappings 表。这3个表之间的关系如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201231173050257.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xpaHVpaHVpMDA2,size_16,color_FFFFFF,t_70#pic_center)

cell_mappings 表记录了每个 Cell 的名字和其消息队列连接地址与数据库连接地址，通过该表中记录的信息，API Cell 中的 Nova API 服务和 Nova Conductor 服务就知道该如何连接到 Cell 中的消息队列和数据库了，并进一步将消息发送到 Cell 中的消息队列，或者直接访问 Cell 中的 Nova 数据库。

在 host_mappings 表记录了计算节点和 Cell 之间的对应关系，而instance_mappings 表则记录了 instance 和 Cell 之间的对应关系。通过这两个表的映射关系，API Cell 中的服务就可以轻易知道计算节点或者虚拟机所处的 Cell，并通过 cell_mappings 数据表中提供的链接对其进行操作。


## 开发环境部署

### 部署环境

- 使用kolla-ansible快速部署搭建all-in-one：

ubuntu搭建参考链接：https://docs.openstack.org/kolla-ansible/2023.1/user/quickstart.html

- 使用kolla-dev-mode=true部署，将会拉取源码并挂载到容器,可实现直接修改源码之后重启容器即可生效。

参考链接：https://docs.openstack.org/kolla-ansible/latest/contributor/kolla-for-openstack-development.html

- 可在kolla-ansible源码中，修改{{ kolla_dev_repos_git  }}和{ nova_source_version }重新定义拉取源码的仓库和分支。拉取仓库依赖git，请提前安装。

### 接口测试

#### 获取token

1.openstackclient 命令行获取

```plain
# openstack token issue
```

参考： https://blog.csdn.net/qq_30657195/article/details/108055043

2.异地curl获取

```plain
# curl -i   -H "Content-Type: application/json"   -d '
{ "auth": {
    "identity": {
      "methods": ["password"],
      "password": {
        "user": {
          "name": "admin",
          "domain": { "id": "default" },
          "password": "admin12#$"
        }
      }
    }
  }
}'   "http://10.10.15.184:5000/v3/auth/tokens" ; echo
```

https://docs.openstack.org/keystone/pike/api_curl_examples.html

3.工具posttman获取

根据curl命令修改postman参数，根据-H修改header参数，-d修改body参数。

header修改如图所示:

![img](https://cdn.nlark.com/yuque/0/2023/png/26098815/1694746908100-354fb3bb-08f5-4965-9813-3f72a3ef0ec3.png)

body修改如图所示：

![img](https://cdn.nlark.com/yuque/0/2023/png/26098815/1694746941243-c32f94bd-c1e5-4344-b37a-f5a449e548b5.png)

token值如图所示：

![img](https://cdn.nlark.com/yuque/0/2023/png/26098815/1694746997062-460d6c18-1591-4a42-b8a2-3570bb5c7c4c.png)

#### 调用api

1. 使用curl命令调用

```plain
curl -s http://10.10.15.184:9292/images -H 'X-Auth-Token:gAAAAABlA8lIs87kbEYq85mnARenwHlLt_Nv_XflgQXJNBAM4tFcNAf8kG9fmXDRQCHZFaLu4u9cDNCLKADIwpkSSqNWDTI2lVLd02OD74NNG3tdUCSFs1KC6JAW0Bsv9LXnokrema_nwshrXcBwGvsBCb0RnNA60g'
```

参考：https://www.linux.com/training-tutorials/spinning-server-openstack-api/

1. 使用postman调试

![img](https://cdn.nlark.com/yuque/0/2023/png/26098815/1694749070795-8bc5bf37-7a04-4d1d-b844-92ca9f15a762.png)

注：X-Auth-Token是发送请求时使用，X-Subject-Token是服务器响应请求时传回的参数。

https://support.huaweicloud.com/intl/en-us/devg-roma/apic-dev-190216017.html

#### LOG

1. 可以使用pdb 等工具进行调试。

参考：https://docs.openstack.org/oslotest/queens/user/features.html

1. 或者LOG.info LOG.debug等输出日志。log文件输出的信息等级需要配置，只有符合配置文件/etc/nova/nova.conf中的日志等级的日志才会被输出。

参考：https://docs.openstack.org/nova/pike/admin/manage-logs.html

# Openstack-实例网络联通性调研测试

环境：Openstack-v 

先删除所有网络，子网，端口，路由。

## 测试步骤

- 新建selfservice网络并创建实例，测试网络的封闭性。
- 在selfservice网路创建实例，测试局域网的连通性。
- 创建provider 网络，并通过路由连接到selfservice网络。连接实例与外部机器。测试路由有效。
- 删除路由，创建浮动ip。浮动ip绑定实例，通过实例连接外部机器，外部机器通过浮动ip连接实例，测试浮动ip有效。

## 云管平台验证

1.创建私有网络。

![image-20231228184850234](/home/wang/UOS/AccessOK/blog/source/images/image-20231228184850234.png)

![image-20231228185457722](/home/wang/UOS/AccessOK/blog/source/images/image-20231228185457722.png)

2.创建私有网络子网。

![image-20231228185607415](/home/wang/.config/Typora/typora-user-images/image-20231228185607415.png)

![image-20231228185958149](/home/wang/.config/Typora/typora-user-images/image-20231228185958149.png)

3.通过私有网络创建实例。

创建实例失败。

![image-20231228191201350](/home/wang/UOS/AccessOK/blog/source/images/image-20231228191201350.png)

4.选中之后批量删除实例，实例假删除。但是回收站中没有刚删除的实例。

![image-20231228191755526](/home/wang/UOS/AccessOK/blog/source/images/image-20231228191755526.png)

但是后台实例并没有删除。

![image-20231228191834263](/home/wang/UOS/AccessOK/blog/source/images/image-20231228191834263.png)

BUG：批量删除没有删除到回收站，数据库进行了删除操作，但是个人回收站无法查询到刚删除的实例。

5.关闭“新建卷”创建实例成功，但是实例里面没有ip。

![image-20231228201621724](/home/wang/UOS/AccessOK/blog/source/images/image-20231228201621724.png)

![image-20231228203258610](/home/wang/UOS/AccessOK/blog/source/images/image-20231228203258610.png)

6.同样的方式再创建一个实例。

![image-20231228202958546](/home/wang/UOS/AccessOK/blog/source/images/image-20231228202958546.png)

![image-20231228203329441](/home/wang/UOS/AccessOK/blog/source/images/image-20231228203329441.png)

BUG：实例端口显示两个相同的ip

![image-20231228203146416](/home/wang/UOS/AccessOK/blog/source/images/image-20231228203146416.png)

7.通过dashboard 创建实例，查看ip状态。同样没有ip。
![image-20231228212703017](/home/wang/UOS/AccessOK/blog/source/images/image-20231228212703017.png)

8.更新148环境的iptables版本。并重启148环境，创建实例之后，有ip:10.100.10.20。

![image-20231228223002957](/home/wang/UOS/AccessOK/blog/source/images/image-20231228223002957.png)

9.在次创建实例，测试实例互通性。由此可知vxlan网络内实例互通。

![image-20231229135018445](/home/wang/UOS/AccessOK/blog/source/images/image-20231229135018445.png)

10.创建新的vxlan网络和子网。

![image-20231229141136199](/home/wang/UOS/AccessOK/blog/source/images/image-20231229141136199.png)

![image-20231229141435188](/home/wang/UOS/AccessOK/blog/source/images/image-20231229141435188.png)

11.用不同局域网创建实例，测试不同局域网的连通性。预估结果，不同局域网之间实例链接失败。

![image-20231229151744918](/home/wang/UOS/AccessOK/blog/source/images/image-20231229151744918.png)

12.创建路由，链接两个局域网。

![image-20231229151952217](/home/wang/UOS/AccessOK/blog/source/images/image-20231229151952217.png)

![image-20231229152313834](/home/wang/UOS/AccessOK/blog/source/images/image-20231229152313834.png)

![image-20231229152826265](/home/wang/UOS/AccessOK/blog/source/images/image-20231229152826265.png)

13.再次连通两个实例，实例链接成功。

![image-20231229153048490](/home/wang/UOS/AccessOK/blog/source/images/image-20231229153048490.png)

14.创建provider网络。

![image-20231229153845717](/home/wang/UOS/AccessOK/blog/source/images/image-20231229153845717.png)

15.创建provider子网

![image-20231229154121220](/home/wang/UOS/AccessOK/blog/source/images/image-20231229154121220.png)

![image-20231229155051281](/home/wang/UOS/AccessOK/blog/source/images/image-20231229155051281.png)

16.验证实例不能连通外部网络。

![image-20231229155148823](/home/wang/UOS/AccessOK/blog/source/images/image-20231229155148823.png)

17.使用provider网络创建实例，并测试实例的外网连通性。

![image-20231229160925504](/home/wang/UOS/AccessOK/blog/source/images/image-20231229160925504.png)

18.外部连通失败。执行如下脚本，关闭环境宿主机端口。嵌套openstack环境，所以需要关闭所有网络的端口安全。

```bash
server_id=$1

source /root/admin-openrc.sh
port_ids=`openstack port list --server $server_id -f value -c ID`

for i in ${port_ids[@]}; 
do
    echo "disable security for port $i";
    openstack port set --disable-port-security --no-security-group $i;
done
```



## 问题反馈

wlw：删除快照失败。
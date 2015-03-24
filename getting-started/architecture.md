---
title: Architecture
layout: default
---

Xapi toolstack为下列产品管理着集群主机，网络交换机和存储
[XenCenter](https://github.com/xenserver/xenadmin),
[Xen Orchestra](https://xen-orchestra.com),
[OpenStack](http://www.openstack.org)
and [CloudStack](http://cloudstack.apache.org).

最基本的概念是*资源池*: 整个集群作为一个单一节点进行管理. 下面这张图显示了一个运行Xapi的集群，它们还使用了共享存储:

![A Resource Pool](pool.png)

在任何时候, 最多只有一个主机叫做 *主节点*，它负责资源池内的资源交流和锁定. 当资源池第一次创建的时候会选定一个节点作为主节点. 其他节点在下列情况下会转换为主节点

- 由 (`xe pool-designate-new-master`)命令发起的请求
- 由用户发起的紧急请求 (`xe pool-emergency-transition-to-master`)
- 由集群中的高可用(HA)触发.

所有的主机都提供一个HTTP(运行在80端口上)和XML/RPC接口，还有运行在443端口上的TLS/SSL接口, 但是控制操作都值在主节点上被处理.
尝试往其他节点上发送的请求会导致一个XenAPI重定向的错误消息. 为了执行效率下列的操作可以在从节点上运行:

- 查询性能数据(和历史数据)
- 连接VNC控制台
- 导入/导出(特别是虚拟磁盘在本地存储上的情况)

因为主节点作为资源交流和锁定的管理者, 其他的节点经常与它通信. 从节点相互之间也经常通过(同样的HTTP和XMLRPC通道) 通信，为了

- 传递虚拟机内存镜像(*虚拟机迁移*)
- 传递磁盘镜像(*存储迁移*)

注意有些类型的共享存储(特别是使用vhd格式的)需要交流磁盘GC和合并的信息. 这种交流现在是由xapi来做的因为不可能在资源池里共享这种信息.

下面的显示了一个节点上运行的软件. 注意所有的节点都运行同样软件 (尽管在池滚动升级的中间时刻，可能版本不同).

![A Host](host.png)

Xapi toolstack预期主机运行在Xen 4.4及以上版本, x86或者ARM架构. Xen hypervisor把主机分为不同的 *Domains*, 有些有特权硬件访问权限, 剩下的是非特权guest.
Xapi toolstack通常运行在内部的特权domain, Domain 0, 也叫做 "control domain". 然后有些试验性质的代码可以支持 "driver domains"， 它允许存储和网络的驱动被隔离在自己的domain里.

Xapi toolstack包含一系列用于交流的守护进程:

- [Xapi](https://github.com/xapi-project/xen-api): 管理主机集群, 控制存储和网络的访问.
- [Xenopsd](https://github.com/xapi-project/xenopsd): 一个底层的
  "domain管理程序"，它使用libxc和libxl与Xen交互来创建，挂起，恢复，迁移，重启domain.
- [Xcp-rrdd](https://github.com/xapi-project/xcp-rrdd): 一个性能统计监测进程程序，聚合了由API插件定义的数据集和历史数据.
- [Xcp-networkd](https://github.com/xapi-project/xcp-networkd):
  一个网络管理程序，负责配置网卡, 网桥和OpenVSwitch实例
- [SM](https://github.com/xapi-project/sm): 一个存储管理插件，它跟Xapi内部存储接口通信来控制外部的存储系统.
- *perfmon*: 一个监控性能和发送超过预定义阀值的"警报"的守护进程
- *mpathalert*: 一个监控存储路径的守护进程，它会在存储路径出问题需要修复时发送"警报"
- *snapwatchd*: 一个相应快照请求的守护进程，通常请求来自Windows guest内部的VSS代理.
- *stunnel*: 一个解析TLS/SSL和把通讯信息转交给Xapi的守护进程.
- *xenconsoled*: 运行访问guest控制台. Xen主机上的常用功能.
- *xenstored*: 一个由键值对组成的数据库，包含虚拟机的磁盘网络等配置信息. 这也是所有的主机都常见的功能

Domain 0中的内部控制操作通信均使用Unix domain socket结合json RPC和XenAPI. Xen domain之间的通信使用shared memory (通过 grant mapping or grant copy)和事件通道(event-channels), 所有的参数均配置在Xenstore中.

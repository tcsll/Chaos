# ovirt是什么

ovirt是rhev的开源版，rhev全称为Red Hat Enterprise virtualization，红帽公司对企业推出的商业私有云平台的一个软件。其中可以做服务器虚拟化，桌面虚拟化，比较能拿得出手的功能

1. 支持spice协议、RDP协议、vnc协议

2. 支持物理设备直通

3. cloud-init，开机时做一些自动化安装工作，例如配置主机名，网卡信息，可以定义脚本装一下软件包等等。。

4. 貌似可以直接接AWS的存储或者openstack的cinder，但是没用过，公司没有对应的环境

5. 支持各种存储，nfs，posix，iscsi，FC

**目前公司用的ovirt超融合版本是4.2.4.5**

目前最新4.41

# ovirt组织架构

1. 网页，网页采用GWT架构，基本国内没有做相关开发的人员，就这个编译环境搭建就特别麻烦，java写的，3版本的ovirt性能很差，如果客户端的机器性能比较低，那么虚拟机太多批量开机时都有可能卡死，网页支持java的api，和python的sdk接口，可以直接调用做二次开发
2. 底层，kvm不多说了，就是virsh那一套
3. 数据库，神奇的pgsql。用这个数据库，我只想说，老外果然跟中国人不一样，中国人肯定会选mysql。
4. 操作系统，红帽做的，肯定是rhel和centos 系列的系统
5. 系统原生兼容glusterfs

# ovirt的搭建模式

ovit分为管理端与运算端，管理端他们叫为engine，运算端统称为node，三种搭建模式分别为allinone，hosted-engine，普通模式

1. allinone，这种模式3.6以后就不再支持了，之前基本作为测试环境
2. hosted-engine，管理端高可用模式，其中engine作为一台虚拟机漂在所有的node节点上，如果当engine所在的node节点锁坏，那么会自动迁移到另一个node节点上。
3. 普通模式，engine一台机器，node一台机器。engine是单点

# ovirt运维

**ovirt-engine无法接入域控用户认证**

1. 检查/etc/hosts 文件 ，是否能ping通域名
2. 检查/etc/resolve.conf文件，是否能解析域名
3. 检查域控服务器是否安装dns服务
4. 域控服务器中新建的admin用户是否隶属于超级管理员
5. 是否开启了委派模式
6. 都没有问题，域控服务器中删除admin用户重新创建（到这部已经是人类无法解释的问题了）

**如何解锁虚拟机，当engine管理页面中虚拟机被锁死了，那么可能需要进入后台进行一些数据库相关操作**

```sql
#psql engine -U postgres -c "UPDATE vm_dynamic set status=0 where vm_guid=(select vm_guid from vms where vm_name='Win2008_QBPT');"
```

**如何使用glusterfs服务，如果不使用原生的gluster服务，需要在iptables里开启对应端口**
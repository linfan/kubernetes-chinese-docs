# 离线安装（使用裸机和CoreOS系统）
`译者：林帆` `校对：无`


这个部分将介绍在CoreOS上运行Kubernetes的方法。比较特别的是，我们将关注于如何在离线的情况下完成部署操作，这对于进行概念验证和在访问受限的网络环境时，都将派上用场。

内容目录：

- 前提条件
- 总体设计思路
- 示例环境
- 构建CentOS的PXELINUX服务端
- 添加CoreOS系统到PXE
- 配置DHCP服务
- 部署Kubernetes
- CoreOS的Cloudinit启动配置
  - master.yml
  - node.yml
- 新建pxelinux.cfg文件
- 指定pxelinux目标
- 创建测试的Pod
- 用于调试的指令

## 前提条件

1. 已经有一个主机安装有运行着PXE服务端的CentOS6系统
2. 至少有两个裸机主机节点

## 总体设计思路

- 通过TFTP服务管理以下目录
  - /tftpboot/(coreos)(centos)(RHEL)
  - /tftpboot/pxelinux.0/(MAC) -> 链接到Linux镜像中的config文件
- 将PXELinux的链接更新到正确状态
- 将DHCP服务配置根据实际主机需求更新到正确状态
- 构建部署CoreOS的主机节点，并运行Etcd的集群
- 在不使用公有网络的情况下完成后续配置
- 部署更多的CoreOS节点作为Kubernetes的Node节点

## 示例环境

| 节点 | MAC | IP |
| --- |:---:| ---:|
| CoreOS/etcd/Kubernetes Master | d0:00:67:13:0d:00 | 10.20.30.40 |
| CoreOS Slave 1 | d0:00:67:13:0d:01 | 10.20.30.41 |
| CoreOS Slave 2 | d0:00:67:13:0d:02 | 10.20.30.42 |

## 构建CentOS的PXELINUX服务端

完整的构建CentOS PXELinux环境方法参见[这篇文档](http://docs.fedoraproject.org/en-US/Fedora/7/html/Installation_Guide/ap-pxe-server.html)。下面简单叙述这个流程：

1. 安装CentOS上所需的包：<br>
   
   `sudo yum install tftp-server dhcp syslinux`
2. 编辑TFTP服务配置，将`disable`一项修改为`no`：<br>
   
   `vi /etc/xinetd.d/tftp`
3. 拷贝所需的syslinux镜像文件<br>

	```
	su - 
	mkdir -p /tftpboot
	cd /tftpboot 
	cp /usr/share/syslinux/pxelinux.0 /tftpboot 
	cp /usr/share/syslinux/menu.c32 /tftpboot 
	cp /usr/share/syslinux/memdisk /tftpboot 
	cp /usr/share/syslinux/mboot.c32 /tftpboot 
	cp /usr/share/syslinux/chain.c32 /tftpboot
	/sbin/service dhcpd start 
	/sbin/service xinetd start 
	/sbin/chkconfig tftp on
	```
4. 生成默认的启动菜单

   ```
   mkdir /tftpboot/pxelinux.cfg 
   touch /tftpboot/pxelinux.cfg/default
   ```
5. 编辑启动菜单`vi /tftpboot/pxelinux.cfg/default`，内容如下：

   ```
   default menu.c32 prompt 0 timeout 15 ONTIMEOUT local display boot.msg
	MENU TITLE Main Menu
	LABEL local MENU LABEL Boot local hard drive LOCALBOOT 0
	```
现在你就已经有了一个可以用于构建CoreOS节点的PXELinux服务了，可以通过本地的VirtualBox虚拟机或裸机上验证这个服务。

## 添加CoreOS系统到PXE


## 配置DHCP服务
## 部署Kubernetes
## CoreOS的Cloudinit启动配置
### master.yml
### node.yml
## 新建pxelinux.cfg文件
## 指定pxelinux目标
## 创建测试的Pod
## 用于调试的指令

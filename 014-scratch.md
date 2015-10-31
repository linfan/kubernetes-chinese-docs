
# 从零开始

这部文档是面对想要订制Kubernetes集群的读者。如果你发现现有的入门指南已经可以满足你对 [这个列表](README.md)上所列的需求，我们建议你继续阅读这个根据前人积累经验所写的新手指南。但如果你有入门指南所不能满足的对IaaS，网络，配置管理或对操作系统有特殊要求，这个指南将会提供给你一个指导性的概述。
这个指南也会对希望对现有集群配置有进一步理解的读者提供到帮助。

## 设计和准备

### 学习

  1. 你应该已经熟悉Kubernetes了。我们建议根据 其他入门指南架设一个临时的集群。这样可以帮助你先熟悉Kubernetes命令行([kubectl](../user-guide/kubectl/kubectl.md))和一些概念([pods](../user-guide/pods.md), [services](../user-guide/services.md), etc.)。
  2. 当你浏览完其他入门指南的时候，你应该已经安装好了`kubectl`。如果没有，你可以根据[这个](../user-guide/prereqs.md)说明安装。

### Cloud Provider

Kubernetes有一个概念叫Cloud Provider，是指一个提供管理TCP负载均衡，节点（实例）和路由器接口的模块。`pkg/cloudprovider/cloud.go`里具体定义了这个接口。
当然，你也可以不用实现这个Cloud Provider去新建一个自定义的聚群（例如，使用裸机集群）。取决于不同部件是如何设置的，并不是所有接口都需要实现的。

### 节点
* 你可以使用虚拟机或物理机。
* 为了运行本指南给出的例子和测试用例，你最好有4个节点以上。
* 虽说很多入门指南讲主节点和普通节点做了区分，但严格意义上讲，这不是必要的。
* 节点需要运行在x86_64的Linux系统上。当然也可能运行在其他的系统和CPU架构上，但这个指南不会提供相关的帮助。
* 对于一个拥有10个以上节点的集群来说，Apiserver和etcd可以一起安装在一台1核和1GB内存的机子上。
* 你可以给其他节点可以分配合理的内存和CPU内核。不是所有节点需要同样的配置。

### 网络

Kubernetes有一个独特的[网络模型](../admin/networking.md).

Kubernetes给每一个pod分配IP地址。当你新建一个集群，为了保证Pod获得IP地址，你需要给Kubernetes分配一个IP地址池。最简单的做法是每当

节点与节点之间的通信可以以一下两种方式实现：
* 配置网络完成Pod的IP地址路由
 * 因为一切从头开始，所以难度会大一些。
 * Google Compute Engine ([GCE](gce.md)) 和 [AWS](aws.md)会指导如何使用这种方式
 * 需要编程配置路由器和交换机去实现Pod的IP地址路由。
 * 可在Kubernetes环境外配置或者通过在Cloud Provider模块的“路由”接口里实现。
 * 通常情况下，提供最优网络性能。

* 建立一个拓扑网络
  * 较容易建立
  * 因为数据流量被封装，所以每个Pod的IP地址是可以路由的。
  * 例如：    
    *  [Flannel](https://github.com/coreos/flannel)
    *  [Weave](http://weave.works/)
    *  [Open vSwitch (OVS)](http://openvswitch.org/)
    
  * 不需要CLoud Provider模块里的“路由”部分 
  * 较为不太理想的性能（具体的性能弱化取决于你的实际情况） 

你需要为Pod所需要的IP地址选一个IP地址范围。
*  一些可选择配置方式：
  * GCE：每一个项目有一个自己的IP子网“10.0.0.0/8”。项目中的每一个Kubernetes集群从中获得一个“/16”的子网。每一个节点从'/16'的子网里获IP地址。
  * AWS：在一个组织内使用一个VPC。从中分配地址给每一个聚群或者使用给不同的集群分配不同的VPC。
  * 暂不支持支持IPv6
*  给每一的node的Pod地址分配一个CIDR子网或者一个
  * 你总共需要max-pods-per-node * max-number-of-nodes个IP地址。一个“/24” 如果缺少IP地址，一个“/26”（62个节点）或者“/27”（30个节点）也能满足。
  * 例如，使用“10.10.0.0/16” “10.10.0.0/24” “10.10.255.0/24”
  * 需要路由设置或连接到拓扑网络

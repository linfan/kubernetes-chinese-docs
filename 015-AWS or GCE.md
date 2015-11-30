
# **[CoreOS](https://coreos.com)上部署入门指南**

在[Coreos](https://coreos.com/kubernetes/docs/latest/)运行Kubernetes有多个指南:

## **CoreOS官方指南**

这些指南由CoreOs维护，“CoreOS Way”提供了TLS协议，DNS add－on等部署Kubernetes的指南。指南都通过了Kubernetes的一致性测试，当然，也鼓励自己[测试自己的部署](https://coreos.com/kubernetes/docs/latest/conformance-tests.html)。

### [Vagrant Multi-Node](https://coreos.com/kubernetes/docs/latest/kubernetes-on-vagrant.html)

这是在Vagrant上搭建多节点集群的指南。部署者可以单独配置etcd nodes，master nodes和worker nodes节点数，来调出一个完整的HA的控制板。

### [Vagrant Single-Node](https://coreos.com/kubernetes/docs/latest/kubernetes-on-vagrant-single.html)

这是一个在本地快速搭建Kubernetes开发环境的方式。简单的```git clone```，```vagrant up```和配置```kubectl```三步就可以了。



### [Full Step by Step Guide](https://coreos.com/kubernetes/docs/latest/getting-started.html)

这是在任何云或者机器上搭建一个TLS协议的HA集群的一般的指南。根据角色重复master或者worker的部署去配置更多的机器。



## **社区指南** 

这些指南由社区人员维护，包括一些特殊平台，使用案例和在CoreOS使用不同的方式来配置Kubernetes的经验。

### [Multi-node Cluster](coreos/coreos_multinode_cluster.html)

在选择的平台中：AWS，GCE，或者VMware Fusion搭建一个单个master，multi-worker集群的指南。

### [Easy Multi-node Cluster on Google Compute Engine](https://github.com/rimusz/coreos-multi-node-k8s-gce/blob/master/README.md)

在GCE上通过脚本安装一个单个master，multi-worker的集群的指南。使用[fleet](https://github.com/coreos/fleet)来管理Kubernetes的部件。 

### [Multi-node cluster using cloud-config and Weave on Vagrant](https://github.com/errordeveloper/weave-demos/blob/master/poseidon/README.md)

配置一个Vagrant-based集群，包括3台带有Weave网络的机器的指南。

### [Multi-node cluster using cloud-config and Vagrant](https://github.com/pires/kubernetes-vagrant-coreos-cluster/blob/master/README.md)

通过选择的虚拟管理程序：VirtualBox，Parallels或者Parallels配置一个单个master，multi-worker本地集群的指南。

### [Multi-node cluster with Vagrant and fleet units using a small OS X App](https://github.com/rimusz/coreos-osx-gui-kubernetes-cluster/blob/master/README.md)

这是通过OS X应用程序控制运行一个单个master, multi-worker集群指南。Under the hood使用Vagrant。

### [Resizable multi-node cluster on Azure with Weave](coreos/azure/README.html)

在Azure上运行一个HA etcd集群，包括一个单个master的指南。使用Azure node.js CLI去扩展集群。


### [Multi-node cluster using cloud-config, CoreOS and VMware ESXi](https://github.com/xavierbaude/VMware-coreos-multi-nodes-Kubernetes)

在VMware ESXi配置一个单个master，单个worker集群的指南。
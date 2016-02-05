# 创建Kubernetes集群
`译者：razr` `校对：钟健鑫`


Kubernetes可以在多种平台运行，从笔记本电脑，到云服务商的虚拟机，再到机架上的裸机服务器。要创建一个Kubernetes集群，根据不同场景需要做的也不尽相同，可能是运行一条命令，也可能是配置自己的定制集群。这里我们将引导你根据自己的需要选择合适的解决方案。

## 选择正确的解决方案

如果你只是想试一试Kubernetes，我们推荐[基于Docker的本地](docker.md)方案。

基于Docker的本地方案是众多能够完成快速搭建的[本地集群](#本地服务器方案)方案中的一种，但是局限于单台机器。

当你准备好扩展到多台机器和更高可用性时，[托管](#托管方案)解决方案是最容易搭建和维护的。

[全套云端方案](#全套云端方案) 只需要少数几个命令就可以在更多的云服务提供商搭建Kubernetes。

[定制方案](#定制方案) 需要花费更多的精力，但是覆盖了从零开始搭建Kubernetes集群的通用建议到分步骤的细节指引。

### 本地服务器方案

本地服务器方案再一台物理机上创建拥有一个或者多个Kubernetes节点的单机集群。创建过程是全自动的，且不需要任何云服务商的账户。但是这种单机集群的规模和可用性都受限于单台机器。

本地服务器方案有：

  - [本地Docker](docker.md)（上手建议）
  - [Vagrant](vagrant.md) (任何支持Vagrant的平台：Linux，MacOS，或者Windows。)
  - [无虚拟机本地集群](locally.md) (Linux)

### 托管方案

[Google Container Engine](https://cloud.google.com/container-engine) 提供创建好的Kubernetes集群。

### 全套云端方案

以下方案让你可以通过几个命令就在很多IaaS云服务中创建Kubernetes集群，并且有很活跃的社区支持。

- [GCE](gce.md)
- [AWS](aws.md)
- [Azure](coreos/azure/README.md)

### 定制方案

Kubernetes可以在云服务提供商和裸机环境运行，并支持很多基本操作系统。

如果你再如下的指南中找到了符合你需要的，可直接使用。某些指南可能有些过时，但是比起从零开始还是有不少参考价值。如果你确实因为特殊原因或因为想了解底层原理，想要从零开始搭建，可以试试参考[从零开始](scratch.md)指南。

如果你对在新的平台支持Kubernetes感兴趣，可以看看我们的[写新方案的建议](../../docs/devel/writing-a-getting-started-guide.md)。

#### 云
以下是上文没有列出的云服务商或云操作系统支持的方案。

- [AWS + coreos](coreos.md)
- [GCE + CoreOS](coreos.md)
- [AWS + Ubuntu](juju.md)
- [Joyent + Ubuntu](juju.md)
- [Rackspace + CoreOS](rackspace.md)

#### 私有虚拟机

- [Vagrant](coreos.md)（采用CoreOS和flannel）
- [CloudStack](cloudstack.md)（采用Ansible，CoreOS和flannel）
- [Vmware](vsphere.md)（采用Debian）
- [juju.md](juju.md)（采用Juju，Ubuntu和flannel）
- [Vmware](coreos.md)（采用CoreOS和flannel）
- [libvirt-coreos.md](libvirt-coreos.md)（采用CoreO）
- [oVirt](ovirt.md)
- [libvirt](fedora/flannel_multi_node_cluster.md)（采用Fedora和flannel）
- [KVM](fedora/flannel_multi_node_cluster.md)（采用Fedora和flannel）

#### 裸机

- [Offline](coreos/bare_metal_offline.md)（无需互联网，采用CoreOS和flannel）
- [fedora/fedora_ansible_config.md](fedora/fedora_ansible_config.md)
- [Fedora单节点](fedora/fedora_manual_config.md)
- [Fedora多节点](fedora/flannel_multi_node_cluster.md)
- [Centos](centos/centos_manual_config.md)
- [Ubuntu](ubuntu.md)
- [Docker多节点](docker-multinode.md)

#### 集成

- [Kubernetes on Mesos](mesos.md)（采用GCE）

## Table of Solutions

以下用表格形式列出上面的所有方案。

IaaS Provider        | Config. Mgmt | OS     | Networking  | Docs                                              | Conforms | Support Level
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
GKE                  |              |        | GCE         | [docs](https://cloud.google.com/container-engine) | [✓][3]   | Commercial
Vagrant              | Saltstack    | Fedora | flannel     | [docs](vagrant.md)                                | [✓][2]   | Project
GCE                  | Saltstack    | Debian | GCE         | [docs](gce.md)                                    | [✓][1]   | Project
Azure                | CoreOS       | CoreOS | Weave       | [docs](coreos/azure/README.md)                    |          | Community ([@errordeveloper](https://github.com/errordeveloper), [@squillace](https://github.com/squillace), [@chanezon](https://github.com/chanezon), [@crossorigin](https://github.com/crossorigin))
Docker Single Node   | custom       | N/A    | local       | [docs](docker.md)                                 |          | Project ([@brendandburns](https://github.com/brendandburns))
Docker Multi Node    | Flannel      | N/A    | local       | [docs](docker-multinode.md)                       |          | Project ([@brendandburns](https://github.com/brendandburns))
Bare-metal           | Ansible      | Fedora | flannel     | [docs](fedora/fedora_ansible_config.md)           |          | Project
Digital Ocean        | custom       | Fedora | Calico      | [docs](fedora/fedora-calico.md)                   |          | Community (@djosborne)
Bare-metal           | custom       | Fedora | _none_      | [docs](fedora/fedora_manual_config.md)            |          | Project
Bare-metal           | custom       | Fedora | flannel     | [docs](fedora/flannel_multi_node_cluster.md)      |          | Community ([@aveshagarwal](https://github.com/aveshagarwal))
libvirt              | custom       | Fedora | flannel     | [docs](fedora/flannel_multi_node_cluster.md)      |          | Community ([@aveshagarwal](https://github.com/aveshagarwal))
KVM                  | custom       | Fedora | flannel     | [docs](fedora/flannel_multi_node_cluster.md)      |          | Community ([@aveshagarwal](https://github.com/aveshagarwal))
Mesos/Docker         | custom       | Ubuntu | Docker      | [docs](mesos-docker.md)                           |          | Community ([Kubernetes-Mesos Authors](https://github.com/mesosphere/kubernetes-mesos/blob/master/AUTHORS.md))
Mesos/GCE            |              |        |             | [docs](mesos.md)                                  |          | Community ([Kubernetes-Mesos Authors](https://github.com/mesosphere/kubernetes-mesos/blob/master/AUTHORS.md))
AWS                  | CoreOS       | CoreOS | flannel     | [docs](coreos.md)                                 |          | Community
GCE                  | CoreOS       | CoreOS | flannel     | [docs](coreos.md)                                 |          | Community ([@pires](https://github.com/pires))
Vagrant              | CoreOS       | CoreOS | flannel     | [docs](coreos.md)                                 |          | Community ([@pires](https://github.com/pires), [@AntonioMeireles](https://github.com/AntonioMeireles))
Bare-metal (Offline) | CoreOS       | CoreOS | flannel     | [docs](coreos/bare_metal_offline.md)              |          | Community ([@jeffbean](https://github.com/jeffbean))
Bare-metal           | CoreOS       | CoreOS | Calico      | [docs](coreos/bare_metal_calico.md)               |          | Community ([@caseydavenport](https://github.com/caseydavenport))
CloudStack           | Ansible      | CoreOS | flannel     | [docs](cloudstack.md)                             |          | Community ([@runseb](https://github.com/runseb))
Vmware               |              | Debian | OVS         | [docs](vsphere.md)                                |          | Community ([@pietern](https://github.com/pietern))
Bare-metal           | custom       | CentOS | _none_      | [docs](centos/centos_manual_config.md)            |          | Community ([@coolsvap](https://github.com/coolsvap))
AWS                  | Juju         | Ubuntu | flannel     | [docs](juju.md)                                   |          | [Community](https://github.com/whitmo/bundle-kubernetes) ( [@whit](https://github.com/whitmo), [@matt](https://github.com/mbruzek), [@chuck](https://github.com/chuckbutler) )
OpenStack/HPCloud    | Juju         | Ubuntu | flannel     | [docs](juju.md)                                   |          | [Community](https://github.com/whitmo/bundle-kubernetes) ( [@whit](https://github.com/whitmo), [@matt](https://github.com/mbruzek), [@chuck](https://github.com/chuckbutler) )
Joyent               | Juju         | Ubuntu | flannel     | [docs](juju.md)                                   |          | [Community](https://github.com/whitmo/bundle-kubernetes) ( [@whit](https://github.com/whitmo), [@matt](https://github.com/mbruzek), [@chuck](https://github.com/chuckbutler) )
AWS                  | Saltstack    | Ubuntu | OVS         | [docs](aws.md)                                    |          | Community ([@justinsb](https://github.com/justinsb))
Azure                | Saltstack    | Ubuntu | OpenVPN     | [docs](azure.md)                                  |          | Community
Bare-metal           | custom       | Ubuntu | Calico      | [docs](ubuntu-calico.md)                          |          | Community ([@djosborne](https://github.com/djosborne))
Bare-metal           | custom       | Ubuntu | flannel     | [docs](ubuntu.md)                                 |          | Community ([@resouer](https://github.com/resouer), [@WIZARD-CXY](https://github.com/WIZARD-CXY))
Local                |              |        | _none_      | [docs](locally.md)                                |          | Community ([@preillyme](https://github.com/preillyme))
libvirt/KVM          | CoreOS       | CoreOS | libvirt/KVM | [docs](libvirt-coreos.md)                         |          | Community ([@lhuard1A](https://github.com/lhuard1A))
oVirt                |              |        |             | [docs](ovirt.md)                                  |          | Community ([@simon3z](https://github.com/simon3z))
Rackspace            | CoreOS       | CoreOS | flannel     | [docs](rackspace.md)                              |          | Community ([@doublerr](https://github.com/doublerr))
any                  | any          | any    | any         | [docs](scratch.md)                                |          | Community ([@erictune](https://github.com/erictune))


*注意*：以上表格按照支持级别和测试及使用的版本进行排序。

表格中列说明：

  - **IaaS Provider** 是指提供Kubernetes运行环境的虚拟机或物理机（节点）资源的提供商。
  - **OS** 是指节点上运行的基础操作系统。
  - **Config. Mgmt** 是指节点上安装和管理Kubernetes软件的的配置管理系统。
  - **Networking** 是指实现[网络模型](../../docs/admin/networking.md)的软件。 _none_ 表示只支持一个节点，或支持单物理节点上的虚拟机节点。
  - **Conformance** 表示使用该种配置创建的集群是否通过了项目一致性测试，支持Kubernetes v1.0.0的API和基本特性。
  - Support Levels（支持级别）
    - **Project**：Kubernetes贡献者们经常使用该配置，所以通常最新的版本可使用。
    - **Commercial**：某些厂商负责在自己的平台支持。
    - **Community**：在社区中有活跃支持，但可能最新版本不适用。
    - **Inactive**: 对于初次使用Kubernetes的用户不推荐，并且有可能在将来被移除。
  - **Notes** 说明，比如适用的Kubernetes版本。


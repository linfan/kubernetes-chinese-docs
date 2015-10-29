# 创建Kubernetes集群

Kubernetes可以在多种平台运行，从笔记本电脑，到云服务商的虚拟机，再到机架上的裸机服务器。要创建一个Kubernetes集群，根据不同场景需要做的也不尽相同，可能是运行一条命令，也可能是配置自己的定制集群。这里我们将引导你根据自己的需要选择合适的解决方案。

## 选择正确的解决方案

如果你只是想试一试Kubernetes，我们推荐[基于Docker的本地](docker.md)方案。

基于Docker的本地方案是众多能够完成快速搭建的[本地集群](#本地集群解决方案)方案中的一种，但是局限于单台机器。

当你准备好扩展到多台机器和更高可用性时，[托管](#托管方案)解决方案是最容易搭建和维护的。

[全套云端方案](#全套云端解决方案) 只需要少数几个命令就可以在更多的云服务提供商搭建Kubernetes。

[Custom solutions](#custom-solutions) require more effort to setup but cover and even
they vary from step-by-step instructions to general advice for setting up
a Kubernetes cluster from scratch.

[自定制方案](#自定制解决方案) 需要花费更多的精力，但是覆盖了从零开始搭建Kubernetes集群的通用建议到分步骤的细节指引。

### 本地集群解决方案

Local-machine solutions create a single cluster with one or more Kubernetes nodes on a single
physical machine.  Setup is completely automated and doesn't require a cloud provider account.
But their size and availability is limited to that of a single machine.

The local-machine solutions are:
  - [Local Docker-based](docker.md) (recommended starting point)
  - [Vagrant](vagrant.md) (works on any platform with Vagrant: Linux, MacOS, or Windows.)
  - [No-VM local cluster](locally.md) (Linux only)


### Hosted Solutions

[Google Container Engine](https://cloud.google.com/container-engine) offers managed Kubernetes
clusters.

### Turn-key Cloud Solutions

These solutions allow you to create Kubernetes clusters on a range of Cloud IaaS providers with only a
few commands, and have active community support.
- [GCE](gce.md)
- [AWS](aws.md)
- [Azure](coreos/azure/README.md)

### Custom Solutions

Kubernetes can run on a wide range of Cloud providers and bare-metal environments, and with many
base operating systems.

If you can find a guide below that matches your needs, use it.  It may be a little out of date, but
it will be easier than starting from scratch.  If you do want to start from scratch because you
have special requirements or just because you want to understand what is underneath a Kubernetes
cluster, try the [Getting Started from Scratch](scratch.md) guide.

If you are interested in supporting Kubernetes on a new platform, check out our [advice for
writing a new solution](../../docs/devel/writing-a-getting-started-guide.md).

#### Cloud

These solutions are combinations of cloud provider and OS not covered by the above solutions.
- [AWS + coreos](coreos.md)
- [GCE + CoreOS](coreos.md)
- [AWS + Ubuntu](juju.md)
- [Joyent + Ubuntu](juju.md)
- [Rackspace + CoreOS](rackspace.md)

#### On-Premises VMs

- [Vagrant](coreos.md) (uses CoreOS and flannel)
- [CloudStack](cloudstack.md) (uses Ansible, CoreOS and flannel)
- [Vmware](vsphere.md)  (uses Debian)
- [juju.md](juju.md) (uses Juju, Ubuntu and flannel)
- [Vmware](coreos.md)  (uses CoreOS and flannel)
- [libvirt-coreos.md](libvirt-coreos.md)  (uses CoreOS)
- [oVirt](ovirt.md)
- [libvirt](fedora/flannel_multi_node_cluster.md) (uses Fedora and flannel)
- [KVM](fedora/flannel_multi_node_cluster.md)  (uses Fedora and flannel)

#### Bare Metal

- [Offline](coreos/bare_metal_offline.md) (no internet required.  Uses CoreOS and Flannel)
- [fedora/fedora_ansible_config.md](fedora/fedora_ansible_config.md)
- [Fedora single node](fedora/fedora_manual_config.md)
- [Fedora multi node](fedora/flannel_multi_node_cluster.md)
- [Centos](centos/centos_manual_config.md)
- [Ubuntu](ubuntu.md)
- [Docker Multi Node](docker-multinode.md)

#### Integrations

- [Kubernetes on Mesos](mesos.md) (Uses GCE)

## Table of Solutions

Here are all the solutions mentioned above in table form.

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


*Note*: The above table is ordered by version test/used in notes followed by support level.

Definition of columns:

  - **IaaS Provider** is who/what provides the virtual or physical machines (nodes) that Kubernetes runs on.
  - **OS** is the base operating system of the nodes.
  - **Config. Mgmt** is the configuration management system that helps install and maintain Kubernetes software on the
    nodes.
  - **Networking** is what implements the [networking model](../../docs/admin/networking.md).  Those with networking type
    _none_ may not support more than one node, or may support multiple VM nodes only in the same physical node.
  - **Conformance** indicates whether a cluster created with this configuration has passed the project's conformance
    tests for supporting the API and base features of Kubernetes v1.0.0.
  - Support Levels
    - **Project**:  Kubernetes Committers regularly use this configuration, so it usually works with the latest release
      of Kubernetes.
    - **Commercial**: A commercial offering with its own support arrangements.
    - **Community**: Actively supported by community contributions. May not work with more recent releases of Kubernetes.
    - **Inactive**: No active maintainer.  Not recommended for first-time Kubernetes users, and may be deleted soon.
  - **Notes** is relevant information such as the version of Kubernetes used.


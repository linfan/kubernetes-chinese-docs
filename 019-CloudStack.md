# CloudStack的入门指南
------------------------------------------------------------

**内容列表**

- [介绍](#introduction)
- [先决条件](#prerequisites)
- [克隆脚本](#clone-the-playbook)
- [创建一个 Kubernetes 集群](#create-a-kubernetes-cluster)

### Introduction

[CloudStack](http://cloudstack.apache.org)是一个用于构建基于硬件虚拟化的公有云和私有云（传统IaaS）的软件。在 CloudStack 上部署 Kubernetes 有好几种方法，需要根据 CloudStack 所使用的哪种云和有哪些可用镜像来决定。 例如[ Exoscale ](http://exoscale.ch)就提供了一个[ coreOS ](http://coreos.com)的可用模版，因此也可以使用在 coreOS 部署 Kubernetes 的指令来部署。 CloudStack 同样也提供了一个 Vagrant 插件，因此也可以用 Vagrant 来部署 Kubernetes ，既可以选择原有的基于 shell 脚本的部署方式，也可以选择新的基于 Salt 的部署方式。

CloudStack的[CoreOS](http://coreos.com)模版会[每日](http://stable.release.core-os.net/amd64-usr/current/)构建。 在执行安装 Kubernetes 部署指令之前需要先将模版[注册](http://docs.cloudstack.apache.org/projects/cloudstack-administration/en/latest/templates.html)到云上。

本指引使用了[Ansible playbook](https://github.com/runseb/ansible-kubernetes)。
完全自动化构建，单个 Kubernetes 部署脚本基于 coreOS [指令](coreos/coreos_multinode_cluster.md)构建。


 [Ansible](http://ansibleworks.com) 脚本基于 coreOS 镜像将 Kubernetes 部署到 CloudStack 基础云上。 该脚本创建一个SSH密钥对、一个安全组和相关规则并最终通过云初始化配置来启动coreOS实例。

### 先决条件

    $ sudo apt-get install -y python-pip
    $ sudo pip install ansible
    $ sudo pip install cs

[_cs_](https://github.com/exoscale/cs) 是一个 CloudStack API 的 python 模块。

可以通过使用API密钥和HTTP的方式来配置CloudStack终端。

你可以将它们定义成环境变量: `CLOUDSTACK_ENDPOINT`, `CLOUDSTACK_KEY`, `CLOUDSTACK_SECRET` 和 `CLOUDSTACK_METHOD`.

或者通过创建 `~/.cloudstack.ini`文件的方式：

    [cloudstack]
    endpoint = <your cloudstack api endpoint>
    key = <your api access key>
    secret = <your api secret key>
    method = post

我们需要使用 http POST 请求来将 _large_ 用户的数据上传到各个coreOS实例。

### 克隆脚本

    $ git clone --recursive https://github.com/runseb/ansible-kubernetes.git
    $ cd ansible-kubernetes

[ansible-cloudstack](https://github.com/resmo/ansible-cloudstack) 模块被设置成了该仓库的一个子模块，因此需要使用`--recursive`。
### 创建一个 Kubernetes 集群

你只需要简单运行如下脚本。

    $ ansible-playbook k8s.yml

编辑`k8s.yml`文件中的一些变量。

    vars:
      ssh_key: k8s
      k8s_num_nodes: 2
      k8s_security_group_name: k8s
      k8s_node_prefix: k8s2
      k8s_template: Linux CoreOS alpha 435 64-bit 10GB Disk
      k8s_instance_type: Tiny

它将启动一个 Kubernetes 主机和一些计算节点（默认为2个）。`instance_type`和`template`默认指定的是 [exoscale](http://exoscale.ch)，编辑这两个参数来指定你CloudStack云的模板和实例类型（即服务提供商）。

如果你想修改一些其他参数的话，请参照`roles/k8s`里面的任务和模版。

一旦脚本执行完成，命令行会打印出Kubernetes主机的IP地址：

    TASK: [k8s | debug msg='k8s master IP is {{ k8s_master.default_ip }}'] ********

使用 _core_ 用户和刚创建的密钥通过ssh登录到主机，你可以列出集群中的所有机器：

    $ ssh -i ~/.ssh/id_rsa_k8s core@<master IP>
    $ fleetctl list-machines
    MACHINE		IP		       METADATA
    a017c422...	<node #1 IP>   role=node
    ad13bf84...	<master IP>	   role=master
    e9af8293...	<node #2 IP>   role=node

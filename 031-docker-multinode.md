# 利用Docker安装多节点Kubernetes
`译者：王乐` `校对：无`


_注意_:
这个介绍一定程度上会比[single node](docker.md)里的介绍要更进一步。如果你有兴趣探索Kubernetes，我们建议你从这里开始。
_注意_:
Docker 1.7.0里的一个[bug](https://github.com/docker/docker/issues/14106)影响Docker上多节点的正常安装。
请安装Docker版本1.6.2或1.7.1.

## 前提条件

1. 你需要你一台安装有正确版本的Docker的主机。

## 概括介绍

本指南会指导架设有2个节点的Kubernetes集群。其中包括一个支持API服务器和编排工作的_主节点_，和一个从主节点接受任务的_从节点_。你可以按照同样的步骤添加任意数量的从节点，从而假设一个庞大的集群。

这里的图标展示了最终的结果：
![Kubernetes Single Node on Docker](k8s-docker.png)

### 引导启动Docker

本指南同样运行两个Docker后台程序实例的模式
 1) 一个_引导启动_的Docker容器实例用来运行例如`flanneld`和`etcd`系统后台程序。
 2) 一个_主_Docker容器实例来服务Kubernetes和用户容器。

这个模式是有必要的，因为`flannel`的后台程序是负责建立和管理Kubernetes新建的Docker容器间的相互通信。所以`flannel`必须要运行在_主_Docker后台程序之外。为了利用容器来方便部署和管理，我们使用了这个较简单的_引导启动_的Docker后台程序来实现这一点。

在安装前你可以在每个节点上选择k8s的版本：
```
export K8S_VERSION=<your_k8s_version (e.g. 1.0.3)>
```

否则, 我们会使用最新的`hyperkube`镜像当作默认k8s版本。
## 主节点
第一步是初始化主节点。
复制Kubernetes在Github上的repo，并在主节点的主机上以root身份运行脚本[master.sh](docker-multinode/master.sh):

```sh
cd kubernetes/docs/getting-started-guides/docker-multinode/
./master.sh
```

`Master done!`

参考[这里](docker-multinode/master.md) for detailed instructions explanation.

## 添加从节点
当主节点正常运行后，你可以在不同的主机上添加更多的从节点。

复制Kubernetes在Github上的repo，并在从节点的主机上以root身份运行脚本[worker.sh](docker-multinode/worker.sh):

```sh
export MASTER_IP=<your_master_ip (e.g. 1.2.3.4)>
cd kubernetes/docs/getting-started-guides/docker-multinode/
./worker.sh
```

`Worker done!`

参考[这里](docker-multinode/worker.md) for detailed instructions explanation.

## 部署DNS

参考[这里](docker-multinode/deployDNS.md) for instructions.

## 测试你的集群
一旦你的集群创建完毕，你可以进行[测试](docker-multinode/testing.md)

请参见[examples directory](../../examples/)里更多的完整应用。
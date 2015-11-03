
# 从Ferdora入门Kubernetes

本节内容
* 前提条件
* 说明

## 前提条件

1.你需要2台或以上安装有Fedora的机器

## 说明

本文是针对Fedora系统的Kubernetes入门教程。通过手动配置，你将会理解所有底层的包、服务、端口等。
本文只会让一个节点（之前称从节点）工作。多节点需要在Kubernetes之外配置一个可用的网络环境，尽管这个额外的配置条件是显而易见的，（本节也不会去配置）。
Kubernetes包提供了一些服务：kube-apiserver, kube-scheduler, kube-controller-manager, kubelet, kube-proxy。这些服务通过systemd进行管理，配置信息都集中存放在一个地方：/etc/kubernetes。我们将会把这些服务运行到不同的主机上。第一台主机，fed-master，将是Kubernetes 的master主机。这台机器上将运行kube-apiserver, kube-controller-manager和kube-scheduler这几个服务，此外，master主机上还将运行etcd（如果etcd运行在另一台主机上，那就不需要了，本文假设etcd和Kubernetes master运行在同一台主机上）。其余的主机，fed-node，将是从节点，上面运行kubelet, proxy和docker。




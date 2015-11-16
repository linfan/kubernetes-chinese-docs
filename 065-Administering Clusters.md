# Kubernetes集群管理指南：集群组件

本文概述了各种二进制组件，这些组件运行可以提供一个有效的Kubernetes集群。

## 主组件
主组件可以提供集群的控制面板。例如，主组件负责对有关集群做全局性的决策（如，调度），以及探测和对集群事件的相应（如，当Replication Controller的‘replicas’字段不满足时，启动一个新的Pod）。

理论上，主组件可以运行在集群的任何节点上。但是，为了简单化，当前设置的脚本通常在同样的VM上启动所有的主组件，而且不会在该VM上运行用户容器。参考[high-availability.md](083-High Availability Clusters.md)，有关Multi-Master-VM设置的实例。

即便在未来，如果Kubernetes完全自托管，将只允许主组件调度节点的子集，以限制具有用户Pod的共同运行，减少节点损害安全漏洞的可能范围。

### Kube-apiserver
[Kube-apiserver](066-kube-apiserver Binary.md)展示了Kubernetes API；它是Kubernetes控制面的前端，可以横向扩展（i.e., one scales it by running more of them-- [high-availability.md](083-High Availability Clusters.md)）。

### Etcd
Etcd作为Kubernetes的后备存储。所有的集群数据都存储在这里。一个Kubernetes集群的适当管理包括对ETCD的数据备份计划。

### Kube-Controller-Manager
Kube-Controller-Manager是运行控制器的一个二进制文件，处理集群中日常任务的后端进程。 逻辑上，每个控制器是独立的进程，但是为了减少系统中移动片（moving pieces）的数量，他们都被编译成一个独立的二进制，并且运行在一个单一的进程中。
这些控制器包括：
	Node Controller – 节点控制器
		当节点异常时，负责查看和响应。
	Replication Controller – 复制控制器
		负责对系统中的每一个控制器对象，保持Pod的正确值。
	Endpoints Controller – 端点控制器
		填充端点对象（即加入Service & Pod）
	Service Account & Token Controllers （服务账户 & ）
		为新的Namespace创建默认账户和API接入Token。
	…等等。

### Kube-scheduler

KUBE-调度观察新创建、还没有分配节点的Pod，并且从中选择一个节点运行。

### Addons
Addons是实现了集群功能的Pod和服务。它们不能在主VM上运行，但是目前能够调用API创建这些Pod和服务的默认启动脚本可以在主VM上运行。可参考kube-master-addons
Addon对象在“kube-system”空间被创建。
Addons实例：
	DNS提供集群本地DNS。
	Kube-ui为集群提供图形用户界面。
	Fluentd-elasticsearch提供日志存储。参考gcp version。
	Cluster-monitoring监控集群。

## Node components 节点组件

节点组件运行在每一个节点上，维护运行Pod，并给它们提供Kubernetes运行环境。

### Kubelet
Kubelet是主节点代理。它，
	监督已经分配到节点的Pod（或者被apiserver分配，或者通过本地配置文件）：
		挂载Pod需要的空间
		下载Pod的秘钥
		通过Docker（或，experimentally，rkt）运行Pod的容器
		定期执行任何请求的容器活性探针（container liveness probes）
		给系统的其它部分报告Pod的状态，如果有必要，可以创建“mirror pod”
	返回节点的状态给系统的其它部分。

### Kube-proxy

Kube-proxy使得Kubernetes服务抽象化，在主机上维护网络规则，并且执行连接转发。

### Docker

Docker主要用于运行容器。

### Rkt

Rkt，实验证明，可以作为Docker的替代者。

### Monit

Monit是一个轻量级进程维护系统，可以保持Kubelet和Docker正常运行。
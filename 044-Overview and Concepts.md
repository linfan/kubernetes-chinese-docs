#Kubernetes用户指南：应用程序管理

Table of Contents

- [Kubernetes用户指南：应用程序管理](#Kubernetes用户指南：应用程序管理)
 - [实战入门](#实战入门)
 - [进阶路线](#进阶路线)
 - [概念指南](#概念指南)
 - [阅读延伸](#阅读延伸)



>用户指南旨在为了每一个想运行程序或者服务在已有Kubernetes集群中的人。Kubernetes集群的安装和管理相关内容在[集群管理指南](http://kubernetes.io/v1.0/docs/admin/README.html)中可以找到。[开发者指南](http://kubernetes.io/v1.0/docs/devel/README.html)是为了那些想写代码直接访问Kubernetes的API,或者为Kubernetes项目共享代码的每一个人。

>请确认你已经完成这个事列： [运行用户指南中实例的先决条件](http://kubernetes.io/v1.0/docs/user-guide/prereqs.html).

##实战入门

* >Kubernetes 101
* >Kubernetes 201

##进阶路线图

如果你对Kubernetes布什很熟悉的话, 我们建议你按顺序阅读下面的部分:

* 1.快速入门: 运行并展示一个应用程序
* 2.容器的配置与启动: 配置容器的公共参数
* 3.部署可持续运行的容器
* 4.链接应用程序: 将应用程序展示给客户和用户
* 5.在生产环境中是用容器
* 6.部署管理
* 7.应用程序的自我检查和调试
    * 1.使用Kubernetes的Web用户界面
    * 2.日志
    * 3.监控
    * 4.通过exec命令进入容器
    * 5.通过代理连接容器
    * 6.通过接口转发连接容器

##概念指南

>概论 : Kubernetes中概念的简要概述

>Cluster : 集群是指由Kubernetes使用一系列的物理机、虚拟机和其他基础资源来运行你的应用程序。

>Node : 一个node就是一个运行着Kubernetes的物理机或虚拟机，并且pod可以在其上面被调度。.

>Pod : 一个pod对应一个由相关容器和卷组成的容器组 

>Label : A label is a key/value pair that is attached to a resource, such as a pod, to convey a user-defined identifying attribute. Labels can be used to organize and to select subsets of resources.

>Selector : A selector is an expression that matches labels in order to identify related resources, such as which pods are targeted by a load-balanced service.

>Replication Controller : A replication controller ensures that a specified number of pod replicas are running at any one time. It both allows for easy scaling of replicated systems and handles re-creation of a pod when the machine it is on reboots or otherwise fails.

>Service : A service defines a set of pods and a means by which to access them, such as single stable IP address and corresponding DNS name.

>Volume : A volume is a directory, possibly with some data in it, which is accessible to a Container as part of its filesystem. Kubernetes volumes build upon Docker Volumes, adding provisioning of the volume directory and/or device.

>Secret : A secret stores sensitive data, such as authentication tokens, which can be made available to containers upon request.

>Name : A user- or client-provided name for a resource.

>Namespace : A namespace is like a prefix to the name of a resource. Namespaces help different projects, teams, or customers to share a cluster, such as by preventing name collisions between unrelated teams.

>Annotation : A key/value pair that can hold larger (compared to a label), and possibly not human-readable, data, intended to store non-identifying auxiliary data, especially data manipulated by tools and system extensions. Efficient filtering by annotation values is not supported.

##阅读延伸

###API resources

Working with resources

###Pods and containers

>Pod lifecycle and restart policies
Lifecycle hooks
Compute resources, such as cpu and memory
Specifying commands and requesting capabilities
Downward API: accessing system configuration from a pod
Images and registries
Migrating from docker-cli to kubectl
Tips and tricks when working with config
Assign pods to selected nodes
Perform a rolling update on a running group of pods
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

* 1.快速入门: 运行并展示一个应用
* 2.Configuring and launching containers: configuring common     container parameters
* 3.Deploying continuously running applications
* 4.Connecting applications: exposing applications to clients and users
* 5.Working with containers in production
* 6.Managing deployments
* 7.Application introspection and debugging
    * 1.Using the Kubernetes web user interface
    * 2.Logging
    * 3.Monitoring
    * 4.Getting into containers via exec
    * 5.Connecting to containers via proxies
    * 6.Connecting to containers via port forwarding

##概念指南

>Overview : A brief overview of Kubernetes concepts.

>Cluster : A cluster is a set of physical or virtual machines and other infrastructure resources used by Kubernetes to run your applications.

>Node : A node is a physical or virtual machine running Kubernetes, onto which pods can be scheduled.

>Pod : A pod is a co-located group of containers and volumes.

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
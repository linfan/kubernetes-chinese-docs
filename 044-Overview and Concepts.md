#Kubernetes用户指南：应用程序管理

目录

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

>Label : 一个label是一个被附加到资源上的键/值对，譬如附加到一个Pod上，为它传递一个用户自定的并且可识别的属性.Label还可以被应用来组织和选择子网中的资源

>selector是一个通过匹配labels来定义资源之间关系得表达式，例如为一个负载均衡的service指定所目标Pod.

>Replication Controller : replication controller 是为了保证一定数量被指定的Pod的复制品在任何时间都能正常工作.它不仅允许复制的系统易于扩展，还会处理当pod在机器在重启或发生故障的时候再次创建一个

>Service : 一个service定义了访问pod的方式，就像单个固定的IP地址和与其相对应的DNS名之间的关系。

>Volume: 一个volume是一个目录，可能会被容器作为未见系统的一部分来访问。Kubernetes volume 构建在Docker Volumes之上,并且支持添加和配置volume目录或者其他存储设备。

>Secret : Secret 存储了敏感数据，例如能允许容器接收请求的权限令牌。

>Name :  用户为Kubernetes中资源定义的名字

>Namespace : Namespace 好比一个资源名字的前缀。它帮助不同的项目、团队或是客户可以共享cluster,例如防止相互独立的团队间出现命名冲突

>Annotation : 相对于label来说可以容纳更大的键值对，它对我们来说可能是不可读的数据，只是为了存储不可识别的辅助数据，尤其是一些被工具或系统扩展用来操作的数据

##阅读延伸

###API resources

>[更多可使用的API资源](http://kubernetes.io/v1.0/docs/user-guide/working-with-resources.html)

###Pods and containers

>- [Pod 的生命周期和重启策略](http://kubernetes.io/v1.0/docs/user-guide/pod-states.html)
- [生命周期事件钩子](http://kubernetes.io/v1.0/docs/user-guide/container-environment.html)
- [计算资源，例如cpu和内存](http://kubernetes.io/v1.0/docs/user-guide/compute-resources.html)
- [指定命令和请求的能力](http://kubernetes.io/v1.0/docs/user-guide/containers.html)
- [Downward API（用于向下支持容器访问集群的API,如Docker）: 从pod中访问系统配置](http://kubernetes.io/v1.0/docs/user-guide/downward-api.html)
- [镜像和仓库](http://kubernetes.io/v1.0/docs/user-guide/images.html)
- [从使用docker-cli转向kubectl](http://kubernetes.io/v1.0/docs/user-guide/docker-cli-to-kubectl.html)
- [配置集群时的提示和技巧](http://kubernetes.io/v1.0/docs/user-guide/config-best-practices.html)
- [把pod分配到指定node](http://kubernetes.io/v1.0/docs/user-guide/node-selection/)
- [展示为一组正在运行的pod左滚动更新](http://kubernetes.io/v1.0/docs/user-guide/update-demo/)
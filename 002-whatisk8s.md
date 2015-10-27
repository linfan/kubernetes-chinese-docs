
# 什么是Kubernetes？

Kubernetes一个用于容器集群的自动化部署、扩容以及运维的开源平台。

使用Kubernetes，你可以快速高效地响应客户需求：

 - 动态地对应用进行扩容。
 - 无缝地发布新特性。
 - 通过仅使用需要的资源来优化硬件使用。

我们希望培育出一个组件和工具的生态，帮助大家减轻在公有云及私有云上运行应用的负担。

#### Kubernetes是：

* **简洁的**：轻量级，简单，易上手
* **可移植的**：公有，私有，混合，多重云（multi-cloud）
* **可扩展的**: 模块化, 插件化, 可挂载, 可编排
* **可自愈的**: 自动布置, 自动重启, 自动复制

Kubernetes项目是Google在2014年启动的。Kubernetes构建在[Google公司十几年的大规模高负载生产系统运维经验](https://research.google.com/pubs/pub43438.html)之上，同时结合了社区中各项最佳设计和实践。

##### 准备好[开始](003-getting-started-guides.md)了吗？

<hr>

想了解为何要使用[容器](http://aucouranton.com/2014/06/13/linux-containers-parallels-lxc-openvz-docker-and-more/)技术？

下面是一些关键点:

* **以应用程序为中心的管理**：
    将抽象级别从在虚拟硬件上运行操作系统上升到了在使用特定逻辑资源的操作系统上运行应用程序。这在提供了Paas的简洁性的同时拥有IssS的灵活性，并且相对于运行[12-factor应用程序](http://12factor.net/)有过之而无不及。
* **开发和运维的关注点分离**:
    提供构建和部署的分离；这样也就将应用从基础设施中解耦。
* **敏捷的应用创建和部署**:
    相对使用虚拟机镜像，容器镜像的创建更加轻巧高效。
* **持续开发，持续集成以及持续部署**:
    提供频繁可靠地构建和部署容器镜像的能力，同时可以快速简单地回滚(因为镜像是固化的)。
* **松耦合，分布式，弹性，自由的[微服务](http://martinfowler.com/articles/microservices.html)**:
    应用被分割为若干独立的小型程序，可以被动态地部署和管理 -- 而不是一个运行在单机上的超级臃肿的大程序。
* **开发，测试，生产环境保持高度一致**:
    无论是再笔记本电脑还是服务器上，都采用相同方式运行。
* **兼容不同的云平台或操作系统上**:
    可运行与Ubuntu，RHEL，on-prem或者Google Container Engine，覆盖了开发，测试和生产的各种不同环境。
* **资源分离**:
    带来可预测的程序性能。
* **资源利用**:
    高性能，大容量。

#### Kubernetes不是：

Kubernetes不是PaaS（平台即服务）。

* Kubernetes does not limit the types of applications supported. It does not dictate application frameworks, restrict the set of supported language runtimes, nor cater to only [12-factor applications](http://12factor.net/). Kubernetes aims to support an extremely diverse variety of workloads: if an application can run in a container, it should run great on Kubernetes.
* Kubernetes is unopinionated in the source-to-image space. It does not build your application. CI workflow is an area where different users and projects have their own requirements and preferences, so we support layering CI workflows on Kubernetes but don't dictate how it should work.
* On the other hand, a number of PaaS systems run *on* Kubernetes, such as [Openshift](https://github.com/openshift/origin) and [Deis](http://deis.io/). You could also roll your own custom PaaS, integrate with a CI system of your choice, or get along just fine with just Kubernetes: bring your container images and deploy them on Kubernetes.
* Since Kubernetes operates at the application level rather than at just the hardware level, it provides some generally applicable features common to PaaS offerings, such as deployment, scaling, load balancing, logging, monitoring, etc. However, Kubernetes is not monolithic, and these default solutions are optional and pluggable.

Kubernetes并不是单单的"编排系统"；它排除了对编排的需要:

* The technical definition of "orchestration" is execution of a defined workflow: do A, then B, then C. In contrast, Kubernetes is comprised of a set of control processes that continuously drive current state towards the provided desired state. It shouldn't matter how you get from A to C: make it so. This results in a system that is easier to use and more powerful, robust, and resilient.


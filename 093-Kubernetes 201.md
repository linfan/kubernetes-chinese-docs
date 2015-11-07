# **Kubernetes 201 - 标签, 副本控制, 服务和健康检查**

如果你浏览过[Kubernetes 101](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/README.html), 你能够学习到kubectl, pods, 卷，多容器的概念。在 Kubernetes 201中，我们将学习201剩下的部分， 包含一些Kubernetes稍微高级的主题, 讨论关于应用的生产环境化, 部署以及扩展。

为了让kubectl操作的示例可以正常工作，确保本地存在示例的目录, 从[发布](https://github.com/GoogleCloudPlatform/kubernetes/releases)或者是[源](https://github.com/GoogleCloudPlatform/kubernetes)获取。

内容列表

* [Kubernetes 201 - 标签, 副本控制, 服务和健康检查](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#kubernetes-201---labels-replication-controllers-services-and-health-checking)
   * [标签](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#labels)
   * [副本控制器](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#replication-controllers)
     * [副本控制器管理](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#replication-controller-management)
   * [服务](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#services)
     * [服务管理](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#service-management)
   * [健康检查](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#health-checking)
     * [进程健康检查](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#process-health-checking)
     * [应用程序健康检查](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#application-health-checking)
   * [下面是什么？](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#whats-next)
  
## **标签**

已经学习了Pods以及如何创建他们, 你可能会在紧急的情况下创建许多，许多pods。请做！但是最终你需要一个系统来通过组管理这些pods。为了实现这个功能，在Kubernetes系统中使用标签。标签是一个键值对，在Kubernetes中会标记到每一个对象上。标签选择器将RESTful `list`请求传递给apiserver来获取和标签选择器匹配的对象列表。

To add a label, add a labels section under metadata in the pod definition:
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

增加一个标签，在pod定义文件中的元数据下增加一个标签部分：

```json
labels:
    app: nginx
```

例如，下面是一个带标签的nginx pod定义[pod-nginx-with-label.yaml](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/pod-nginx-with-label.yaml):

```json
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

创建一个标签过的pod [pod-nginx-with-label.yaml](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/pod-nginx-with-label.yaml):

```bash
$ kubectl create -f docs/user-guide/walkthrough/pod-nginx-with-label.yaml
```

列出所有标签`app=nginx`为nginx的pod：

```bash
$ kubectl get pods -l app=nginx
```

更多信息，参考[标签](http://kubernetes.io/v1.0/docs/user-guide/labels.html)。其他两个Kubernetes构建部分：副本控制和服务，使用标签作为其核心概念。

## **副本控制器**

好的，现在你已经知道如何创建许多，多容器，标签化的pods, 使用他们来构建应用程序， 你也许会很想开始构建一堆完整的独立pods,但如果你这样做，整个主机的运营问题摆在了眼前。例如：如何做到让pods的数量扩展，增加或者减少， 如何保证所有的pods是同质化的？

副本控制器对象可以给出这些问题的答案。Replication controllers are the objects to answer these questions.一个副本控制器可以将创建一个pod模板（一个饼干切割机如果你会的话）以及需要的一定数量的副本到单个Kubernetes对象。副本控制器也包含一个标签选择器来识别由副本控制器管理的对象集合。副本控制器通过不断的测量这个集合对象申请的容量大小来采取创建或者删除pods的动作。

例如，下面是用副本控制器来初始化两个nginx pods[replication-controller.yaml)](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/replication-controller.yaml):

### **副本控制器管理**

创建nginx副本控制器[replication-controller.yaml](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/replication-controller.yaml):

```bash
$ kubectl create -f docs/user-guide/walkthrough/replication-controller.yaml
```

列出所有副本控制器：

```bash
$ kubectl get rc
```

通过名称删除副本控制器：

```bash
$ kubectl delete rc nginx-controller
```

更多信息，请参考[副本控制器](http://kubernetes.io/v1.0/docs/user-guide/replication-controller.html)。

## **服务**

一旦你拥有一个pods的副本集合，你需要在能够在应用程序层之间提供连接的抽象。例如，如果你已经有个一个副本控制器来管理后端任务，当你需要重新扩展你的后端应用的时候，你不需要重新配置你的前端应用。Likewise, if the pods in your backends are scheduled (or rescheduled) onto different machines, you can't be required to re-configure your front-ends. In Kubernetes, the service abstraction achieves these goals. A service provides a way to refer to a set of pods (selected by labels) with a single static IP address. It may also provide load balancing, if supported by the provider.

For example, here is a service that balances across the pods created in the previous nginx replication controller example (service.yaml):
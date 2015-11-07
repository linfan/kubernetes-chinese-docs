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

一旦你拥有一个pods的副本集合，你需要在能够在应用程序层之间提供连接的抽象。例如，如果你已经有个一个副本控制器来管理后端任务，当你需要重新扩展你的后端应用的时候，你不需要重新配置你的前端应用。同样，如果后端的pods被调度（或者重新调度）到不同的机器上，你也不需要重新配置前端应用。在Kubernetes中,对服务的抽象能够达到这个目标。一个服务会提供一个指向pods集合（通过标签选择）的IP地址。如果支持的话，它同时能够提供负载均衡功能。

例如，这里有一个在之前例子中通过nginx副本控制器创建的pods之间提供负载均衡功能的服务[service.yaml](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/service.yaml):

```json
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  ports:
  - port: 8000 # the port that this service should serve on
    # the container on each pod to connect to, can be a name
    # (e.g. 'www') or a number (e.g. 80)
    targetPort: 80
    protocol: TCP
  # just like the selector in the replication controller,
  # but this time it identifies the set of pods to load balance
  # traffic to.
  selector:
    app: nginx
```

### **服务管理**

创建一个nginx服务[service.yaml](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/service.yaml)：

```bash
$ kubectl create -f docs/user-guide/walkthrough/service.yaml
```

列出所有的服务：

```bash
$ kubectl get services
```

对于大多数供应商，服务的IP地址外部无法访问。测试服务可以访问的最简单的方式为创建一个busybox pod在上面远程执行命令。详细内容查看[命令执行文档](http://kubernetes.io/v1.0/docs/user-guide/kubectl/kubectl_exec.html).

一旦提供的服务IP地址可以访问，你便可以通过80端口使用curl命令来访问http终端节点：

```bash
$ export SERVICE_IP=$(kubectl get service nginx-service -o=template -t={{.spec.clusterIP}})
$ export SERVICE_PORT=$(kubectl get service nginx-service -o=template '-t={{(index .spec.ports 0).port}}')
$ curl http://${SERVICE_IP}:${SERVICE_PORT}
```

通过名称删除服务：
```bash
$ kubectl delete service nginx-controller
```

一旦服务被创建，就会分配一个唯一的IP地址。这个地址同服务的生命周期绑定，服务存活期间不会发生变化。通过配置Pods来和服务通信，并且能够同一些自动被负载均衡的pods进行通信，这些服务中的pod是由标签选择器识别出来的集合中的一员。

更多信息，查看[服务](http://kubernetes.io/v1.0/docs/user-guide/services.html)。

## **健康检查**

我写的代码，永远不会崩溃，对不对？不幸的是，Kubernetes问题列表中另有说明...

一个更好的方法是使用管理系统来进行定期的应用程序健康检查和修复工作，而不是尝试编写无bug的代码。你的应用程序之外本身有一套监控系统负责监控和修复工作。很重要的一点是这个系统必须放置在应用程序外部，应用程序一旦失败以及健康检查代理是应用程序的一部分，这个代理也会失败并且你永远不会知道。在Kubernetes中, 这个健康检查监控代理是Kubelet。

### **进程健康检查**

最简单形式的健康检查是进程级别的健康检查。Kubelet不停的问Docker进程容器进程是不是还在运行，如果不在运行，容器进程就会被重启。至今为止，在你运行的全部Kubernetes示例中，这种健康检查已经开启。所有在Kubernetes中运行的单个容器都存在这种机制。


### **应用程序健康检查**

然而，多数情况下底层健康检查是不够的，例如，考虑下面的代码：

```golang
lockOne := sync.Mutex{}
lockTwo := sync.Mutex{}

go func() {
  lockOne.Lock();
  lockTwo.Lock();
  ...
}()

lockTwo.Lock();
lockOne.Lock();
```

这是计算机科学中典型的[“死锁”](https://en.wikipedia.org/wiki/Deadlock)问题。从Docker的角度看，你的应用仍然在进行操作并且进程仍然在运行，但是从应用程序角度来看，你的代码锁死了，再也会不正确响应了。

为了解决这个问题，Kubernetes支持用户自己实现应用程序健康检查。这些检查通过Kubelet来确保应用程序按照你定义的“正确方式”来操作。

目前，有三种应用程序健康检查机制你可以选择：

* HTTP 检查检查 - Kubelet会调用web钩子。如果返回200到399之间的返回码，代表成功，反之代表失败，在[这里](http://kubernetes.io/v1.0/docs/user-guide/liveness/)查看健康检查示例。
* 容器执行 - Kubelet会在容器里执行一条命令，如果返回值为0认为是成功。在[这里](http://kubernetes.io/v1.0/docs/user-guide/liveness/)查看健康检查示例。
* TCP套接字 - Kubelet将会尝试打开一个套接字连接到容器。如果可以建立连接，认为容器是健康 的，否则认为是失败的。

所有情况下，如果Kubelet一旦发现失败，容器会被重启。

可以在容器配置文件的`livenessProbe`部分配置你的容器的健康检查功能。你也可以指定`initialDelaySeconds`参数，这个参数指的是从容器启动到进行健康检查的宽松期。让你的容器有足够的时间进行任何初始化工作。

这里有一个HTTP健康检查的pod配置示例[pod-with-http-healthcheck.yaml](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/pod-with-http-healthcheck.yaml)：

```json
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-healthcheck
spec:
  containers:
  - name: nginx
    image: nginx
    # defines the health checking
    livenessProbe:
      # an http probe
      httpGet:
        path: /_status/healthz
        port: 80
      # length of time to wait for a pod to initialize
      # after pod startup, before applying health checking
      initialDelaySeconds: 30
      timeoutSeconds: 1
    ports:
    - containerPort: 80
```

更多关于健康检查信息，参考[容器探针](http://kubernetes.io/v1.0/docs/user-guide/pod-states.html#container-probes)
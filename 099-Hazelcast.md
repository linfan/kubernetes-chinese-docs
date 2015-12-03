#Kubernetes在Hazelcast平台上部署原生云应用
`译者：贾澜鹏` `校对：无`


这篇文档主要是描述Kubernetes在[Hazelcast](http://hazelcast.org/)平台上部署原生云应用的方法。当我们提到原生云应用时，意味着我们的应用程序是运行在一个集群之上，同时使用这个集群的基础设施实现这个应用程序。值得注意的是，在此情况下，一个定制化的Hazelcast`引导程序`被用来使Hazelcast可以动态的发现已经加入集群的Hazelcast节点。

当拓扑结构发生变化时，需要Hazelcast节点自身进行交流和处理。

本文档同样也尝试去解释Kubernetes的核心组件：Pods、Service和ReplicationController。

##前提

下面的例子假定你已经安装了Kubernetes集群并且可以运行。同时你也已经在你的路径下安装了kubectl命令行工具。可以从“[开始](http://kubernetes.io/v1.0/docs/getting-started-guides/)”这个小节中得到相应平台的安装指令。

##给急性子的备注

下面的介绍会有点长，如果你想跳过直接开始的话，请看结尾处的“太长不读版”。

##资源

免费的资源如下：

- Hazelcast节点发现 - [https://github.com/pires/hazelcast-kubernetes-bootstrapper](https://github.com/pires/hazelcast-kubernetes-bootstrapper)
- Dockerfile文件 -[https://github.com/pires/hazelcast-kubernetes](https://github.com/pires/hazelcast-kubernetes)
- 官方的Docker镜像 - [https://quay.io/repository/pires/hazelcast-kubernetes](https://quay.io/repository/pires/hazelcast-kubernetes)
 
##简单的单调度单元的Hazelcast节点

在Kubernetes中，最小的应用单元就是[Pod](http://kubernetes.io/v1.0/docs/user-guide/pods.html)。一个Pod就是同一个主机调度下的一个或者多个容器。在一个Pod中的所有容器共享同一个网络命名空间，同时可以有选择性的共享同一个数据卷。

在这种情况下，我们不能单独运行一个Hazelcast Pod，因为它的发现机制依赖于Service的定义。

##添加一个Hazelcast 服务

在Hazelcast中，一个[Service](http://kubernetes.io/v1.0/docs/user-guide/services.html)被描述为执行同一任务的Pods集合。比如，一个Hazelcast集群中的节点集合。Service的一个重要用途就是通过建立一个均衡负载器将流量均匀的分到集合中的每一个成员。此外，Service还可以作为一个标准的查询器，使动态变化的Pod集合提供有效通过Kubernetes的API。实际上，这个就是探索机制的工作原理，就是在service的基础上去发现Hazelcast Pods。
下面是对Service的描述：

```
apiVersion: v1
kind: Service
metadata: 
  labels: 
	name: hazelcast
  name: hazelcast
spec:
  ports:
	- port: 5701
  selector: 
	name: hazelcast
```

这里值得注意的是*selector*（选择器）。在标签的上层有一个查询器，它标识了被Service所覆盖的Pods集合。在这种情况下，selector就是代码中`name: hazelcast`。在接下来的 Replication Controller说明书中，你会看到Pods中有对应的标签，那么它就会被这个Service中对应的成员变量所选中。

创建该Serviced的命令如下：

```
$ kubectl create -f examples/hazelcast/hazelcast-service.yaml
```

##添加一个拷贝节点

Kubernetes和Hazelcast真正强大的地方在于它们可以轻松的建立一个可拷贝的、大小可调的Hazelcast集群。

在Kubernetes中，存在一个叫做[Replication Controller](http://kubernetes.io/v1.0/docs/user-guide/replication-controller.html)的管理器，专门用来管理相同Pods的拷贝集合。和Service一样，它也存在一个在集合成员变量中定义的选择查询器。和Service不同的是，它对拷贝的个数有要求，会通过创建或者删除Pods来确保当前Pods的数量符合要求。

Replication Controllers会通过匹配响应的选择查询器来确认要接收的Pods，下面我们将创建一个单拷贝的Replication Controller去接收已经存在的Hazelcast Pod。
 
```
apiVersion: v1
kind: ReplicationController
metadata: 
  labels: 
	name: hazelcast
  name: hazelcast
spec: 
  replicas: 1
  selector: 
	name: hazelcast
  template: 
	metadata: 
  	  labels: 
    	name: hazelcast
	spec: 
  	  containers: 
		- resources:
        	limits:
        	  cpu: 0.1
     	  image: quay.io/pires/hazelcast-kubernetes:0.5
      	  name: hazelcast
          env:
		  - name: "DNS_DOMAIN"
        	value: "cluster.local"
		  - name: POD_NAMESPACE
       		valueFrom:
         	  fieldRef:
            	fieldPath: metadata.namespace
    	  ports: 
			- containerPort: 5701
          	  name: hazelcast
```

在这段代码中，有一些需要注意的东西。首先要注意的是，我们运行的是*quay.io/pires/hazelcast-kubernetes* image, tag *0.5*。这个*busybox*安装在JRE8上。尽管如此，它还是添加了一个用户端的*应用程序*，从而可以发现集群中的Hazelcast节点并且引导一个Hazelcast实例。*HazelcastDiscoveryController*通过内置的搜索服务器来探索Kubernetes API Server，之后用Kubernetes API来发现新的节点。

你可能已经注意到了，我们会告知Kubernetes，容器会暴露*Hazelcast*端口。最终，我们需要告诉集群的管理器，我们需要一个CPU核。

对于Hazelcast Pod而言，Replication Controller块的配置基本相同，以上就是声明，它只是给管理器提供一种简单的建立新节点的方法。其它的部分就是包含了Controller选择条件的`selector`，以及配置Pod数量的`replicas`，在这个例子中数量为1。

最后，我们需要根据你的Kubernetes集群的DNS配置来设置`DNS_DOMAIN`的环境变量。

创建该控制器的命令：

```
$ kubectl create -f examples/hazelcast/hazelcast-controller.yaml
```

当控制器成功的准备好后，你就可以查询服务端点：

```
$ kubectl get endpoints hazelcast -o json
{
	"kind": "Endpoints",
	"apiVersion": "v1",
	"metadata": {
    	"name": "hazelcast",
        "namespace": "default",
	    "selfLink": "/api/v1/namespaces/default/endpoints/hazelcast",
	    "uid": "094e507a-2700-11e5-abbc-080027eae546",
	    "resourceVersion": "4094",
	    "creationTimestamp": "2015-07-10T12:34:41Z",
    	"labels": {
        "name": "hazelcast"
	    }
	},
	"subsets": [
	    {
	        "addresses": [
	            {
	                "ip": "10.244.37.3",
	                "targetRef": {
	                    "kind": "Pod",
	                    "namespace": "default",
	                    "name": "hazelcast-nsyzn",
	                    "uid": "f57eb6b0-2706-11e5-abbc-080027eae546",
	                    "resourceVersion": "4093"
	                }
	            }
	        ],
	        "ports": [
	            {
	                "port": 5701,
	                "protocol": "TCP"
	            }
	        ]
	    }
	]
}
```

你可以看到Service发现那些被Replication Controller建立的Pods。

这下变的更加有趣了。让我们把集群提高到2个Pod。

```
$ kubectl scale rc hazelcast --replicas=2
```

现在，如果你去列出集群中的Pods,你应该会看到2个Hazelcast Pods:

```
$ kubectl get pods
NAME              READY     STATUS    RESTARTS   AGE
hazelcast-nanfb   1/1       Running   0          40s
hazelcast-nsyzn   1/1       Running   0          2m
kube-dns-xudrp    3/3       Running   0          1h
```

如果想确保每一个Pods都在工作，你可以通过*log*命令来进行日志检查，如下：

```
$ kubectl log hazelcast-nanfb hazelcast
2015-07-10 13:26:34.443  INFO 5 --- [           main] com.github.pires.hazelcast.Application   : Starting Application on hazelcast-nanfb with PID 5 (/bootstrapper.jar started by root in /)
2015-07-10 13:26:34.535  INFO 5 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@42cfcf1: startup date [Fri Jul 10 13:26:34 GMT 2015]; root of context hierarchy
2015-07-10 13:26:35.888  INFO 5 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2015-07-10 13:26:35.924  INFO 5 --- [           main] c.g.p.h.HazelcastDiscoveryController     : Asking k8s registry at https://kubernetes.default.svc.cluster.local..
2015-07-10 13:26:37.259  INFO 5 --- [           main] c.g.p.h.HazelcastDiscoveryController     : Found 2 pods running Hazelcast.
2015-07-10 13:26:37.404  INFO 5 --- [           main] c.h.instance.DefaultAddressPicker        : [LOCAL] [someGroup] [3.5] Interfaces is disabled, trying to pick one address from TCP-IP config addresses: [10.244.77.3, 10.244.37.3]
2015-07-10 13:26:37.405  INFO 5 --- [           main] c.h.instance.DefaultAddressPicker        : [LOCAL] [someGroup] [3.5] Prefer IPv4 stack is true.
2015-07-10 13:26:37.415  INFO 5 --- [           main] c.h.instance.DefaultAddressPicker        : [LOCAL] [someGroup] [3.5] Picked Address[10.244.77.3]:5701, using socket ServerSocket[addr=/0:0:0:0:0:0:0:0,localport=5701], bind any local is true
2015-07-10 13:26:37.852  INFO 5 --- [           main] com.hazelcast.spi.OperationService       : [10.244.77.3]:5701 [someGroup] [3.5] Backpressure is disabled
2015-07-10 13:26:37.879  INFO 5 --- [           main] c.h.s.i.o.c.ClassicOperationExecutor     : [10.244.77.3]:5701 [someGroup] [3.5] Starting with 2 generic operation threads and 2 partition operation threads.
2015-07-10 13:26:38.531  INFO 5 --- [           main] com.hazelcast.system                     : [10.244.77.3]:5701 [someGroup] [3.5] Hazelcast 3.5 (20150617 - 4270dc6) starting at Address[10.244.77.3]:5701
2015-07-10 13:26:38.532  INFO 5 --- [           main] com.hazelcast.system                     : [10.244.77.3]:5701 [someGroup] [3.5] Copyright (c) 2008-2015, Hazelcast, Inc. All Rights Reserved.
2015-07-10 13:26:38.533  INFO 5 --- [           main] com.hazelcast.instance.Node              : [10.244.77.3]:5701 [someGroup] [3.5] Creating TcpIpJoiner
2015-07-10 13:26:38.534  INFO 5 --- [           main] com.hazelcast.core.LifecycleService      : [10.244.77.3]:5701 [someGroup] [3.5] Address[10.244.77.3]:5701 is STARTING
2015-07-10 13:26:38.672  INFO 5 --- [        cached1] com.hazelcast.nio.tcp.SocketConnector    : [10.244.77.3]:5701 [someGroup] [3.5] Connecting to /10.244.37.3:5701, timeout: 0, bind-any: true
2015-07-10 13:26:38.683  INFO 5 --- [        cached1] c.h.nio.tcp.TcpIpConnectionManager       : [10.244.77.3]:5701 [someGroup] [3.5] Established socket connection between /10.244.77.3:59951
2015-07-10 13:26:45.699  INFO 5 --- [ration.thread-1] com.hazelcast.cluster.ClusterService     : [10.244.77.3]:5701 [someGroup] [3.5]

Members [2] {
    Member [10.244.37.3]:5701
    Member [10.244.77.3]:5701 this
}

2015-07-10 13:26:47.722  INFO 5 --- [           main] com.hazelcast.core.LifecycleService      : [10.244.77.3]:5701 [someGroup] [3.5] Address[10.244.77.3]:5701 is STARTED
2015-07-10 13:26:47.723  INFO 5 --- [           main] com.github.pires.hazelcast.Application   : Started Application in 13.792 seconds (JVM running for 14.542)
```

接着是4个Pods:

```
$ kubectl scale rc hazelcast --replicas=4
```

然后通过刚才的操作去检查这4个成员是否连接。

##太长不读版

对于那些急性子，下面是这一章所用到的所有命令：

```
# 建立一个service去跟踪所有的Hazelcast Nodes
kubectl create -f examples/hazelcast/hazelcast-service.yaml

# 建立一个Replication Controller去拷贝Hazelcast Nodes
kubectl create -f examples/hazelcast/hazelcast-controller.yaml

# 升级成2个节点
kubectl scale rc hazelcast --replicas=2

# 升级成4个节点
kubectl scale rc hazelcast --replicas=4
```

##Hazelcast的发现机制源代码

点击[这里](https://github.com/pires/hazelcast-kubernetes-bootstrapper/blob/master/src/main/java/com/github/pires/hazelcast/HazelcastDiscoveryController.java)
#使用Kubernetes在云上原生部署cassandra

下面的文档描述了在Kubernetes上部署*cloud native* [Cassandra](http://cassandra.apache.org/)的开发过程。当我们讲*cloud native时，我们是指这样一款应用，它运行在集群管理器中，并利用集群管理器的基础设施来实现应用的功能。特别的是，在本章介绍的实例中，一个定制好的Cassandra *SeedProvider* 可以确保Cassandra能够动态发现新的Cassandra节点并把它添加到集群中。

这篇文档也在努力描述Kubernetes中的核心概念：*Pods*,*Services*和*Replication Controllers*。

##预备知识
下面的例子假设你已经安装和运行一个Kubernetes集群，并且你也下载好**kubectl**命令行工具以及配置在你的path中。请先阅读[入门指南](http://kubernetes.io/v1.0/docs/getting-started-guides/)获取在你平台上安装Kubernetes的指令。

本文中的例子还需要一些代码和配置文件。为了避免重复输入（代码和配置文件），你可以使用**git clone**命令从Kubernetes仓库中克隆文件到本地计算机。

###对于不耐心的友善提醒
这是一个有点长的教程，如果你想直接跳到“马上操作”的命令，可以到文末看[这些](http://kubernetes.io/v1.0/examples/cassandra/README.html#tl-dr)

##简单的单个Pod Cassandra节点
在Kubernetes中，应用的原子单元（【译者注】原子单元指不可再分的最小单元）是[Pod](http://kubernetes.io/v1.0/docs/user-guide/pods.html)。一个Pod指*必须*被部署在同一个主机的一个或者多个容器。同一个pod中的所有容器共享一个网络名称空间，以及可选地共享挂载卷。在这个简单的例子中，我们在pod中定义一个运行Cassandra的容器：
```sh
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: cassandra
  name: cassandra
spec:
  containers:
  - args:
    - /run.sh
    resources:
      limits:
        cpu: "0.5"
    image: gcr.io/google_containers/cassandra:v5
    name: cassandra
    ports:
    - name: cql
      containerPort: 9042
    - name: thrift
      containerPort: 9160
    volumeMounts:
    - name: data
      mountPath: /cassandra_data
    env:
    - name: MAX_HEAP_SIZE
      value: 512M
    - name: HEAP_NEWSIZE
      value: 100M
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
  volumes:
    - name: data
```

上面的描述中有几点需要注意。首先我们运行的镜像是*kubernetes/cassandra*。这是一个标准的安装在Debian上的Cassandra镜像。但是它还给Cassandra添加一个定制的[SeedProvider](https://svn.apache.org/repos/asf/cassandra/trunk/src/java/org/apache/cassandra/locator/SeedProvider.java)，在Cassandra中， SeedProvider设置一个gossip协议用来发现其它Cassandra节点。**KubernetesSeedProvider**使用内置的Kubernetes发现服务找到KubernetesAPI服务器，然后利用Kubernetes API发现新的节点。（稍后将详细介绍）

也许你注意到（在上面的脚本中）我们设置了一些Cassandra参数（**MAX_HEAP_SIZE** 和**HEAP_NEWSIZE**）以及添加了关于[名称空间](http://kubernetes.io/v1.0/docs/user-guide/namespaces.html)的信息。我们还告诉Kubernetes容器暴露了CQL和Thrift的API端口。最后，我们告诉集群管理器我们需要0.5个CPU（0.5个内核core）

理论上会立刻创建一个Cassandra pod，但是在Cassandra部署过程中，**KubernetesSeedProvider**需要知道现在哪个节点上创建服务（【译者注】这会消耗一些时间）

##Cassandra服务
在Kubernetes中一个[服务](http://kubernetes.io/v1.0/docs/user-guide/services.html)是指执行相同任务的一系列pods集合（【译者注】一个任务可能由多个pod协同完成）。例如，Cassandra集群中的pods集合可以是Kubernetes服务，甚至仅仅是一个pod也可以。服务的一个重要用途是在pods成员间分发流量实现负载均衡。*服务*另外一个用途也可以用作长期查询，查询调用Kubernetes API使得动态改变pod（或者是上面创建的一个pod）成为可能。这就是我们最初使用Cassandra服务的两种方式。

下面是服务的描述：
```sh
apiVersion: v1
kind: Service
metadata:
  labels:
    name: cassandra
  name: cassandra
spec:
  ports:
    - port: 9042
  selector:
    name: cassandra
```

这里对于节点最重要的是**选择器**，（选择器）是建立在标签上的查询，标签则标记了服务中包含的pods集合。在这个例子中选择器是*name=cassandra*。如果你现在回头看Pod的描述，你会发现pod有和它对应的标签。所以这个pod会被选中成为服务中的成员。

通过下面命令创建服务：
```sh
$ kubectl create -f examples/cassandra/cassandra-service.yaml
```
现在服务运行起来了，我们可以使用前面提到的说明来创建第一个Cassandra。
```language
$ kubectl create -f examples/cassandra/cassandra.yaml
```

不久，你会看到pod也运行了，以及pod中运行的单个容器：
```language
$ kubectl get pods cassandra
NAME        READY     STATUS    RESTARTS   AGE
cassandra   1/1       Running   0          55s
```

你也可以查询服务的端点来检查pod是否被正确选中：
```language
$ kubectl get endpoints cassandra -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  creationTimestamp: 2015-06-21T22:34:12Z
  labels:
    name: cassandra
  name: cassandra
  namespace: default
  resourceVersion: "944373"
  selfLink: /api/v1/namespaces/default/endpoints/cassandra
  uid: a3d6c25f-1865-11e5-a34e-42010af01bcc
subsets:
- addresses:
  - ip: 10.244.3.15
    targetRef:
      kind: Pod
      name: cassandra
      namespace: default
      resourceVersion: "944372"
      uid: 9ef9895d-1865-11e5-a34e-42010af01bcc
  ports:
  - port: 9042
    protocol: TCP
```

##添加复制节点
当然，单节点集群并不特别有趣。Kubernetes和Cassandra的真正能量在于可以很方便创建一个副本，使得Cassandra集群可扩展。

在Kubernetes中一个[复制控制器](http://kubernetes.io/v1.0/docs/user-guide/replication-controller.html)负责复制相同的pods。复制控制器的作用和*服务*的相同点在于复制控制器有选择器能查询集合中的指定成员。与*服务*不同的是复制控制器有预期数量的副本，所以它能创建或删除*Pods*确保pod数量达到预期状态。

复制控制器会"接受"已经存在并且符合查询条件的pods，现在我们会创建一个只有一个副本的复制控制器，用它来接收已经存在的Cassandra pod：
```language
apiVersion: v1
kind: ReplicationController
metadata:
  labels:
    name: cassandra
  name: cassandra
spec:
  replicas: 1
  selector:
    name: cassandra
  template:
    metadata:
      labels:
        name: cassandra
    spec:
      containers:
        - command:
            - /run.sh
          resources:
            limits:
              cpu: 0.5
          env:
            - name: MAX_HEAP_SIZE
              value: 512M
            - name: HEAP_NEWSIZE
              value: 100M
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          image: gcr.io/google_containers/cassandra:v5
          name: cassandra
          ports:
            - containerPort: 9042
              name: cql
            - containerPort: 9160
              name: thrift
          volumeMounts:
            - mountPath: /cassandra_data
              name: data
      volumes:
        - name: data
          emptyDir: {}
```

大部分复制控制器的定义和上面的Cassandra pod的定义类似，定义简单地为复制控制器提供一个模板，创建新的Cassandra pod时可以很方便拿来使用。不同的部分是**选择器**属性 包含控制器的选择器查询以及**副本**属性中的副本的预期数量，在这个例子中是1。

创建控制器的指令：
```language
$ kubectl create -f examples/cassandra/cassandra-controller.yaml
```

现在事实上已经不那么无趣了，因为之前没做新的事情。现在它会变得有趣。

现在我们把集群数量扩展到2：
```language
$ kubectl scale rc cassandra --replicas=2
```

现在如果你列出集群中的pods，查询符合标签**name=cassandra**的pods，你会看到两个cassandra pod：
```language
$ kubectl get pods -l="name=cassandra"
NAME              READY     STATUS    RESTARTS   AGE
cassandra         1/1       Running   0          3m
cassandra-af6h5   1/1       Running   0          28s
```
注意其中一个pod的名字是人们可读的**cassandra**，它是我们在配置文件中指定的。另一个是复制控制器随机生成的字符串。

为了证明所有的pods都正常工作，你可以使用**nodetool**命令检查集群的状态。使用**kubectl exec**命令在Cassandra Pod中运行**nodetool**。
```language
$ kubectl exec -ti cassandra -- nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  10.244.0.5  74.09 KB   256     100.0%            86feda0f-f070-4a5b-bda1-2eeb0ad08b77  rack1
UN  10.244.3.3  51.28 KB   256     100.0%            dafe3154-1d67-42e1-ac1d-78e7e80dce2b  rack1
```

现在扩展集群到4个节点：
```language
kubectl scale rc cassandra --replicas=4
```

一会后再次检查状态：
```language
$ kubectl exec -ti cassandra -- nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  10.244.2.3  57.61 KB   256     49.1%             9d560d8e-dafb-4a88-8e2f-f554379c21c3  rack1
UN  10.244.1.7  41.1 KB    256     50.2%             68b8cc9c-2b76-44a4-b033-31402a77b839  rack1
UN  10.244.0.5  74.09 KB   256     49.7%             86feda0f-f070-4a5b-bda1-2eeb0ad08b77  rack1
UN  10.244.3.3  51.28 KB   256     51.0%             dafe3154-1d67-42e1-ac1d-78e7e80dce2b  rack1
```

##tl; dr;
下面是给那些不耐心的小伙伴的，下面是以上命令的总结：
```language
# create a service to track all cassandra nodes
kubectl create -f examples/cassandra/cassandra-service.yaml

# create a single cassandra node
kubectl create -f examples/cassandra/cassandra.yaml

# create a replication controller to replicate cassandra nodes
kubectl create -f examples/cassandra/cassandra-controller.yaml

# scale up to 2 nodes
kubectl scale rc cassandra --replicas=2

# validate the cluster
kubectl exec -ti cassandra -- nodetool status

# scale up to 4 nodes
kubectl scale rc cassandra --replicas=4
```

##Seed Provider Source
[看这里](http://kubernetes.io/v1.0/examples/cassandra/java/src/io/k8s/cassandra/KubernetesSeedProvider.java)
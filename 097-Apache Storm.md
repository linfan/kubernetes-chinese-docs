# Storm 示例
`译者：White` `校对：无`


接下来的例子中，你将会使用Kubernetes和[Docker](http://docker.io/)来创建一个多功能的[Apache Storm](http://storm.apache.org/)集群。

你将会设置一个[Apache ZooKeeper](http://zookeeper.apache.org/)服务，一个Storm master服务（又名Nimbus主机），以及一个Storm工作者集合（又名监管者）。

如果你已经熟悉这个部分，请直接跳到[tl;dr](http://kubernetes.io/v1.0/examples/storm/README.html#tldr)章节。

### 资源

可以自由获取这些资源文件：

* Docker镜像 - [https://github.com/mattf/docker-storm](https://github.com/mattf/docker-storm)
* Docker受信的构建文件 - [https://registry.hub.docker.com/search?q=mattf/storm](https://registry.hub.docker.com/search?q=mattf/storm)


## **步骤零：前期准备**

这个示例假设你已经安装运行了一个Kubernetes集群，环境路径下已经安装了`kubectl`命令行工具。请查看不同平台的安装说明[开始](http://kubernetes.io/v1.0/docs/getting-started-guides/)。

## **步骤一：启动ZooKeeper服务**

ZooKeeper是一个分布式协调者[服务](http://kubernetes.io/v1.0/docs/user-guide/services.html)，Storm使用它来作为引导程序和存储运行转态数据。

使用这个[examples/storm/zookeeper.json](http://kubernetes.io/v1.0/examples/storm/zookeeper.json)文件来创建一个运行ZooKeeper服务的[pod](http://kubernetes.io/v1.0/docs/user-guide/pods.html)。

```bash
$ kubectl create -f examples/storm/zookeeper.json
```

然后使用[examples/storm/zookeeper-service.json](http://kubernetes.io/v1.0/examples/storm/zookeeper-service.json)文件创建一个逻辑服务终端节点用来给Storm访问ZooKeeper pod。

```bash
$ kubectl create -f examples/storm/zookeeper-service.json
```

在这之前，你需要确保ZooKeeper pod处于运行态并且可以被访问。

### **查看ZooKeeper是否运行**

```bash
$ kubectl get pods
NAME        READY     STATUS    RESTARTS   AGE
zookeeper   1/1       Running   0          43s
```

### **查看ZooKeeper是否可以访问**

```bash
$ kubectl get services
NAME                LABELS                                    SELECTOR            IP(S)               PORT(S)
kubernetes          component=apiserver,provider=kubernetes   <none>              10.254.0.2          443
zookeeper           name=zookeeper                            name=zookeeper      10.254.139.141      2181

$ echo ruok | nc 10.254.139.141 2181; echo
imok
```

## **步骤二：启动Nimbus服务**

Nimbus服务是Storm集群的主节点(或者首要)服务。Nimbus依赖于多种功能的ZooKeeper服务。

使用[examples/storm/storm-nimbus.json](http://kubernetes.io/v1.0/examples/storm/storm-nimbus.json)文件创建一个运行Nimbus服务的pod。

```bash
$ kubectl create -f examples/storm/storm-nimbus.json
```

然后使用[examples/storm/storm-nimbus-service.json](http://kubernetes.io/v1.0/examples/storm/storm-nimbus-service.json)文件创建一个逻辑服务终端节点用来给Storm工作者访问Nimbus pod。

```bash
$ kubectl create -f examples/storm/storm-nimbus-service.json
```

确保Nimbus服务运行正常。

### **查看Nimbus节点是否运行以及可以访问**

```bash
$ kubectl get services
NAME                LABELS                                    SELECTOR            IP(S)               PORT(S)
kubernetes          component=apiserver,provider=kubernetes   <none>              10.254.0.2          443
zookeeper           name=zookeeper                            name=zookeeper      10.254.139.141      2181
nimbus              name=nimbus                               name=nimbus         10.254.115.208      6627

$ sudo docker run -it -w /opt/apache-storm mattf/storm-base sh -c '/configure.sh 10.254.139.141 10.254.115.208; ./bin/storm list'
...
No topologies running.
```

## **步骤三：启动Storm工作者**

在Storm集群中，Storm工作者（或者监督者）用来完成繁重的工作。Nimbus服务管理这些运行流处理拓扑应用的工人。

Storm工作者需要保证ZooKeeper和Nimbus服务处于运行态。

使用[examples/storm/storm-worker-controller.json](http://kubernetes.io/v1.0/examples/storm/storm-worker-controller.json)文件来创建[副本控制器](http://kubernetes.io/v1.0/docs/user-guide/replication-controller.html)来管理工作者pods。

```bash
$ kubectl create -f examples/storm/storm-worker-controller.json
```

### **查看工作者们是否在运行**

一种查看工作者信息的方式是，通过ZooKeeper服务查看有多少客户端在运行。

```bash
$  echo stat | nc 10.254.139.141 2181; echo
Zookeeper version: 3.4.6--1, built on 10/23/2014 14:18 GMT
Clients:
 /192.168.48.0:44187[0](queued=0,recved=1,sent=0)
 /192.168.45.0:39568[1](queued=0,recved=14072,sent=14072)
 /192.168.86.1:57591[1](queued=0,recved=34,sent=34)
 /192.168.8.0:50375[1](queued=0,recved=34,sent=34)
 /192.168.45.0:39576[1](queued=0,recved=34,sent=34)

Latency min/avg/max: 0/2/2570
Received: 23199
Sent: 23198
Connections: 5
Outstanding: 0
Zxid: 0xa39
Mode: standalone
Node count: 13
```

Nimbus服务和每个工作者都对应着一个客户端。理想情况下，应该可以在副本控制器创建前后从ZooKeeper获取`stat`输出。

（欢迎提交pull requests使用不同的方式来验证workers）

### **tl;dr**

kubectl create -f zookeeper.json

kubectl create -f zookeeper-service.json

确保ZooKeeper Pod正在运行（使用：`kubectl get pods`）。

kubectl create -f storm-nimbus.json

kubectl create -f storm-nimbus-service.json

确保Nimbus Pod正在运行。

kubectl create -f storm-worker-controller.json

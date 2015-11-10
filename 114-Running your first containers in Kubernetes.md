#在Kubernetes上运行你的第一个容器

好了，如果你已经开始了任何一个入门指南，并且启动了一个Kubernetes集群。那么接下来呢？ 这个片指南会帮助你正对KUbernetes,在其集群上运行第一个容器。

##运行一个容器 (简单版)

>从这时开始，我假设你已经根据其它入门指南安装了kubectl。

>下面这行kubectl命令会穿件两个监听80端口的nginx pod. 还会创建一名为my-nginx个replication controller,用来保证始终会有两个pod在运行。

```
kubectl run my-nginx --image=nginx --replicas=2 --port=80
```

>一旦这些pod被创建好了， 你可以列出他们并查看他们的启动和运行。

```
kubectl get pods
```

>你也能够看见replication controller被创建了：

```
kubectl get rc
```
To stop the two replicated containers, stop the replication controller:
如果要停止这两个被复制的容器，你可以通过停止replication： controller

```
kubectl stop rc my-nginx
```

##让你的的pod可以被外网方位.

在一些平台上（例如Google Compute Engine），kubectl命令能够集成云端提供的API来给pod条件公有IP地址，可以通过以下命令来实现：

```
kubectl expose rc my-nginx --port=80 --type=LoadBalancer
```

>这个命令会打印出被创建了的service,以及一个外部IP地址映射到service. 对外的IP地址根你实际运行环境有关。例如，对于Google Compute Engine的外部IP地址会被列为新创建的服务的一部分，还可以通在运行时检索。

```
kubectl get services
```

>为了访问你的nginx初始页面,你还不得不保证通过外部IP的通信是被允许的。那么就要通过让防火墙允许80端口通信才可以做到

##接下来: 配置文件

Most people will eventually want to use declarative configuration files for creating/modifying their applications. A simplified introduction is given in a different document.

大多数人最终都会响使用声明式的配置文件来创建或修改他们的应用程勋。另外一个文档给出了一个[简单介绍](http://kubernetes.io/v1.0/docs/user-guide/simple-yaml.html)。

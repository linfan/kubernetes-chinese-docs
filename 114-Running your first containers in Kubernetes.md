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

On some platforms (for example Google Compute Engine) the kubectl command can integrate with your cloud provider to add a public IP address for the pods, to do this run:

在一些平台上（例如Google Compute Engine），kubectl命令能够集成云端提供的API来给pod条件公有IP地址，可以通过以下命令来实现：

kubectl expose rc my-nginx --port=80 --type=LoadBalancer
This should print the service that has been created, and map an external IP address to the service. Where to find this external IP address will depend on the environment you run in. For instance, for Google Compute Engine the external IP address is listed as part of the newly created service and can be retrieved by running

kubectl get services
In order to access your nginx landing page, you also have to make sure that traffic from external IPs is allowed. Do this by opening a firewall to allow traffic on port 80.

##Next: Configuration files

Most people will eventually want to use declarative configuration files for creating/modifying their applications. A simplified introduction is given in a different document.


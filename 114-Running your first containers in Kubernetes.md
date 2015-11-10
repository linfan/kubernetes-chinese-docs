#在Kubernetes上运行你的第一个容器

好了，如果你已经开始了任何一个入门指南，并且启动了一个Kubernetes集群。那么接下来呢？ 这个片指南会帮助你正对KUbernetes,在其集群上运行第一个容器。

##运行一个容器 (简单版)

From this point onwards, it is assumed that kubectl is on your path from one of the getting started guides.

从这时开始，我假设你已经根据其它入门指南安装了kubectl。

The kubectl run line below will create two nginx pods listening on port 80. It will also create a replication controller named my-nginx to ensure that there are always two pods running.

```
kubectl run my-nginx --image=nginx --replicas=2 --port=80
```
Once the pods are created, you can list them to see what is up and running:

```
kubectl get pods
```
You can also see the replication controller that was created:

```
kubectl get rc
```
To stop the two replicated containers, stop the replication controller:

```
kubectl stop rc my-nginx
```

##Exposing your pods to the internet.

On some platforms (for example Google Compute Engine) the kubectl command can integrate with your cloud provider to add a public IP address for the pods, to do this run:

kubectl expose rc my-nginx --port=80 --type=LoadBalancer
This should print the service that has been created, and map an external IP address to the service. Where to find this external IP address will depend on the environment you run in. For instance, for Google Compute Engine the external IP address is listed as part of the newly created service and can be retrieved by running

kubectl get services
In order to access your nginx landing page, you also have to make sure that traffic from external IPs is allowed. Do this by opening a firewall to allow traffic on port 80.

##Next: Configuration files

Most people will eventually want to use declarative configuration files for creating/modifying their applications. A simplified introduction is given in a different document.

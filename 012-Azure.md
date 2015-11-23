#在Azure上使用CoreOS和Weave的Kubernetes

##介绍
在本指南中我将演示如何在Azure云端部署Kubernetes集群。您将使用CoreOS与[Weave](http://weave.works/)，Weave以透明而可靠的方式实现了简单、安全的网络。本指南的目的是提供一个即开即装即用的实现方法，以便最终可以稍加改变就可以投入到生产环境中。本文将演示如何提供一个专门的Kubernetes主节点和ETCD节点，并展示如何轻松地扩展集群。
###前提条件
1.您需要一个Azure账号。
###开始吧！
开始之前，您需要checkout下代码：

```
git clone https://github.com/kubernetes/kubernetes  

cd kubernetes/docs/getting-started-guides/coreos/azure/
```
您需要在您的机器上安装Node.js，如果您之前使用过Azure CIL，那么您应该已经安装了。
首先，您需要安装一些依赖：

```
npm install
```
现在，您需要做的是：

```
./azure-login.js -u <your_username>
./create-kubernetes-cluster.js
```
这个脚本会提供适用于生产环境的集群，集群中有一个3个专用的ETCD节点形成环形：1个kubernetes主节点和2个kubernetes节点。**KUBE-00**虚拟机将是主节点，您的工作负荷只会部署在KUBE-01节点和KUBE-02节点上。最初，所有的虚拟机都是单核的，以确保自由层的用户无需额外的代价就可以复制它。稍后我将展示如何添加更多更大的虚拟机。

[图1](http://kubernetes.io/v1.1/docs/getting-started-guides/coreos/azure/initial_cluster.png)
[图2](http://kubernetes.io/v1.1/docs/getting-started-guides/coreos/azure/initial_cluster.png)  
一旦Azure虚拟机创建完成，你应该可以看到下面这样的信息：

```
...
azure_wrapper/info: Saved SSH config, you can use it like so: `ssh -F  ./output/kube_1c1496016083b4_ssh_conf <hostname>`  
azure_wrapper/info: The hosts in this deployment are:
 [ 'etcd-00', 'etcd-01', 'etcd-02', 'kube-00', 'kube-01', 'kube-02' ]
azure_wrapper/info: Saved state into `./output/kube_1c1496016083b4_deployment.yml`
```
像下面这样登陆进主节点：

```
ssh -F  ./output/kube_1c1496016083b4_ssh_conf kube-00

```
**注**：配置文件名字可能有所不同，确保使用你所看到的那个。  
检查一下集群中的两个节点：

```
core@kube-00 ~ $ kubectl get nodes
NAME      LABELS                           STATUS
kube-01   kubernetes.io/hostname=kube-01   Ready
kube-02   kubernetes.io/hostname=kube-02   Ready
```
##部署工作负载
现在让我们按照Guestbook的实例来部署：

```
kubectl create -f ~/guestbook-example

```
您需要等待pod部署完成，然后执行下面的命令，等待**STATUS**
从**Pending**变为**Running**：

```
kubectl get pods --watch

```
**注**：大部分的时间将会花在下载每个节点的Docker容器镜像上。
最后您将会看到：

```
NAME                READY     STATUS    RESTARTS   AGE
frontend-0a9xi      1/1       Running   0          4m
frontend-4wahe      1/1       Running   0          4m
frontend-6l36j      1/1       Running   0          4m
redis-master-talmr  1/1       Running   0          4m
redis-slave-12zfd   1/1       Running   0          4m
redis-slave-3nbce   1/1       Running   0          4m
```
##扩展
两个单核的节点肯定是无法满足现如今的生产系统，让我们通过添加几个更大的节点来扩展集群。
您需要再打开一个您机器上的终端窗口，进入相同的工作目录（也就是说这个目录：~/Workspace/kubernetes/docs/getting-started-guides/coreos/azure/）
首先，让我们设置一下新虚拟机的大小：

```
export AZ_VM_SIZE=Large

```
现在，我们使用先前部署的状态文件和添加的一系列节点来运行扩展脚本：

```
core@kube-00 ~ $ ./scale-kubernetes-cluster.js ./output/kube_1c1496016083b4_deployment.yml 2
...
azure_wrapper/info: Saved SSH config, you can use it like so: `ssh -F  ./output/kube_8f984af944f572_ssh_conf <hostname>`
azure_wrapper/info: The hosts in this deployment are:
 [ 'etcd-00',
  'etcd-01',
  'etcd-02',
  'kube-00',
  'kube-01',
  'kube-02',
  'kube-03',
  'kube-04' ]
azure_wrapper/info: Saved state into `./output/kube_8f984af944f572_deployment.yml`
```
**注**：这一步在`./output`下产生了一些新文件。 
 
回到**kube-00**：

```
core@kube-00 ~ $ kubectl get nodes
NAME      LABELS                           STATUS
kube-01   kubernetes.io/hostname=kube-01   Ready
kube-02   kubernetes.io/hostname=kube-02   Ready
kube-03   kubernetes.io/hostname=kube-03   Ready
kube-04   kubernetes.io/hostname=kube-04   Ready
```
您可以看到又有两个节点顺利地加入进来，现在，让我们来扩展Guestbook实例的数量。

首先，再次检查一下有多少replication controller：

```
core@kube-00 ~ $ kubectl get rc
ONTROLLER     CONTAINER(S)   IMAGE(S)                                    SELECTOR            REPLICAS
frontend       php-redis      kubernetes/example-guestbook-php-redis:v2   name=frontend       3
redis-master   master         redis                                       name=redis-master   1
redis-slave    worker         kubernetes/redis-slave:v2                   name=redis-slave    2
```
基于现在有四个节点，让我们进行相应地扩展：

```
core@kube-00 ~ $ kubectl scale --replicas=4 rc redis-slave
>>>>>>> coreos/azure: Updates for 1.0
scaled
core@kube-00 ~ $ kubectl scale --replicas=4 rc frontend
scaled
```
现在，再来检查下：

```
core@kube-00 ~ $ kubectl get rc
CONTROLLER     CONTAINER(S)   IMAGE(S)                                    SELECTOR            REPLICAS
frontend       php-redis      kubernetes/example-guestbook-php-redis:v2   name=frontend       4
redis-master   master         redis                                       name=redis-master   1
redis-slave    worker         kubernetes/redis-slave:v2                   name=redis-slave    4
```
现在，您已经拥有了更多的前端Guestbook和redis slave实例。如果您查看一下所有`name=fronted`的节点，您会看到每个节点上都运行着一个实例。

```
core@kube-00 ~/guestbook-example $ kubectl get pods -l name=frontend
NAME             READY     STATUS    RESTARTS   AGE
frontend-0a9xi   1/1       Running   0          22m
frontend-4wahe   1/1       Running   0          22m
frontend-6l36j   1/1       Running   0          22m
frontend-z9oxo   1/1       Running   0          41s
```

##将应用暴露给外部
Kubernetes 1.0 中没有原生的Azure负载均衡器，不过，下面演示了如何将Guestbook应用暴露给Internet。

```
./expose_guestbook_app_port.sh ./output/kube_1c1496016083b4_ssh_conf
Guestbook app is on port 31605, will map it to port 80 on kube-00
info:    Executing command vm endpoint create
+ Getting virtual machines
+ Reading network configuration
+ Updating network configuration
info:    vm endpoint create command OK
info:    Executing command vm endpoint show
+ Getting virtual machines
data:      Name                          : tcp-80-31605
data:      Local port                    : 31605
data:      Protcol                       : tcp
data:      Virtual IP Address            : 137.117.156.164
data:      Direct server return          : Disabled
info:    vm endpoint show command OK
```
然后，您就可以通过上面所展示的**kube-00**的Azure虚拟ip（在我的例子中，也就是指http://137.117.156.164/），在任何地方连接它。

##接下来
现在您已经拥有一个运行在Azure上规模性的集群，祝贺！  
或许，您应该尝试部署其他的[应用示例](http://kubernetes.io/v1.1/examples/)，或者动手写一个自己的。

##移除...
如果您不希望顾虑Azure的费用问题，您可以移除集群。正如您看到的，移除它非常简单：

```
./destroy-cluster.js ./output/kube_8f984af944f572_deployment.yml

```
**注**：确保使用最新的状态文件，因为扩展之后生成了新的文件。
顺便说一下，如果您喜欢，您可以使用文中所示脚本部署多个集群。

#管理应用：部署持续运行的应用
在前面的章节里，我们了解了如何用`kubectl run`快速部署一个简单的复制的应用以及如何用pods（configuring-containers.md）配置并生成单次运行的容器。本文，我们将使用基于配置的方法来部署一个持续运行的复制的应用。
##用配置文件生成复制品集合
Kubernetes用`Replication Controllers`创建并管理复制的容器集合（实际上是复制的Pods）。`Replication Controller`简单地确保在任一时间里都有特定数量的pod副本在运行。如果运行的太多，它会杀掉一些；如果运行的太少，它会启动一些。这和谷歌计算引擎的Instance Group Manager以及AWS的Auto-scaling Group（不带扩展策略）类似。在[快速开始](http://kubernetes.io/v1.0/docs/user-guide/quick-start.html)章节里用`kubctl run`创建的用来跑Nginx的`Replication Controller`可以用下面的YAML描述：
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
和指定一个单独的Pod相比，不同的是设置了这里的kind域为ReplicationController，设定了需要的副本（replicas）数量以及把Pod的定义放到了template域下面。pods的名字不需要显示指定，因为它们是由`replication controller`的名字生成的。要查看支持的域列表，可以看[replication controller API object](https://htmlpreview.github.io/?https://github.com/GoogleCloudPlatform/kubernetes/v1.0.1/docs/api-reference/definitions.html#_v1_replicationcontroller)。
和创建pods一样，也可以用`create`命令来创建这个replication controller：
```
$ kubectl create -f ./nginx-rc.yaml
replicationcontrollers/my-nginx
```
`replication controller`会替换删除的或者因不明原因终止的（比如节点失败）pods，这和直接创建的pods的情况是不一样。基于这样的考量，对于一个需要持续运行的应用，即便你的应用只需要一个单独的pod，我们也推荐使用`replication controller`。对于单独的pod，在配置文件里可以省略`replicas`这个域，因为不设置的时候默认就只有一个副本。
##查看replication controller的状态
可以用`get`命令查看你创建的replication controller：
```
$ kubectl get rc
CONTROLLER   CONTAINER(S)   IMAGE(S)   SELECTOR    REPLICAS
my-nginx     nginx          nginx      app=nginx   2
```

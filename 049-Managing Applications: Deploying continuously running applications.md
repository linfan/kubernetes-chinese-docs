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

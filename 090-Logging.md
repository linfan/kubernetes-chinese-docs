# 对接集群级别日志到谷歌云日志服务
通常一个忙碌的Kubernetes集群运行着很多系统和应用级别的pods.那么系统管理员如何才能收集，管理和查看这些系统pods的日志呢?那么用户应该如何查询这些由pods组成的应用日志呢?这些pods可能会被重新启动或者是自动的被Kubernetes系统创建出来。这些问题可以通过Kubernetes集群日志服务来解决。

集群级别的日志服务能够让我们收集来自于运行在pod中的容器镜像或者是pod，甚至是整个集群的日志，这些日志持久性记录了超出他们运行生命周期的全部状态。这篇文章中，假设我们已经创建了Kubernetes集群，并且支持向谷歌云日志服务发送集群级别日志。集群一旦创建，你会拥有一个系统pods的集合，这些pods运行在`kube-system`命名空间中，它们支持收集监控、日志记录以及Kubernetes服务的域名名称解析信息。

```
$ kubectl get pods --namespace=kube-system
NAME                                           READY     REASON    RESTARTS   AGE
fluentd-cloud-logging-kubernetes-minion-0f64   1/1       Running   0          32m
fluentd-cloud-logging-kubernetes-minion-27gf   1/1       Running   0          32m
fluentd-cloud-logging-kubernetes-minion-pk22   1/1       Running   0          31m
fluentd-cloud-logging-kubernetes-minion-20ej   1/1       Running   0          31m
kube-dns-v3-pk22                               3/3       Running   0          32m
monitoring-heapster-v1-20ej                    0/1       Running   9          32m
```


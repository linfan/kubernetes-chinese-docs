
# 应用：kubectl port-forward

kubectl port-forward命令转发了本地端口到pod端口的连接。它的[手册](http://kubernetes.io/v1.0/docs/user-guide/kubectl/kubectl_port-forward.html)现在这里可以查看。相比于[kubectl proxy](http://kubernetes.io/v1.0/docs/user-guide/accessing-the-cluster.html#using-kubectl-proxy)，`kubectl port-forward`也可以转发TCP流量而`kubectl proxy`只能转发HTTP流量。本页阐述了如何使用`kubectl port-forward`去连接Redis 数据库，这在数据库查错中很有用。

# 创建一个Redis主服务

```
$ kubectl create examples/redis/redis-master.yaml
pods/redis-master

```
等Redis主服务的pod状态变为Running和Ready。
```
$ kubectl get pods
NAME           READY     STATUS    RESTARTS   AGE
redis-master   2/2       Running   0          41s

```

# 连接到Redis主服务

Redis主服务监听在端口6379，使用下面命令可以验证，
```
$ kubectl get pods redis-master -t='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
6379

```
然后我们转发redis主服务pod的端口6379到本地6379端口，
```
$ kubectl port-forward -p redis-master 6379:6379
I0710 14:43:38.274550    3655 portforward.go:225] Forwarding from 127.0.0.1:6379 -> 6379
I0710 14:43:38.274797    3655 portforward.go:225] Forwarding from [::1]:6379 -> 6379

```
想要验证连接成功，我们可以在本地启动一个redis客户端，
```
$ redis-cli
127.0.0.1:6379> ping
PONG

```
现在我们就可以从本地来检查数据库了。

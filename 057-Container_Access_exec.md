# 使用kubectl exec检查容器中的环境变量
`译者：李昂` `校对：无`


Kubernetes通过环境变量来暴露[services](http://kubernetes.io/v1.0/docs/user-guide/services.html#environment-variables)。使`用kubectl exec`去检查环境变量会很方便。

首先我们创建一个pod和一个service，
```
$ kubectl create -f examples/guestbook/redis-master-controller.yaml
$ kubectl create -f examples/guestbook/redis-master-service.yaml

```
等到pod的状态为Running和Ready，
```
$ kubectl get pod
NAME                 READY     REASON       RESTARTS   AGE
redis-master-ft9ex   1/1       Running      0          12s

```
然后我们就可以检查pod的环境变量了，
```
$ kubectl exec redis-master-ft9ex env
...
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_SERVICE_HOST=10.0.0.219
...

```

# 使用kubectl exec检查挂载的数据卷（volume）

使用`kubectl exec`检查你希望挂载的数据卷（volume）同样很方便。首先还是创建一个pod并且挂载一个数据卷到这个pod的/data/redis目录，
```
kubectl create -f docs/user-guide/walkthrough/pod-redis.yaml
```
等待pod状态为Running和Ready，
```
$ kubectl get pods
NAME      READY     REASON    RESTARTS   AGE
storage   1/1       Running   0          1m

```
然后我们就可以使用`kubectl exec`去验证数据卷已经挂载到了/data/redis目录，
```
$ kubectl exec storage ls /data
redis

```

# 使用kubectl exec在pod中打开一个bash终端

在pod中打开一个终端毕竟才是最直接检查pod的方式。假设pod已经在运行，
```
$ kubectl exec -ti storage -- bash
root@storage:/data#

```
如上所做，你就可以打开pod的终端了。

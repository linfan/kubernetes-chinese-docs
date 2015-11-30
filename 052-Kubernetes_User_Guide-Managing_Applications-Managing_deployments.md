#Kubernetes用户指南：管理应用：管理部署
`译者：iT2afL0rd` `校对：无`


##组织资源配置
很多应用程序需求创建多重资源，比如Replication Controller和Service。对于多重资源的管理，可以简单地把它们聚在一起，放在同一个文件里（在YAML里面用`---`分隔）。例如：

```
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: nginx
---
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

多重资源可以和单一资源以同样的方式创建：

```
$ kubectl create -f ./nginx-app.yaml
services/my-nginx-svc
replicationcontrollers/my-nginx
```

资源会按照在文件中出现的顺序被创建。因此，最好先定义Service，这会保证当Replication Controller创建Pod的时候，调度器可以铺开和这个Service相关的Pod。

`kubectl create`也接受多重`-f`参数：

```
$ kubectl create -f ./nginx-svc.yaml -f ./nginx-rc.yaml
```

而且除单个文件之外还可以传一个目录：

```
$ kubectl create -f ./nginx/
```

`kubectl`会读取任何后缀名为`.yaml`,`.yml`或者`.json`的文件。

推荐的做法是把同样的微服务或者应用层相关的资源放在同一个文件里，然后把整个应用相关的文件都放在同一个目录里。如果应用中的各个层通过DNS互相绑定了，那么你就可以简单地一起部署它们。

URL也可以成为配置文件的源，这样对于直接部署GitHub上的配置文件非常好用：

```
$ kubectl create -f https://raw.githubusercontent.com/GoogleCloudPlatform/kubernetes/master/docs/user-guide/replication.yaml
replicationcontrollers/nginx
```

##kubectl批量操作

`kubectl`可以批量做的不仅仅只有创建资源。它也可以从配置文件里抽取资源名字来执行其他操作，特别是用来删除创建的重复资源：

```
$ kubectl delete -f ./nginx/
replicationcontrollers/my-nginx
services/my-nginx-svc
```

在只有两个资源的情况下，可以简单地在命令行上指定资源或者名字：

```
$ kubectl delete replicationcontrollers/my-nginx services/my-nginx-svc
```

为了处理更大量的资源，可以用Label来过滤资源。Selector用`-l`参数来指定：

```
$ kubectl delete all -lapp=nginx
replicationcontrollers/my-nginx
services/my-nginx-svc
```

因为`kubectl`用它可以接受的语法输出资源名字，用`$()`或者`xargs`就可以简单地把这些才做串起来：

```
$ kubectl get $(kubectl create -f ./nginx/ | grep my-nginx)
CONTROLLER   CONTAINER(S)   IMAGE(S)   SELECTOR    REPLICAS
my-nginx     nginx          nginx      app=nginx   2
NAME           LABELS      SELECTOR    IP(S)          PORT(S)
my-nginx-svc   app=nginx   app=nginx   10.0.152.174   80/TCP
```

##有效地使用Label

目前为止我们使用的例子，对任何资源来说，最多只有一个Label。有许多必须要使用多重Label的场景，来把不同的集合区分开。

比如，不同的应用
The examples we’ve used so far apply at most a single label to any resource. There are many scenarios where multiple labels should be used to distinguish sets from one another.

For instance, different applications would use different values for the app label, but a multi-tier application, such as the [guestbook example](http://kubernetes.io/v1.1/examples/guestbook/), would additionally need to distinguish each tier. The frontend could carry the following labels:

```
labels:
        app: guestbook
        tier: frontend
```

while the Redis master and slave would have different tier labels, and perhaps even an additional role label:

```
labels:
        app: guestbook
        tier: backend
        role: master
```

and 

```
labels:
        app: guestbook
        tier: backend
        role: slave
```

The labels allow us to slice and dice our resources along any dimension specified by a label:

```
$ kubectl create -f ./guestbook-fe.yaml -f ./redis-master.yaml -f ./redis-slave.yaml
replicationcontrollers/guestbook-fe
replicationcontrollers/guestbook-redis-master
replicationcontrollers/guestbook-redis-slave
$ kubectl get pods -Lapp -Ltier -Lrole
NAME                           READY     STATUS    RESTARTS   AGE       APP         TIER       ROLE
guestbook-fe-4nlpb             1/1       Running   0          1m        guestbook   frontend   <n/a>
guestbook-fe-ght6d             1/1       Running   0          1m        guestbook   frontend   <n/a>
guestbook-fe-jpy62             1/1       Running   0          1m        guestbook   frontend   <n/a>
guestbook-redis-master-5pg3b   1/1       Running   0          1m        guestbook   backend    master
guestbook-redis-slave-2q2yf    1/1       Running   0          1m        guestbook   backend    slave
guestbook-redis-slave-qgazl    1/1       Running   0          1m        guestbook   backend    slave
my-nginx-divi2                 1/1       Running   0          29m       nginx       <n/a>      <n/a>
my-nginx-o0ef1                 1/1       Running   0          29m       nginx       <n/a>      <n/a>
$ kubectl get pods -lapp=guestbook,role=slave
NAME                          READY     STATUS    RESTARTS   AGE
guestbook-redis-slave-2q2yf   1/1       Running   0          3m
guestbook-redis-slave-qgazl   1/1       Running   0          3m
```

##Canary deployments
Another scenario where multiple labels are needed is to distinguish deployments of different releases or configurations of the same component. For example, it is common practice to deploy a canary of a new application release (specified via image tag) side by side with the previous release so that the new release can receive live production traffic before fully rolling it out. For instance, a new release of the guestbook frontend might carry the following labels:

```
labels:
        app: guestbook
        tier: frontend
        track: canary
```

and the primary, stable release would have a different value of the track label, so that the sets of pods controlled by the two replication controllers would not overlap:

```
labels:
        app: guestbook
        tier: frontend
        track: stable
```

The frontend service would span both sets of replicas by selecting the common subset of their labels, omitting the track label:

```
selector:
     app: guestbook
     tier: frontend
```

##Updating labels
Sometimes existing pods and other resources need to be relabeled before creating new resources. This can be done with kubectl label. For example:

```
$ kubectl label pods -lapp=nginx tier=fe
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-9gw19   1/1       Running   0          14m
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-hayza   1/1       Running   0          13m
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-mde6m   1/1       Running   0          17m
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-sh6m8   1/1       Running   0          18m
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-wfof4   1/1       Running   0          16m
$ kubectl get pods -lapp=nginx -Ltier
NAME                READY     STATUS    RESTARTS   AGE       TIER
my-nginx-v4-9gw19   1/1       Running   0          15m       fe
my-nginx-v4-hayza   1/1       Running   0          14m       fe
my-nginx-v4-mde6m   1/1       Running   0          18m       fe
my-nginx-v4-sh6m8   1/1       Running   0          19m       fe
my-nginx-v4-wfof4   1/1       Running   0          16m       fe
```

##Scaling your application
When load on your application grows or shrinks, it’s easy to scale with kubectl. For instance, to increase the number of nginx replicas from 2 to 3, do:

```
$ kubectl scale rc my-nginx --replicas=3
scaled
$ kubectl get pods -lapp=nginx
NAME             READY     STATUS    RESTARTS   AGE
my-nginx-1jgkf   1/1       Running   0          3m
my-nginx-divi2   1/1       Running   0          1h
my-nginx-o0ef1   1/1       Running   0          1h
```

##Updating your application without a service outage
At some point, you’ll eventually need to update your deployed application, typically by specifying a new image or image tag, as in the canary deployment scenario above. kubectl supports several update operations, each of which is applicable to different scenarios.

To update a service without an outage, kubectl supports what is called [“rolling update”](http://kubernetes.io/v1.1/docs/user-guide/kubectl/kubectl_rolling-update.html), which updates one pod at a time, rather than taking down the entire service at the same time. See [the rolling update design document](http://kubernetes.io/v1.1/docs/design/simple-rolling-update.html) and [the example of rolling update](http://kubernetes.io/v1.1/docs/user-guide/update-demo/) for more information.

Let’s say you were running version 1.7.9 of nginx:

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

To update to version 1.9.1, you can use kubectl rolling-update --image:

```
$ kubectl rolling-update my-nginx --image=nginx:1.9.1
Creating my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
```

In another window, you can see that kubectl added a deployment label to the pods, whose value is a hash of the configuration, to distinguish the new pods from the old:

```
$ kubectl get pods -lapp=nginx -Ldeployment
NAME                                              READY     STATUS    RESTARTS   AGE       DEPLOYMENT
my-nginx-1jgkf                                    1/1       Running   0          1h        2d1d7a8f682934a254002b56404b813e
my-nginx-ccba8fbd8cc8160970f63f9a2696fc46-k156z   1/1       Running   0          1m        ccba8fbd8cc8160970f63f9a2696fc46
my-nginx-ccba8fbd8cc8160970f63f9a2696fc46-v95yh   1/1       Running   0          35s       ccba8fbd8cc8160970f63f9a2696fc46
my-nginx-divi2                                    1/1       Running   0          2h        2d1d7a8f682934a254002b56404b813e
my-nginx-o0ef1                                    1/1       Running   0          2h        2d1d7a8f682934a254002b56404b813e
my-nginx-q6all 
```

kubectl rolling-update reports progress as it progresses:

```
Updating my-nginx replicas: 4, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 1
At end of loop: my-nginx replicas: 4, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 1
At beginning of loop: my-nginx replicas: 3, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 2
Updating my-nginx replicas: 3, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 2
At end of loop: my-nginx replicas: 3, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 2
At beginning of loop: my-nginx replicas: 2, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 3
Updating my-nginx replicas: 2, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 3
At end of loop: my-nginx replicas: 2, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 3
At beginning of loop: my-nginx replicas: 1, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 4
Updating my-nginx replicas: 1, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 4
At end of loop: my-nginx replicas: 1, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 4
At beginning of loop: my-nginx replicas: 0, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 5
Updating my-nginx replicas: 0, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 5
At end of loop: my-nginx replicas: 0, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 5
Update succeeded. Deleting old controller: my-nginx
Renaming my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 to my-nginx
my-nginx
```

If you encounter a problem, you can stop the rolling update midway and revert to the previous version using --rollback:

```
$ kubectl kubectl rolling-update my-nginx  --image=nginx:1.9.1 --rollback
Found existing update in progress (my-nginx-ccba8fbd8cc8160970f63f9a2696fc46), resuming.
Found desired replicas.Continuing update with existing controller my-nginx.
Stopping my-nginx-02ca3e87d8685813dbe1f8c164a46f02 replicas: 1 -> 0
Update succeeded. Deleting my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
my-nginx
```

This is one example where the immutability of containers is a huge asset.

If you need to update more than just the image (e.g., command arguments, environment variables), you can create a new replication controller, with a new name and distinguishing label value, such as:

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx-v4
spec:
  replicas: 5
  selector:
    app: nginx
    deployment: v4
  template:
    metadata:
      labels:
        app: nginx
        deployment: v4
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.2
        args: [“nginx”,”-T”]
        ports:
        - containerPort: 80
```

and roll it out:

```
$ kubectl rolling-update my-nginx -f ./nginx-rc.yaml
Creating my-nginx-v4
At beginning of loop: my-nginx replicas: 4, my-nginx-v4 replicas: 1
Updating my-nginx replicas: 4, my-nginx-v4 replicas: 1
At end of loop: my-nginx replicas: 4, my-nginx-v4 replicas: 1
At beginning of loop: my-nginx replicas: 3, my-nginx-v4 replicas: 2
Updating my-nginx replicas: 3, my-nginx-v4 replicas: 2
At end of loop: my-nginx replicas: 3, my-nginx-v4 replicas: 2
At beginning of loop: my-nginx replicas: 2, my-nginx-v4 replicas: 3
Updating my-nginx replicas: 2, my-nginx-v4 replicas: 3
At end of loop: my-nginx replicas: 2, my-nginx-v4 replicas: 3
At beginning of loop: my-nginx replicas: 1, my-nginx-v4 replicas: 4
Updating my-nginx replicas: 1, my-nginx-v4 replicas: 4
At end of loop: my-nginx replicas: 1, my-nginx-v4 replicas: 4
At beginning of loop: my-nginx replicas: 0, my-nginx-v4 replicas: 5
Updating my-nginx replicas: 0, my-nginx-v4 replicas: 5
At end of loop: my-nginx replicas: 0, my-nginx-v4 replicas: 5
Update succeeded. Deleting my-nginx
my-nginx-v4
```

You can also run the [update demo](http://kubernetes.io/v1.1/docs/user-guide/update-demo/) to see a visual representation of the rolling update process.

##In-place updates of resources
Sometimes it’s necessary to make narrow, non-disruptive updates to resources you’ve created. For instance, you might want to add an [annotation](http://kubernetes.io/v1.1/docs/user-guide/annotations.html) with a description of your object. That’s easiest to do with kubectl patch:

```
$ kubectl patch rc my-nginx-v4 -p '{"metadata": {"annotations": {"description": "my frontend running nginx"}}}' 
my-nginx-v4
$ kubectl get rc my-nginx-v4 -o yaml
apiVersion: v1
kind: ReplicationController
metadata:
  annotations:
    description: my frontend running nginx
...
```

The patch is specified using json.

For more significant changes, you can get the resource, edit it, and then replace the resource with the updated version:

```
$ kubectl get rc my-nginx-v4 -o yaml > /tmp/nginx.yaml
$ vi /tmp/nginx.yaml
$ kubectl replace -f /tmp/nginx.yaml
replicationcontrollers/my-nginx-v4
$ rm $TMP
```

The system ensures that you don’t clobber changes made by other users or components by confirming that the resourceVersion doesn’t differ from the version you edited. If you want to update regardless of other changes, remove the resourceVersion field when you edit the resource. However, if you do this, don’t use your original configuration file as the source since additional fields most likely were set in the live state.

##Disruptive updates
In some cases, you may need to update resource fields that cannot be updated once initialized, or you may just want to make a recursive change immediately, such as to fix broken pods created by a replication controller. To change such fields, use replace --force, which deletes and re-creates the resource. In this case, you can simply modify your original configuration file:

```
$ kubectl replace -f ./nginx-rc.yaml --force
replicationcontrollers/my-nginx-v4
replicationcontrollers/my-nginx-v4
```

##What's next?
- [Learn about how to use kubectl for application introspection and debugging](http://kubernetes.io/v1.1/docs/user-guide/introspection-and-debugging.html).
- [Tips and tricks when working with config](http://kubernetes.io/v1.1/docs/user-guide/config-best-practices.html)


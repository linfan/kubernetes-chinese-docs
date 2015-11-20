# 管理应用： 在生产环境中使用Pods和容器

## 永久性存储

容器中的文件系统会随着容器的停止而消失；如果容器重启或出错停止，则会使得容器文件中的数据丢失，而且容器即使重新运行也会忘记之前的数据和状态。如果需要比容器的生命更为长久的存储方案，我们需要使用数据卷。这个需求对于有状态服务更为重要，例如 数据库，健－值存储等。

举例来说，Redis是一个健－值存储缓存（我们在[Guestbook](http://kubernetes.io/v1.1/examples/guestbook/)和一些其他的例子中使用过）。我们可以用如下方法为它加一个数据卷：

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis
spec:
  template:
    metadata:
      labels:
        app: redis
        tier: backend
    spec:
      # 为Pod提供一个数据卷
      volumes:
        - name: data
          emptyDir: {}
      containers:
      - name: redis
        image: kubernetes/redis:v1
        ports:
        - containerPort: 6379
        # 将数据卷加载到Pod中
        volumeMounts:
        - mountPath: /redis-master-data
          name: data   # 必须和上面定义的数据卷名字匹配
```

`emptyDir`数据卷的生命周期与Pod的生命周期一致，因此会比任何一个容器更长久；如果某个容器失效或重启了，我们的数据还会继续存在。

除了本地磁盘所支持的`emptyDir`，Kubernetes还支持多种网络存储方案，包括GCE的PD，EC2的EBS。这些存储方案更适合用来存储关键数据，同时还能处理在节点上动态加载，卸载数据卷。详情请查看[数据卷文档](http://kubernetes.io/v1.0/docs/user-guide/volumes.html)。

## 分发认证信息

很多应用需要认证信息，例如密码，OAuth令牌，和TLS的秘钥等，用来和其他的应用、数据库、服务等进行认证。在容器镜像中或是通过环境变量来存储这些私密信息并不理想，因为只要能够访问这些镜像，或是Pod/容器配置，或是宿主机的文件系统，或是Docker Daemon，一个人就可以获取这些私密信息。

Kubernetes提供一个叫做秘密的机制用来帮助分发敏感的认证信息。一个秘密是一种含有map数据的资源。举例来说，一个含有用户名和密码的秘密可以定义如下：

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: dmFsdWUtMg0K
  username: dmFsdWUtMQ0K
```

如同其它资源一样，这个秘密可以通过`create`命令来生成，同时可以通过`get`命令来查看：

```
$ kubectl create -f ./secret.yaml
secrets/mysecret
$ kubectl get secrets
NAME                  TYPE                                  DATA
default-token-v9pyz   kubernetes.io/service-account-token   2
mysecret              Opaque                                2
```

要使用秘密，我们需要在一个pod的模版中引用它。`secret`数据卷可以让我们像内存目录一样来把秘密加载到容器里。

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis
spec:
  template:
    metadata:
      labels:
        app: redis
        tier: backend
    spec:
      volumes:
        - name: data
          emptyDir: {}
        - name: supersecret
          secret:
            secretName: mysecret
      containers:
      - name: redis
        image: kubernetes/redis:v1
        ports:
        - containerPort: 6379
        # Mount the volume into the pod
        volumeMounts:
        - mountPath: /redis-master-data
          name: data   # must match the name of the volume, above
        - mountPath: /var/run/secrets/super
          name: supersecret
```

如果要查看更多详情，可以参考关于秘密的详细[文档](http://kubernetes.io/v1.0/docs/user-guide/secrets.html)，[例子](http://kubernetes.io/v1.0/docs/user-guide/secrets/)和[设计](http://kubernetes.io/v1.0/docs/design/secrets.html)。

## 与私有镜像仓库进行认证

我们还可以利用秘密来传递私有镜像仓库的认证信息。

第一步，我们生成一个`.dockercfg`文件（例如可以通过运行`docker login <registry.domain>`命令）。然后，我们将生成的`.dockercfg`文件放到一个秘密资源里，例如：

```
$ docker login
Username: janedoe
Password: ●●●●●●●●●●●
Email: jdoe@example.com
WARNING: login credentials saved in /Users/jdoe/.dockercfg.
Login Succeeded

$ echo $(cat ~/.dockercfg)
{ "https://index.docker.io/v1/": { "auth": "ZmFrZXBhc3N3b3JkMTIK", "email": "jdoe@example.com" } }

$ cat ~/.dockercfg | base64
eyAiaHR0cHM6Ly9pbmRleC5kb2NrZXIuaW8vdjEvIjogeyAiYXV0aCI6ICJabUZyWlhCaGMzTjNiM0prTVRJSyIsICJlbWFpbCI6ICJqZG9lQGV4YW1wbGUuY29tIiB9IH0K

$ cat > /tmp/image-pull-secret.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: myregistrykey
data:
  .dockercfg: eyAiaHR0cHM6Ly9pbmRleC5kb2NrZXIuaW8vdjEvIjogeyAiYXV0aCI6ICJabUZyWlhCaGMzTjNiM0prTVRJSyIsICJlbWFpbCI6ICJqZG9lQGV4YW1wbGUuY29tIiB9IH0K
type: kubernetes.io/dockercfg
EOF

$ kubectl create -f ./image-pull-secret.yaml
secrets/myregistrykey
```

现在你就可以生成一个pod，并通过添加`imagePullSecrets`部分来引用上述生成的秘密：

```
apiVersion: v1
kind: Pod
metadata:
  name: foo
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey
```

## 协同容器

Pods支持将多个容器一起调度，保证它们运行在同一个宿主机上。他们可以被用来支持一整个应用栈，但主要应用场景是用来运行辅助程序。经典的例子包括数据发布、获取程序，代理等等。

这些容器往往需要通过文件系统相互通信，而这个通信过程可以通过在两个容器中都加入同样的数据卷来实现。这个模式的一个例子就是网络服务器通过git存储库来等待程序最近更新：

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: www-data
        emptyDir: {}
      containers:
      - name: nginx
        image: nginx
        # This container reads from the www-data volume
        volumeMounts:
        - mountPath: /srv/www
          name: www-data
          readOnly: true
      - name: git-monitor
        image: myrepo/git-monitor
        env:
        - name: GIT_REPO
          value: http://github.com/some/repo.git
        # This container writes to the www-data volume
        volumeMounts:
        - mountPath: /data
          name: www-data
```
更多例子请查看我们的[博客文章](http://blog.kubernetes.io/2015/06/the-distributed-system-toolkit-patterns.html)以及[演示文稿幻灯片](http://www.slideshare.net/Docker/slideshare-burns)。


## 资源管理

只有在有足够的CPU和存储的地方Kubernetes的调度器才会放置应用程序，但在知道应用程序需要多少资源的时候也只能这么做。指定的CPU太小的后果就是，如果太多其他容器被安排到同一个节点的话，CPU资源会被耗尽。同样地，容器也可能会由于在耗尽存储要不到其他内存的情况下而不可预见的终止运行，对于大内存的应用程序尤其可能出现这种情况。

如果没有指定资源需求，那么资源数量在名义上是被假定的（这个默认值是通过Limitrange给默认值的命名空间设定的）。它也可以通过｀kubectl desribe limitrange limits`被查看。）我们可以明确地把要求的资源数量列举如下：

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
        resources:
          limits:
            # cpu units are cores
            cpu: 500m
            # memory units are bytes
            memory: 64Mi
```

容器的运行超过指定限度的时候会因为内存耗尽而终止程序，所以制定一个略高于预期的值提高了总体的可信度。

如果不确定该请求多少资源，那么我们可以先不指定启动应用程序资源，然后根据资源使用情况量监控来确定适当的值。


## 健康检查

许多应用程序经过长时间的运行，最终过渡到了无法运行的状态，除了重启，无法恢复。Kubernetes提供活性探针来检测并且对相应状况进行相应的补救措施。

通常来检测一个应用程序的方法是用HTTP，详细说明如下：

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
        livenessProbe:
          httpGet:
            # Path to probe; should be cheap, but representative of typical behavior
            path: /index.html
            port: 80
          initialDelaySeconds: 30
          timeoutSeconds: 1
```

很多时候，应用程序只是暂时无法服务，之后会自己恢复。通常在这样的情况下，我们不愿意关闭应用程序，但是也不想发送请求，因为应用层程序不正确回应或者根本不回应。常见的这样的场景是加载大数据或者在应用程序启动期间配置文件。Kubernetes提供探针来检测和缓解这种情况。探针的配置只是用到`readinessProbe`字段。由容器组成的pod报告说他们还没准备好通过Kubernetes服务接收通信量。

要知道更多细节（比如：如何指定命令基础上的调查），请查看[walkthrough](http://kubernetes.io/v1.0/docs/user-guide/walkthrough/k8s201.html#health-checking)里的例子, [standalone](http://kubernetes.io/v1.0/docs/user-guide/liveness/)的例子, 还有[文档](http://kubernetes.io/v1.0/docs/user-guide/pod-states.html#container-probes)。

## 生命周期插件接口和终止通知

当然，节点和应用程序在任何时候都可能会运行失败，但是当应用程序的终止被人为下达的时候，很多应用程序受益于正常关机来完成正在处理的请求。为了支持这样的情况，Kubernetes支持两种信息通知：Kubernetes将发送终止信号给应用程序，这种信号可以被接收到来平滑的终止程序。如果应用程序没有尽早终止，无条件终止命令将会在10秒后发送。Kubernetes支持（可选择的）预停止生命周期插件接口，这个动作执行在发送终止信号之前。

预停止插件接口的细节跟调查细节相似，但是没有时限相关参数。例如：

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
        lifecycle:
          preStop:
            exec:
              # SIGTERM triggers a quick exit; gracefully terminate instead
              command: ["/usr/sbin/nginx","-s","quit"]
```

## 终止信息

为了达到一个相当高水平的实用性，特别是为了积极开发应用，快速调试失败是很重要的。除了一般的日志采集，Kubernetes还能通过查出重大错误原因来加速调试，并在某种程度上通过kubectl或者UI陈列出来。可以指定一个’terminationMessagePath’来让容器写下它的“death rattle“，比如声明失败消息，堆栈跟踪，免责条款等等。默认途径是‘/dev/termination-log’。

这是一个小例子：

```

apiVersion: v1
kind: Pod
metadata:
  name: pod-w-message
spec:
  containers:
  - name: messager
    image: "ubuntu:14.04"
    command: ["/bin/sh","-c"]
    args: ["sleep 60 && /bin/echo Sleep expired > /dev/termination-log"]
```

消息和上一个（也就是：最近的）终止的其他状态是一起被记录的。

```
$ kubectl create -f ./pod.yaml
pods/pod-w-message
$ sleep 70
$ kubectl get pods/pod-w-message -o template -t "{{range .status.containerStatuses}}{{.lastState.terminated.message}}{{end}}"
Sleep expired
$ kubectl get pods/pod-w-message -o template -t "{{range .status.containerStatuses}}{{.lastState.terminated.exitCode}}{{end}}"
0
```
















# 配置kubernetes


除了像`kubectl run`和`kubectl expose`这些必要的命令，kubernetes也支持可声明式的配置。配置文件也需要必要的命令，这样就可以在代码审查中检查版本控制和文件改动，而代码审查对复杂的具有鲁棒性的可靠生产系统是非常重要的。

在声明式风格中，所有的配置都保存在YAML或者JSON配置文件中，使用Kubernetes的API资源模式（schema）作为配置的模式（schema）。`kubectl`命令可以创建、更新、删除以及获取API资源。`kubectl`命令用`ApiVersion`（目前是“v1”），`kind`资源，`name`资源去创建合适的API路径来执行特殊的操作。


# 使用配置文件创建一个容器

Kubernetes是在Pod中来运行容器的。一个包含了一个简单的HellWorld容器的Pod可以被如下的YAML文件指定：
```
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:  # specification of the pod’s contents
  restartPolicy: Never
  containers:
  - name: hello
    image: "ubuntu:14.04"
    command: ["/bin/echo","hello”,”world"]
```


`metadata.name的`值`hello-world`，将会成为创建成功后Pod的名称，这个名称必须在集群中唯一，而`container[0].name`只是容器在Pod中的昵称。`image`就是Docker image的名称且Kubernetes默认会从Docker Hub中拉取镜像。

`restartPolicy`: `Never`指明了我们只是想运行容器一次然后就终止Pod。

`Command`覆盖了docker容器的`Entrypoint`。命令的参数（相当于Docker的`Cmd`）可以指定`args`参数，如下所示：

```
command: ["/bin/echo"]
args: ["hello","world"]
```
创建这个pod就可以使用`create`命令了

```
$ kubectl create -f ./hello-world.yaml
pods/hello-world
```
当成功创建时，`kubectl`打印出资源类型和资源名称。


# 配置验证

如果你不确定指定的资源是否正确，你可以`kubectl`帮你验证。
```
$ kubectl create -f ./hello-world.yaml --validate
```
假设我们指定的是`entrypoint`而不是`command`，你会看到如下输出：
```
I0709 06:33:05.600829   14160 schema.go:126] unknown field: entrypoint
I0709 06:33:05.600988   14160 schema.go:129] this may be a false alarm, 
see https://github.com/GoogleCloudPlatform/kubernetes/issues/6842
pods/hello-world
```
`kubectl create –validate`会警告已经检测出问题，除非缺少必须的字段或者字段值不合法，最后kubectl还是会创建出资源。一定要小心，未知的API字段会被忽略。这个pod没有`command`字段而被创建，command字段是一个可选字段，因为image镜像可以指定`Entrypoint`。访问Pod API object来查看合法字段列表。


# 环境变量和增加变量

Kubernetes没有自动的在shell中的运行命令（不是所有镜像都有shell）。如果你想在shell中运行你的命令，例如增加环境便令（使用env字段），你可以按如下做法：
```
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:  # specification of the pod’s contents
  restartPolicy: Never
  containers:
  - name: hello
    image: "ubuntu:14.04"
    env:
    - name: MESSAGE
      value: "hello world"
    command: ["/bin/sh","-c"]
    args: ["/bin/echo \"${MESSAGE}\""]

```
然而，一个shell需要的不仅是增加环境变量。如果你使用$(ENVVAR) 语法Kubernetes将会为你做这些事。

# 查看pod状态

使用get命令，你可以看到你已经创建的pod（事实上是集群中你的所有pod）。如果你在创建后用足够快的速度输入get命令，你会看到下面的内容：
```
$ kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
hello-world   0/1       Pending   0          0s

```
初始化的时候，新创建的pod还未被调度，也就是还没有节点被选中去运行它。pod创建后会进行调度但是它很快，所以你正常是不会看到pods处于未被调度的状态，除非有问题产生。

在pod被调度之后，如果节点中还没有镜像那么就会先通过docker拉取响应的镜像。一切就绪后，你会看到容器在运行：
```
$ kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
hello-world   1/1       Running   0          5s

```
Ready列指示正在运行在pod中的容器数量。

而开始运行后很快这个容器将会被终止。Kubectl会显示容器已经不再运行以及推出状态：
```
$ kubectl get pods
NAME          READY     STATUS       RESTARTS   AGE
hello-world   0/1       ExitCode:0   0          15s

```

# 查看pod输出

你会想要查看你运行命令的输出。像docker logs一样，kubectl logs也会显示输出：
```
$ kubectl logs hello-world
hello world

```

# 删除pods

当你看完的输出，你应该删除pod：
```
$ kubectl delete pod hello-world
pods/hello-world

```
就像create一样，删除成功时kubectl也会打印出资源类型和资源名称。

你也可以使用 资源/名称格式指定一个pod：
```
$ kubectl delete pods/hello-world
pods/hello-world

```
终止pods不会自动的删除，你可以观察他们的最终状态，所以请确定清理了你的已经结束的pods。

在另一方面，为了在节点中释放磁盘空间容器和他们的日志也会被自动删除。

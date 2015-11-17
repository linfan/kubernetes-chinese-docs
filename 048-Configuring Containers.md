
# 配置kubernetes


除了像kubectl run和kubectl expose这些必要的命令，kubernetes也支持可声明式的配置。配置文件也需要必要的命令，这样就可以在代码审查中检查版本控制和文件改动，而代码审查对复杂的具有鲁棒性的可靠生产系统是非常重要的。

在声明式风格中，所有的配置都保存在YAML或者JSON配置文件中，使用Kubernetes的API资源模式（schema）作为配置的模式（schema）。Kubectl命令可以创建、更新、删除以及获取API资源。kubectl命令用ApiVersion（目前是“v1”），kind资源，name资源去创建合适的API路径来执行特殊的操作。


# 使用配置文件创建一个容器

Kubernetes是在Pod中来运行容器的。一个包含了一个简单的HellWorld容器的Pod可以被如下的YAML文件指定：

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


metadata.name的值hello-world，将会成为创建成功后Pod的名称，这个名称必须在集群中唯一，而container[0].name只是容器在Pod中的昵称。image就是Docker image的名称且Kubernetes默认会从Docker Hub中拉取镜像。
restartPolicy: Never指明了我们只是想运行容器一次然后就终止Pod。
Command覆盖了docker容器的Entrypoint。命令的参数（相当于Docker的Cmd）可以指定为args参数，如下所示：
command: ["/bin/echo"]
    args: ["hello","world"]
创建这个pod就可以使用create命令了
$ kubectl create -f ./hello-world.yaml
pods/hello-world
当成功创建时，kubectl打印出资源类型和资源名称。

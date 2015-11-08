# **Kubernetes 101 - Kubectl CLI和Pods**

Kubernetes 101 - Kubectl CLI and Pods

对于Kubernetes 101，我们将覆盖的内容包括kubectl, pods, volumes?, 以及多容器。

为了kubectl使用率示例可以正常运行，请确保你已经在本地有示例目录，可以从release中或者从代码中得到的。

内容列表
•Kubernetes 101 - Kubectl CLI 和 Pods •Kubectl CLI
•Pods •Pod 定义
•Pod 管理
•Volumes
•Volume 类型
•Multiple Containers多容器

•下面是什么?


Kubectl CLI

同Kubernetes交互的最简单方法是通过kubectl命令行界面

对于kubectl的更多信息，包括它的使用率，命令，参数，请阅读kubectl CLI参考手册

如果你没有安装好或者配置好kubectl，在继续阅读之前把这件事完成。

Pods

在Kubernetes，一组一个或者多个容器叫做pod。Pod中的容器是部署在一起的，并且作为一组来启动，停止和复制。

点击pods链接得到更多细节信息。

Pod定义

最简单的pod定义描述了单个容器的部署。比如，一个nginx web服务器的pod可以定义如下：
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

Pod的定义是一种预期状态（Desired State）的声明。预期状态是Kubernetes模型中非常重要的概念。很多事情在系统中体现为预期状态，而Kubernetes有责任确保当前状态匹配预期状态。举个例子，当你创建一个Pod，你指明其中的容器进入运行。如果容器没有运行（例如程序错误，...），为了驱使容器进入预期状态，Kubernetes将持续为你（再）创建这些容器。这个过程将一直持续到该Pod被删除。

获取更多细节信息，请查阅设计文档。

Pod管理

创建一个包含nginx server的pod（pod-nginx.yaml）:

$ kubectl create -f docs/user-guide/walkthrough/pod-nginx.yaml

列出所有pods:

$ kubectl get pods

在Pod部署的大部分环境里，Pod的IP都是外部不可接入的。最便捷的测试Pod是否工作的方法是创建一个busybox pod（？）并且在上面远程运行命令。请查看可执行命令文档以找到更多细节。

如果Pod IP可以访问，你应该可以利用curl访问在80端口访问http端点。

$ curl http://$(kubectl get pod nginx -o=template -t={{.status.podIP}})

通过名字删除pod：

$ kubectl delete pod nginx

Volumes

这对于一个简单的静态网站服务器很不错，那么对于需要持续存储情况如何？

容器文件系统仅仅存在于容器的生存周期。所以，如果你的应用状态需要忍受迁移、重启和崩溃，你需要配置一些持续性存储。

For this example we'll be creating a Redis pod with a named volume and volume mount that defines the path to mount the volume.
1.Define a volume:
在下面的例子中，我们将创建一个Redis Pod，这个Pod包括一个已命名的Volume和包含Volume安装路径的Volume安装点。
volumes:
    - name: redis-persistent-storage
      emptyDir: {}
      
1. 在容器定义内定义一个Volume安装点

volumeMounts:
    # name must match the volume name below
    - name: redis-persistent-storage
      # mount path within the container
      mountPath: /data/redis

带有持续存储Volume的Redis Pod定义举例（pod-redis.yaml）：

apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-persistent-storage
      mountPath: /data/redis
  volumes:
  - name: redis-persistent-storage
    emptyDir: {}

Notes:
•The volume mount name is a reference to a specific empty dir volume.
•The volume mount path is the path to mount the empty dir volume within the container.

Volume Types
•EmptyDir: Creates a new directory that will persist across container failures and restarts.
•HostPath: Mounts an existing directory on the node's file system (e.g. /var/logs).

See volumes for more details.

Multiple Containers

Note: The examples below are syntactically correct, but some of the images (e.g. kubernetes/git-monitor) don't exist yet. We're working on turning these into working examples.

However, often you want to have two different containers that work together. An example of this would be a web server, and a helper job that polls a git repository for new updates:

apiVersion: v1
kind: Pod
metadata:
  name: www
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /srv/www
      name: www-data
      readOnly: true
  - name: git-monitor
    image: kubernetes/git-monitor
    env:
    - name: GIT_REPO
      value: http://github.com/some/repo.git
    volumeMounts:
    - mountPath: /data
      name: www-data
  volumes:
  - name: www-data
    emptyDir: {}

Note that we have also added a volume here. In this case, the volume is mounted into both containers. It is marked readOnly in the web server's case, since it doesn't need to write to the directory.

Finally, we have also introduced an environment variable to the git-monitor container, which allows us to parameterize that container with the particular git repository that we want to track.

What's Next?

Continue on to Kubernetes 201 or for a complete application see the guestbook example

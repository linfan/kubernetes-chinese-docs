# **Kubernetes 101 - Kubectl CLI和Pods**

Kubernetes 101 - Kubectl CLI and Pods

对于Kubernetes 101，我们将覆盖的内容包括kubectl, pods, volumes, 以及多容器。

为了kubectl使用率示例可以正常运行，请确保你已经在本地有示例目录，可以从https://github.com/kubernetes/kubernetes/releases中或者从https://github.com/kubernetes/kubernetes中得到的。

内容列表
•Kubernetes 101 - Kubectl CLI 和 Pods 
    •Kubectl CLI
    •Pods 
        •Pod 定义
        •Pod 管理
    •Volumes
        •Volume 类型
    •多容器

    •下面是什么?


Kubectl CLI

同Kubernetes交互的最简单方法是通过kubectl命令行界面

对于kubectl的更多信息，包括它的使用率，命令，参数，请阅读kubectl CLI参考手册http://kubernetes.io/v1.1/docs/user-guide/kubectl/kubectl.html。

如果你没有安装好或者配置好kubectl，在继续阅读之前按照http://kubernetes.io/v1.1/docs/user-guide/prereqs.html把预置条件完成。

Pods

在Kubernetes，一组单个或者多个容器叫做pod。Pod中的容器是部署在一起的，并且作为一组来启动，停止和复制。

点击pods链接http://kubernetes.io/v1.1/docs/user-guide/pods.html得到更多细节信息。

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

Pod的定义声明了某个预期状态（Desired State）。预期状态是Kubernetes模型中非常重要的概念。很多事情在系统中体现为预期状态，而Kubernetes有责任确保当前状态匹配预期状态。举个例子，当你创建一个Pod，你指明其中的容器进入运行。如果容器没有运行（例如程序错误，...），为了驱使容器进入预期状态，Kubernetes将持续为你（再）创建这些容器。这个过程将一直持续到该Pod被删除。

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
注：
•volume安装点名字是一个纸箱一个特定空目录volume的指针。
•volume安装目录是容器内安装特定空目录volume的路径。

Volume 类型
•EmptyDir：创建一个在容器失效和重启情况下可以持续的新目录
•HostPath：将已经存在的目录安装在节点文件系统上 （e.g. /var/logs）.

查阅 volumes 可以得到更多细节。

多容器

注：下面的例子在语义上正确，但是某些图片（比如 kubernetes/git-monitor）目前还不存在。我们正在努力将这些内容加入到可以工作的例子中。

However, often you want to have two different containers that work together. An example of this would be a web server, and a helper job that polls a git repository for new updates:
然而，通常你会希望存在两种容器在一起工作。一个例子是网站服务器，和一个可以从git仓库中拉出更新的帮助任务。

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
注意我们也在这里增加了一个volume。在这个例子里，这个volume同时被安装在两个容器中。在网页服务器容器中，由于并不需要写这个目录，因此被标注为只读。

Finally, we have also introduced an environment variable to the git-monitor container, which allows us to parameterize that container with the particular git repository that we want to track.
最后，我们也引入了一个给git-monitor容器使用的环境变量，这个变量允许我们将我们希望跟踪的特定git仓库作为参数使用

What's Next?
下面是什么？

Continue on to Kubernetes 201 or for a complete application see the guestbook example
继续学习Kubernetes 201或者阅读guestbook example，这可以得到一个完整的用例。

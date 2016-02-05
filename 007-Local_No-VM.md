#从本地运行k8s开始 v1.0
`译者：卢文泉` `校对：无`

[原文地址](http://kubernetes.io/v1.0/docs/getting-started-guides/locally.html)

###环境需求
####Linux
没有运行Linux？考虑下使用[Vagrant](http://kubernetes.io/v1.0/docs/getting-started-guides/vagrant.html)在虚拟机中运行Linux，或者像[Google Compute Engine](http://kubernetes.io/v1.0/docs/getting-started-guides/gce.html)这样的云提供商上运行。

####Docker
至少Docker1.3+。确保Docker守护进程一直运行，并确保能交互（比如`docker ps`）。一些Kubernetes组件需要root权限运行，这样这些组件才能和Docker正常、良好地工作。

####etcd
你需要配置[etcd](https://github.com/coreos/etcd/releases)到环境变量，请确保安装etcd并且正确配置到**$PATH**中。

####go
go版本至少1.3+，请确保安装好go并配置好**￥PATH**。

###启动集群
新打开一个单独的终端，运行下面的指令（由于启动/停止kubernetes守护进程需要root权限，所以使用root权限运行整个脚本会使操作更加容易）：
```sh
cd kubernetes
hack/local-up-cluster.sh
```
这个操作将会构建和启动一个轻量的本地集群，包含一个master和一个节点。输入ctrl+C关闭集群。

##运行容器
成功运行集群后，我想你很定迫不及待地想启动你的容器了。

现在你可以使用`cluster/kubectl.sh`脚本中的命令来和本地集群交互了：
```sh
cluster/kubectl.sh get pods
cluster/kubectl.sh get services
cluster/kubectl.sh get replicationcontrollers
cluster/kubectl.sh run my-nginx --image=nginx --replicas=2 --port=80

##在等待命令完成前，你可以打开一个新终端查看docker拉取镜像
  sudo docker images
  ##你会看到docker正在拉去nginx镜像
  sudo docker ps
  ## 你会看到你的容器正在运行
  exit
## end wait

## 查看kubernetes相关信息
cluster/kubectl.sh get pods
cluster/kubectl.sh get services
cluster/kubectl.sh get replicationcontrollers
```

##运行用户定义的pod
要注意[容器](http://kubernetes.io/v1.0/docs/user-guide/containers.html)和[pod](http://kubernetes.io/v1.0/docs/user-guide/pods.html)之前的不同。如果你只向kubernetes请求前者（容器），kubernetes会创建一个新的封装好的pod给你。但是（通过这个pod）你不能在本地主机查看到nginx的开始页面。为了验证nginx正确在容器中运行，你需要在容器中运行`curl`（通过docker exec执行）。

你可以通过用户定义的`manifest`来控制pod的信息。指定端口就可以在浏览器中访问nginx了：
```sh
cluster/kubectl.sh create -f docs/user-guide/pod.yaml
```

祝你好运

##解决问题
###我不能通过IP访问服务
一些使用`iptables`工具的防火墙软件不能很好地与kubernetes配合。如果你在网络上遇到麻烦，首先尝试关闭系统的防火墙或者其它使用`iptables`的系统。此外，通过`journalctl --since yesterday | grep avc`指令检查SELinux（【译者注】指安全增强型Linux系统）是否屏蔽了什么。

集群IPs默认范围：`10.0...`，这有可能引起docker 容器IP和集群IP冲突。如果你发现容器IP也在集群的IP范围内，编辑`hack/local-cluster-up.sh`脚本修改集群的IP范围。

###当副本控制器的副本数大于1时我无法创建副本！哪里出了问题？
（也许是）你只运行一个node节点。指超过了一个给定pod支持的最大副本数。如果你对运行更大副本书感兴趣，我们鼓励你使用vagrant或者在云上操作。

###我修改Kubernetes代码后该如何运行它？
```language
cd kubernetes
hack/build-go.sh
hack/local-up-cluster.sh
```

###kubectl表明启动容器但是`get pod`和`docker ps`命令没有显示
本地`local-up-cluster.sh`脚本不会启动DNS服务。类似的解决方案见[这里](https://github.com/GoogleCloudPlatform/kubernetes/issues/6667)。或者你可以手动启动。相关文档见[这里](https://releases.k8s.io/v1.0.6/cluster/addons/dns#how-do-i-configure-it)







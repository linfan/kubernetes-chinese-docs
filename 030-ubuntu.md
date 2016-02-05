# 在Ubuntu物理节点上部署Kubernets
`译者：王乐` `校对：无`


## 介绍
这片文档介绍了如何在Ubuntu节点上部署Kubernetes，这里我们用1个主节点和3个普通节点的安装来作为范例。你可以轻松变动设置扩展到**任意数量的节点**。最初的想法是受到@jainvipin的Ubuntu单节点部署工作的启发。单节点的部署介绍也涵盖在本章节中。

[浙江大学云团队](https://github.com/ZJU-SEL)会维护这项工作。

## 前提条件
1. 所有节点上已经安装docker版本1.2+和用来控制Linux网桥的bridge-utils。
2. 所有的机器可相互通信。主节点需要连接到Interent去下载必须的文件。
3. 本指南介绍的步骤已经在Ubuntu 14.04 LTS 64bit server上测试过了。但在Ubuntu 15不能正常工作，这是因为Ubuntu 15使用systemd代替了upstart。
4. 本指南所依赖于etcd-2.0.12, flannel-0.5.3和k8s-1.0.6。或许能兼容这些软件的较高版本。
5. 所有的服务器能够使用ssh远程密钥认证登入，而不是用密码登入。

## 开始建立一个集群

### 建立正确的目录
复制Github上kubernetes的文件库到本地。

``` console
$ git clone https://github.com/kubernetes/kubernetes.git
```

#### 配置和运行Kubernetes集群
启动过程将会首先自动下载所需的二进制安装文件。默认会下载etc版本2.0.12，Flannel版本0.5.3和k8s版本1.0.6。你可以按一下方法来设置参数`ETCD_VERSION`，`FLANNEL_VERSION`和`KUBE_VERSION`，从而来选定你想要的etc，Flannel和k8s版本。

```console
$ export KUBE_VERSION=1.0.5
$ export FLANNEL_VERSION=0.5.0
$ export ETCD_VERSION=2.2.0
```

请注意我们在这里使用Flannel是用来建立overlay网络，但这并不是必须的。事实上你可以不借助其他工具直接建立k8s集群，或者使用Flannel，Open vSwitch或SDN来建立网络。

这是一个集群IP地址配置的例子：

| IP Address  |   Role   |
|-------------|----------|
|10.10.103.223|   node   |
|10.10.103.162|   node   |
|10.10.103.250| both master and node|

首先cluster/ubuntu/config-default.sh文件内配置集群信息。
以下是个简单的参考。

```sh
export nodes="vcap@10.10.103.250 vcap@10.10.103.162 vcap@10.10.103.223"

export role="ai i i"

export NUM_MINIONS=${NUM_MINIONS:-3}

export SERVICE_CLUSTER_IP_RANGE=192.168.3.0/24

export FLANNEL_NET=172.16.0.0/16
```
第一个参数`nodes`定义了你所有的节点。主节点列在第一位并使用空格来做分割，比如`<user_1@ip_1> <user_2@ip_2> <user_3@ip_3> `

接下来参数`role`按顺序定义了主机的角色，"ai”代表主机可以是主节点或普通节点，"a"代表主节点，"i"代表普通节点。

参数`NUM_MINIONS`定义了所有节点的数量。

参数`SERVICE_CLUSTER_IP_RANGE`定义了Kubernetes服务的IP地址范围。

请确保你所定义的私有IP地址范围是合理的，因为一些IaaS提供商会保留一些私有IP地址。根据RFC1918，一共有三个私有IP地址网段，如下所列。你最好不要选址一个和你自己私有地址网段冲突的网段。

     10.0.0.0        -   10.255.255.255  (10/8 prefix)

     172.16.0.0      -   172.31.255.255  (172.16/12 prefix)

     192.168.0.0     -   192.168.255.255 (192.168/16 prefix)


参数`FLANNEL_NET`定义了Flannel网络所用的IP地址范围。这个地址不能和`SERVICE_CLUSTER_IP_RANGE`的地址冲突。

**注意:** 在部署时，主节点需要连接到Interent去下载必须的文件。如果你的主机在私有网络里，你可以在cluster/ubuntu/config-default.sh中设置参数`PROXY_SETTING`来配置Internet代理。
     PROXY_SETTING="http_proxy=http://server:port https_proxy=https://server:port"

当以上所有参数正确设置后，你就可以用`cluster/`中的命令来启动整个集群了。

`$ KUBERNETES_PROVIDER=ubuntu ./kube-up.sh`
这个脚本自动`scp`传输二进制安装和配置文件到所有的主机上。之后启动这个主机上的kubernetes的服务。你唯一需要做的是当有提示的时候输入所需密码。

```console
Deploying node on machine 10.10.103.223
...
[sudo] password to start node: 
```
如果一切运行正常，k8s集群启动后你会看见以下提示信息。

```console
Cluster validation succeeded
```

### 测试
你可以运行`kubectl`命令来检测刚升级的Kubernetes集群是否工作正常。`kubectl`运行文件放置在`cluster/ubuntu/binaries`文件夹中。你也可以设置环境变量PATH来调用`kubectl`。

比如，使用`$ kubectl get nodes`来检测你的节点是否正常运行。
```console
$ kubectl get nodes
NAME            LABELS                                 STATUS
10.10.103.162   kubernetes.io/hostname=10.10.103.162   Ready
10.10.103.223   kubernetes.io/hostname=10.10.103.223   Ready
10.10.103.250   kubernetes.io/hostname=10.10.103.250   Ready
```

你也可以使用Kubernetes[guest-example](../../examples/guestbook/)来建立Redis后台集群。

### 部署插件
假设你开始运行k8s集群，这一节将会介绍如何在现有集群上部署类似DNS和UI等插件。

可以`cluster/ubuntu/config-default.sh`中配置在DNS。

```sh
ENABLE_CLUSTER_DNS="${KUBE_ENABLE_CLUSTER_DNS:-true}"

DNS_SERVER_IP="192.168.3.10"

DNS_DOMAIN="cluster.local"

DNS_REPLICAS=1
```

参数`DNS_SERVER_IP`定义了DNS的IP，这个IP必须在`SERVICE_CLUSTER_IP_RANGE`的范围中。
参数`DNS_REPLICAS`描述了集群中运行了多少个DNS pod。

默认情况下，我额们也可以配置kube-ui插件。

```sh
ENABLE_CLUSTER_UI="${KUBE_ENABLE_CLUSTER_UI:-true}"
```
当以上参数设置完毕之后，运行以下命令。
```console
$ cd cluster/ubuntu
$ KUBERNETES_PROVIDER=ubuntu ./deployAddons.sh
```
稍等之后，你可以用命令`$ kubectl get pods --namespace=kube-system`来检测这个集群中的DNS和pod的UI是否运行。

### 下一步
我们目前的工作集中在以下的功能：
1.使用[kube-in-docker](https://github.com/ZJU-SEL/kube-in-docker/tree/baremetal-kube)在Docker中运行kubernetes，从而消除不同操作系统的区别。
2.拆除部署脚本：一键式清除和重建整个部署。

### 故障排除
通常，这一步非常简单：
1.下载和复制安装和配置文件到在每个节点上。
2.根据用户的输入的IP来配置主节点上的`etcd`。
3.在每个普通节点上新建和启动Flannel网络。

如果你遇到什么问题，首先检查主节点上`etcd`的配置。
1.查看`/var/log/upstart/etcd.log`中的日志。
2.以下命令或许有帮助，第一个是停止这个集群，第二个是启动这个集群。

```console
$ KUBERNETES_PROVIDER=ubuntu ./kube-down.sh
$ KUBERNETES_PROVIDER=ubuntu ./kube-up.sh
```
3.你也可以在`/etc/default/{component_name}`里做客制化设置，之后用以下命令重启服务
`$ sudo service {component_name} restart`。

## 升级集群
如果你已经有了Kubernetes集群并希望升级到新的版本，你需要根据`cluster/`文件夹中的命令来升级整个或部分集群。
```console
$ KUBERNETES_PROVIDER=ubuntu ./kube-push.sh [-m|-n <node id>] <version>
```
默认情况下会升级全部的节点，你也可以用`-m`来升级主节点或`-n`来升级某个普通节点。如果没有给出所要升级的版本，这个脚本会尝试使用本地的二进制安装文件。你应该确认所有的文件都在`cluster/ubuntu/binaries`文件夹中准备好了。

```console
$ tree cluster/ubuntu/binaries
binaries/
├── kubectl
├── master
│   ├── etcd
│   ├── etcdctl
│   ├── flanneld
│   ├── kube-apiserver
│   ├── kube-controller-manager
│   └── kube-scheduler
└── minion
    ├── flanneld
    ├── kubelet
    └── kube-proxy
```

用以下命令来获得帮助。

```console
$ KUBERNETES_PROVIDER=ubuntu ./kube-push.sh -h
```

这里有几个实例：
* 把主节点升级到版本1.0.5: `$ KUBERNETES_PROVIDER=ubuntu ./kube-push.sh -m 1.0.5`
* 把节点10.10.103.223升级到版本1.0.5 : `$ KUBERNETES_PROVIDER=ubuntu ./kube-push.sh -n 10.10.103.223 1.0.5`
* 把主节点和其他节点升级到版本1.0.5: `$ KUBERNETES_PROVIDER=ubuntu ./kube-push.sh 1.0.5`

这个脚本不会删除你集群上的资源，只是替换了二进制安装包。

### 测试
你可以运行`kubectl`命令来检测刚升级的Kubernetes集群是否工作正常，参考[测试](ubuntu.md#测试)。
为了确保升级后的集群版本和你期望的一致，以下命令会有所帮助。
* 升级主节点或所有节点: `$ kubectl version`。 检查*服务器版本*。
* 升级节点10.10.102.223: `$ ssh -t vcap@10.10.102.223 'cd /opt/bin && sudo ./kubelet --version'`


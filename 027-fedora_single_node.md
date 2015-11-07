从[Ferdora][1]入门Kubernetes
--------------------------
本节内容

- [前提条件](#前提条件)
- [说明](#说明)

## 前提条件

 1. 你需要2台或以上安装有Fedora的机器
 
## 说明
本文是针对Fedora系统的Kubernetes入门教程。通过手动配置，你将会理解所有底层的包、服务、端口等。

本文只会让一个节点（之前称从节点）工作。多节点需要在Kubernetes之外[配置一个可用的网络环境][2]，尽管这个额外的配置条件是显而易见的，（本节也不会去配置）。

Kubernetes包提供了一些服务：kube-apiserver,kube-scheduler,kube-controller-manager,kubelet,kube-proxy。这些服务通过systemd进行管理，配置信息都集中存放在一个地方：/etc/kubernetes。我们将会把这些服务运行到不同的主机上。第一台主机，fed-master，将是Kubernetes的master主机。这台机器上将运行kube-apiserver,kube-controller-manager和kube-scheduler这几个服务，此外，master主机上还将运行etcd（如果etcd运行在另一台主机上，那就不需要了，本文假设etcd和Kubernetesmaster运行在同一台主机上）。其余的主机，fed-node，将是从节点，上面运行kubelet, proxy和docker。

系统信息：

主机：

```sh
fed-master = 192.168.121.9
fed-node = 192.168.121.65
```

准备主机：

- 在所有主机上（fed-master和fed-node）都安装Kubernetes，。这对docker也适用。在fed-master上安装etcd。本文在kubernetes-0.18及之后版本上通过测试。

- yum命令之后的 [--enablerepo=update-testing][3] 参数可以保证安装最新的Kubernetes预览版。如果不加这个参数，你安装的版本将是Fedora提供的较旧的稳定版。
- 如果你想获取最新的版本，你可以[从Fedora Koji上下载最新的RPM包][4]，然后yum install安装。而不是用下面的命令。

```sh
yum -y install --enablerepo=updates-testing kubernetes
```

- 安装etcd和iptables

```sh
yum -y install etcd iptables
```

- 在所有主机的/etc/hosts文件中加入master和node节点，如果DNS中已经有了主机名，就不需要加了。需要保证fed-master和fed-node之间可以正常通信，可使用ping测试网络是否连通。

```sh
echo "192.168.121.9 fed-master
192.168.121.65 fed-node" >> /etc/hosts
```

- 编辑/etc/kubernetes/config文件，加入以下内容。所有主机都是（包括master和node）。

```sh
# Comma separated list of nodes in the etcd cluster
KUBE_MASTER="--master=http://fed-master:8080"

# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow_privileged=false"
```

- 禁用master和node上的防火墙，因为如果有其他防火墙规则管理工具的话，docker会无法正常运行。请注意以默认方式安装的fedora服务器上，iptables服务并不存在。

```sh
systemctl disable iptables-services firewalld
systemctl stop iptables-services firewalld
```

配置master主机上Kubernetes服务

- 按照下面的示例编辑/etc/kubernetes/apiserver文件。注意--service-cluster-ip-range参数后跟的IP地址必须是在任何地方都未使用的地址段，这些地址无需路由也无需分配到任何东西上。

```sh
# The address on the local server to listen to.
KUBE_API_ADDRESS="--address=0.0.0.0"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd_servers=http://127.0.0.1:4001"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

# Add your own!
KUBE_API_ARGS=""
```

- 编辑/etc/etcd/etcd.conf文件，使得etcd监听除了127.0.0.1之外的所有IP，如果没有做这步，将会产生“connection refused”这样的错误。

```sh
ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:4001
```

- 启动master上恰当的服务

```sh
for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler;do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES
done
```

- 添加node结点：


- 在Kubernetes的master上创建下面node.json文件：

```json
{
    "apiVersion": "v1",
    "kind": "Node",
    "metadata": {
        "name": "fed-node",
        "labels":{ "name": "fed-node-label"}
    },
    "spec": {
        "externalID": "fed-node"
    }
}
```

现在在你的Kubernetes集群内部创建一个node对象，执行命令

```sh
$ kubectl create -f ./node.json

$ kubectl get nodes
NAME                LABELS              STATUS
fed-node           name=fed-node-label     Unknown
```

请注意上面的命令，它只在内部创建了一个fed-node的引用（原文representation），并不会真的提供fed-node节点。同时，假定fed-node节点（在name中指定的）能够被解析并可从Kubernetes的master节点访问。下面本文将论述如何提供Kubernetes node节点(fed-node)。

配置node节点上的Kubernetes服务

我们需要在节点上配置kubelet

- 按照下面的示例编辑/etc/kubernetes/kubelet文件：

```sh
###
# Kubernetes kubelet (node) config

# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=0.0.0.0"

# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname_override=fed-node"

# location of the api-server
KUBELET_API_SERVER="--api_servers=http://fed-master:8080"

# Add your own!
#KUBELET_ARGS=""
```

- 启动节点上（fed-node）上恰当的服务

```sh
for SERVICES in kube-proxy kubelet docker; do 
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES 
done
```

- 检测以确认现在集群中fed-master能够看到fed-node，而且状态变为Ready了

```sh
kubectl get nodes
NAME                LABELS              STATUS
fed-node          name=fed-node-label     Ready
```

- 节点的删除
如果要想从Kubernetes集群中删除fed-node节点，需要在fed-master上执行下面命令（请别这样做，只是提示删除的方法）

```sh
kubectl delete -f ./node.json
```

你应该完成了吧！

集群现在应该在运行了，跑一个用于测试pod吧。

你应该有一个可用的集群，查看[101][5]节！


  [1]: http://fedoraproject.org/
  [2]: http://kubernetes.io/v1.0/docs/admin/networking.html
  [3]: https://fedoraproject.org/wiki/QA:Updates_Testing
  [4]: http://koji.fedoraproject.org/koji/packageinfo?packageID=19202
  [5]: http://kubernetes.io/v1.0/docs/user-guide/walkthrough/README.html
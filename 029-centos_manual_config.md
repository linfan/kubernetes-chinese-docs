从[CentOS][1]入门Kubernetes
-------------------------
本节内容

- [前提条件](#前提条件)
- [启动一个集群](#启动一个集群)

## 前提条件 ##
你需要2台或以上安装有CentOS的机器

## 启动一个集群 ##
本文是针对CentOS系统的Kubernetes入门教程。通过手动配置，你将会理解所有底层的包、服务、端口等。

本文只会让一个节点工作。多节点需要在Kubernetes之外配置一个可用的的[网络环境][2]，尽管这个额外的配置条件是显而易见的，（本节也不会去配置）。

Kubernetes包提供了一些服务：kube-apiserver, kube-scheduler, kube-controller-manager, kubelet, kube-proxy。这些服务通过systemd进行管理，配置信息都集中存放在一个地方：/etc/kubernetes。我们将会把这些服务运行到不同的主机上。第一台主机，centos-master，将是Kubernetes 集群的master主机。这台机器上将运行kube-apiserver, kube-controller-manager和kube-scheduler这几个服务，此外，master主机上还将运行etcd。其余的主机，fed-minion，将是从节点，将会运行kubelet, proxy和docker。

系统信息：

主机：

```sh
centos-master = 192.168.121.9
centos-minion = 192.168.121.65
```

准备主机：

- 添加virt7-testing源，在所有主机上（centos-master和centos-minion），使用下面信息添加源：

```sh
[virt7-testing]
name=virt7-testing
baseurl=http://cbs.centos.org/repos/virt7-testing/x86_64/os/
gpgcheck=0
```

- 在所有主机上（centos-master和centos-minion）都安装Kubernetes。这对etcd，docker和cadvisor也适用。

```sh
yum -y install --enablerepo=virt7-testing kubernetes
```

*注意使用etcd-0.4.6-7(这是该文档的临时版本)

如果你没有配套virt7-testing源安装etcd 0.4.6-7版，请用下面命令卸载它：

```sh
yum erase etcd
```

原因是在当前的的 virt7-testing源中，etcd包被更新了，会引起服务错误。
执行下面两行命令安装etcd-0.4.6-7

```sh
yum install http://cbs.centos.org/kojifiles/packages/etcd/0.4.6/7.el7.centos/x86_64/etcd-0.4.6-7.el7.centos.x86_64.rpm
yum -y install --enablerepo=virt7-testing kubernetes
```

- 在所有主机的/etc/hosts文件中加入master和node节点，如果DNS中已经有了主机名，就不需要加了。

```sh
echo "192.168.121.9 centos-master
192.168.121.65 centos-minion" >> /etc/hosts
```

- 编辑/etc/kubernetes/config文件，加入以下内容：

```sh
# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd_servers=http://centos-master:4001"

# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow_privileged=false"
```

- 禁用master和node上的防火墙，因为如果有其他防火墙规则管理工具的话，docker会无法正常运行。

```sh
systemctl disable iptables-services firewalld
systemctl stop iptables-services firewalld
```

配置master主机上Kubernetes服务

- 按照下面的示例编辑/etc/kubernetes/apiserver文件：

```sh
# The address on the local server to listen to.
KUBE_API_ADDRESS="--address=0.0.0.0"

# The port on the local server to listen on.
KUBE_API_PORT="--port=8080"

# How the replication controller and scheduler find the kube-apiserver
KUBE_MASTER="--master=http://centos-master:8080"

# Port kubelets listen on
KUBELET_PORT="--kubelet_port=10250"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

# Add your own!
KUBE_API_ARGS=""
```


- 启动master上恰当的服务

```sh
for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES
done
```

配置node节点上的Kubernetes服务
我们需要在节点上配置kubelet并启动kubelet和proxy


- 按照下面的示例编辑/etc/kubernetes/kubelet文件：

```sh
# The address for the info server to serve on
KUBELET_ADDRESS="--address=0.0.0.0"

# The port for the info server to serve on
KUBELET_PORT="--port=10250"

# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname_override=centos-minion"

# Add your own!
KUBELET_ARGS=""
```

- 启动节点上（fed-node）上恰当的服务

```sh
for SERVICES in kube-proxy kubelet docker; do 
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES 
done
```

你应该完成了！

- 检查以确认现在集群中fed-master能够看到fed-node 

```sh
$ kubectl get nodes
NAME                   LABELS            STATUS
centos-minion          <none>            Ready
```

集群现在应该在运行了，启动一个用于测试的pod吧。

你应该有一个功能正常的集群，查看[101][3]节！


  [1]: http://centos.org/
  [2]: http://kubernetes.io/v1.0/docs/admin/networking.html
  [3]: http://kubernetes.io/v1.0/docs/user-guide/walkthrough/README.html
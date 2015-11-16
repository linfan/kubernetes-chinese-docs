
# 从零开始

这部文档是面对想要订制Kubernetes集群的读者。如果你发现现有的入门指南已经可以满足你对 [这个列表](README.md)上所列的需求，我们建议你继续阅读这个根据前人积累经验所写的新手指南。但如果你有入门指南所不能满足的对IaaS，网络，配置管理或对操作系统有特殊要求，这个指南将会提供给你一个指导性的概述。
这个指南也会对希望对现有集群配置有进一步理解的读者提供到帮助。

## 设计和准备

### 学习

  1. 你应该已经熟悉Kubernetes了。我们建议根据 其他入门指南架设一个临时的集群。这样可以帮助你先熟悉Kubernetes命令行([kubectl](../user-guide/kubectl/kubectl.md))和一些概念([pods](../user-guide/pods.md), [services](../user-guide/services.md), 等等)。
  2. 当你浏览完其他入门指南的时候，你应该已经安装好了`kubectl`。如果没有，你可以根据[这个](../user-guide/prereqs.md)说明安装。

### Cloud Provider

Kubernetes有一个概念叫Cloud Provider，是指一个提供管理TCP负载均衡，节点（实例）和路由器接口的模块。`pkg/cloudprovider/cloud.go`里具体定义了这个接口。
当然，你也可以不用实现这个Cloud Provider去新建一个自定义的聚群（例如，使用裸机集群）。取决于不同部件是如何设置的，并不是所有接口都需要实现的。

### 节点
* 你可以使用虚拟机或物理机。
* 为了运行本指南给出的例子和测试用例，你最好有4个节点以上。
* 虽说很多入门指南讲主节点和普通节点做了区分，但严格意义上讲，这不是必要的。
* 节点需要运行在x86_64的Linux系统上。当然也可能运行在其他的系统和CPU架构上，但这个指南不会提供相关的帮助。
* 对于一个拥有10个以上节点的集群来说，Apiserver和etcd可以一起安装在一台1核和1GB内存的机子上。
* 你可以给其他节点可以分配合理的内存和CPU内核。不是所有节点需要同样的配置。

### 网络

Kubernetes有一个独特的[网络模型](../admin/networking.md).

Kubernetes给每一个pod分配IP地址。当你新建一个集群，为了保证Pod获得IP地址，你需要给Kubernetes分配一个IP地址池。最简单的做法是每当

节点与节点之间的通信可以以一下两种方式实现：
* 配置网络完成Pod的IP地址路由
 * 因为一切从头开始，所以难度会大一些。
 * Google Compute Engine ([GCE](gce.md)) 和 [AWS](aws.md)会指导如何使用这种方式
 * 需要编程配置路由器和交换机去实现Pod的IP地址路由。
 * 可在Kubernetes环境外配置或者通过在Cloud Provider模块的“路由”接口里实现。
 * 通常情况下，提供最优网络性能。

* 建立一个拓扑网络
  * 较容易建立
  * 因为数据流量被封装，所以每个Pod的IP地址是可以路由的。
  * 例如：    
    *  [Flannel](https://github.com/coreos/flannel)
    *  [Weave](http://weave.works/)
    *  [Open vSwitch (OVS)](http://openvswitch.org/)
    
  * 不需要CLoud Provider模块里的“路由”部分 
  * 较为不太理想的性能（具体的性能弱化取决于你的实际情况） 

你需要为Pod所需要的IP地址选一个IP地址范围。
*  一些可选择配置方式：
  * GCE：每一个项目有一个自己的IP子网“10.0.0.0/8”。项目中的每一个Kubernetes集群从中获得一个“/16”的子网。每一个节点从'/16'的子网里获IP地址。
  * AWS：在一个组织内使用一个VPC。从中分配地址给每一个聚群或者使用给不同的集群分配不同的VPC。
  * 暂不支持支持IPv6
*  给每一的node的Pod地址分配一个CIDR子网或者一个
  * 你总共需要max-pods-per-node * max-number-of-nodes个IP地址。一个“/24” 如果缺少IP地址，一个“/26”（62个节点）或者“/27”（30个节点）也能满足。
  * 例如，使用“10.10.0.0/16” “10.10.0.0/24” “10.10.255.0/24”
  * 需要路由设置或连接到拓扑网络

Kubernetes 会给每个[service](../user-guide/services.md)分配一个IP地址。 但是service的地址并不一定是可路由的。当数据离开节点时，kube-proxy需要将Service IP地址翻译成Pod IP地址。因此，你需要给Service也分配一个地址段。这个网段叫做“SERVICE_CLUSTER_IP_RANGE”。例如，你可以这样设置“SERVICE_CLUSTER_IP_RANGE="10.0.0.0/16”，这样的话就会允许65534个不同的Service同时运行。请注意，你可以调整这个IP地址段。但不允许在Service和Pod运行的时候移除这个网段。


同样，你需要为主节点选一个静态IP地址。
－命名这个IP地址为“MASTER_IP”。
－配置防火墙，从而允许访问apiserver端口80和443。
－使用sysctl设置”net.ipv4.ip_forward = 1“从而开启IPv4 forwarding。

### 集群命名
为你的集群选个名字。要选一个简短不会和其他服务重复的名字。
 * 用kubectl来访问不同的集群。比如当你想在其他的区域测试新的Kubernetes版本。
 * Kubernetes集群可以建立一个Cloud Provider资源（例如，AWS ELB）。所以不同的集群要能区分他们之间的相关资源，这个名字叫做“CLUSTERNAME”。

### 软件安装包

你需要以下安装包
  * etcd
  * 以下容器二选一:
    * docker
    * rkt
  * Kubernetes
    * kubelet
    * kube-proxy
    * kube-apiserver
    * kube-controller-manager
    * kube-scheduler

#### 下载和解压缩Kubernetes安装
Kubernets安装版本包包含所有Kuberentes的二进制发行版本和所对应的etcd。你可使直接使用这个二进制发行版本（推荐）或者按照[开发者文档](../devel/README.md)说明编译这些Kubernetes的二进制文件。 本指南只讲述了如何直接使用二进制发行版本。
下载[最新安装版本](https://github.com/kubernetes/kubernetes/releases/latest)并解压缩。之后找到你下载“./kubernetes/server/kubernetes-server-linux-amd64.tar.gz”的路径， 并解压缩这个压缩包。然后在其中找到“./kubernetes/server/bin”文件夹。里面有所所需的可运行的二进制文件。


#### 选择安装镜像
在容器外运行docker，kuberlet和kube-proxy，就像你运行任何后台系统程序。所以你只需要这些基本的二进制运行文件。etcd， kube-apiserver， kube-controller-manager和kube-scheduler，我们建议你在容器里运行etcd， kube-apiserver， kube-controller-manager和kube-scheduler。所以你需要他们的容器镜像。
你可以选择不同的Kubernetes镜像：
* 你可以使用Google Container Registry (GCR)上的镜像文件：
 * 例如“gcr.io/google_containers/hyperkube:$TAG”。这里的“TAG”是指最新的发行版本的标示。这个表示可以从[最新发行说明](https://github.com/kubernetes/kubernetes/releases/latest)找到。
 * 确定$TAG和你使用的kubelet，kube-proxy的标签是一致的。
 * [hyperkube](../../cmd/hyperkube/)是一个集成二进制运行文件
  * 你可以使用“hyperkube kubelet ...”来启动kubelet ，用“hyperkube apiserver ...”运行apiserver， 等等。
* 生成你自己的镜像：
 * 使用私有镜像服务器。
 * 例如，可以使用“docker load -i kube-apiserver.tar”将“./kubernetes/server/bin/kube-apiserver.tar”文件转化成dokcer镜像。
 * 之后可使用“docker images”来验证镜像是否从制定的镜像服务器加载成功。

对于etcd，你可以：
* 使用上Google Container Registry (GCR)的镜像，例如“gcr.io/google_containers/etcd:2.0.12”
* 使用[Docker Hub](https://hub.docker.com/search/?q=etcd)或着[Quay.io](https://quay.io/repository/coreos/etcd)上的镜像，例如“quay.io/coreos/etcd:v2.2.0”
* 使用你操作系统自带的etcd
* 自己编译
   * 你可以使用这个命令行“cd kubernetes/cluster/images/etcd; make”

我们建议你使用Kubernetes发行版本里提供的etcd版本。Kubernetes发行版本里的二进制运行文件只是和这个版本的etcd测试过。你可以在“kubernetes/cluster/images/etcd/Makefile”里“ETCD_VERSION”所对应的值找到所推荐的版本号。
接下来本文档会假设你已经选好镜像标示并设置了对应的环境变量。 设置好了最新的标示和正确的镜像服务器：
  * "HYPERKUBE_IMAGE==gcr.io/google_containers/hyperkube:$TAG"
  * "ETCD_IMAGE=gcr.io/google_containers/etcd:$ETCD_VERSION"

### 安全模式

这里有两种主要的安全选项：
* 使用HTTP访问apiserver
 * 配合使用防火墙。
 * 这种方法比较易用。
* 使用HTTPS访问apiserver
 * 配合电子证书和用户登录信息使用。
 * 推荐使用
 * 设置电子证书较为复杂。

如果要用HTTPS这个方式，你需要准备电子证书和用户登录信息。

#### 准备安全证书
你需要准备多个证书：
* 主节点会是一个HTTPS服务器，所以需要一个证书。
* 如果Kuberlets需要通过HTTPS提供API服务时，这些kuberlets需要出示证书向主节点证明主从关系。

除非你决定要一个真正CA来生成这些证书的话，你需要一个根证书，并用这个证书来给主节点，kuberlet和kubectl的证书签名。
* 参见“cluster/gce/util.sh”脚本里的“create-certs”
* 并参见“cluster/saltbase/salt/generate-cert/make-ca-cert.sh”和cluster/saltbase/salt/generate-cert/make-cert.sh“

你需要修改以下部分(我们之后也会用到这些参数的)
* ”CA_CERT“
  * 放置在apiserver所运行的节点上，比如“/srv/kubernetes/ca.crt”。
* “MASTER_CERT”
  * 用CA_CERT来签名
  * 放置在apiserver所运行的节点上，比如"/srv/kubernetes/server.crt”。
* "MASTER_KEY“
  * 放置在apiserver所运行的节点上，比如”/srv/kubernetes/server.key“。
* “KUBELET_CERT”
  * 可选配置
* ”KUBELET_KEY“
  * 可选配置

#### 准备登录信息
管理员（及任何用户）需要：
 * 对应验证身份的令牌或密码。
 * 令牌可以是长字符串，比如 32个字符
  * 可参见”TOKEN=$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64 | tr -d "=+/" | dd bs=32 count=1 2>/dev/null)““

你的令牌和密码需要保存在apiserver上的一个文件里。本指南使用这个文件”/var/lib/kube-apiserver/known_tokens.csv“。文件的具体格式在[认证文档](../admin/authentication.md)里描述了。
至于如何把登录信息分发给用户，Kubernetes是将登录信息放入[kubeconfig文件](../user-guide/kubeconfig-file.md)里。

管理员可以按如下步骤创建kubeconfig文件：
* 如果你已经在非客制化的集群上运行过Kubernetes(例如，按照入门指南架设过Kubernetes)，那么你已经有“$HOME/.kube/config`”文件了。
* 你需要在kuberconfig文件里添加证书，密钥和主节点IP：
 * 如果你选择了“firewall-only”的安全设置，你需要按如下设置apiserver：
  * “kubectl config set-cluster $CLUSTER_NAME --server=http://$MASTER_IP --insecure-skip-tls-verify=true”
 * 否则，按如下设置你的apiserver的IP，证书，用户登录信息：
  * “kubectl config set-cluster $CLUSTER_NAME --certificate-authority=$CA_CERT --embed-certs=true --server=https://$MASTER_IP”
  * “kubectl config set-credentials $USER --client-certificate=$CLI_CERT --client-key=$CLI_KEY --embed-certs=true --token=$TOKEN”
 * 设置你的集群为缺省集群：
  * “kubectl config set-context $CONTEXT_NAME --cluster=$CLUSTER_NAME --user=$USER”
  * “kubectl config use-context $CONTEXT_NAME”

接下来，为kubelets和kube-proxy准备kubeconfig文件。至于要准备多少不同的文件，这里有几个选项：
 1. 使用和管理员同样的登陆账号
  * 这是最简单的建设方法。
 2. 所有的kubelet使用同一个令牌和kubeconfig文件，另外一套给所有的kube-proxy使用，在一套给管理员使用。
  * 这个设置和GCE的配置类似。
 3. 为每一个kubelet，kube-proxy和管理员准备不同的登陆账号。
  * 这个配置在实现中，目前还不支持。

为了生成这个文件，你可以参照“cluster/gce/configure-vm.sh”中的代码直接从“$HOME/.kube/config”拷贝过去或者参考以下模版：

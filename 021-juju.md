＃ 从juju开始

[Juju](https://jujucharms.com/docs/stable/about-juju)可以通过扩展，安装和配置集群内的所有系统从而实现简易部署Kubernetes。
一旦部署完毕，集群可以轻松支持单命令扩展集群大小。

## 前提条件

> 注意: 如果你在Ubuntu上运行kube-up，所有的相关依赖的安装都会被相对应处理。你可以放心越过章节[运行
> Kubernetes集群](#运行Kubernetes集群)的阅读。

### Ubuntu上

在你的本地Ubuntu系统上安装[安装Juju客户端](https://jujucharms.com/get-started):

    `sudo add-apt-repository ppa:juju/stable`
    `sudo apt-get update`
    `sudo apt-get install juju-core juju-quickstart`


### 使用Docker
如果你不使用Ubuntu或者你倾向于Docker，你可以是用以下命令命令：

    `mkdir ~/.juju`
    `sudo docker run -v ~/.juju:/home/ubuntu/.juju -ti jujusolutions/jujubox:latest`

到这里你不可避免的要需要使用`juju quickstart`命令。
为你的云环境创建登入信息：

    `juju quickstart --constraints="mem=3.75G" -i`

> `constraints`参数是可选项，它用来决定Juju所新建的虚拟机的内存大小。相比较性能越强的虚拟机会占用更多资> 源，造成更高开销。
根据接下来的提示选择`save`和`use`。Quickstart将会引导创建Juju根节点并建立Juju的用户网页界面。

## 运行Kubernetes集群
你需要在启动集群前设置`KUBERNETES_PROVIDER`的环境变量。

    `export KUBERNETES_PROVIDER=juju`
   ` cluster/kube-up.sh`

如果是你第一次运行`kube-up.sh`脚本，这个脚本会安装所依赖的程序，并运行一个配置向导来让你选择你的云服务商和登入信息。

下一步它将会部署Kubernetes主节点，etc和2个基于Flannel的Software Defined Networking (SDN)节点，从而支持容器间的相互通信。

## 发现集群
`juju status`命令提供集群内每一个部署单元的信息：

    $ juju status --format=oneline
    - docker/0: 52.4.92.78 (started)
      - flannel-docker/0: 52.4.92.78 (started)
      - kubernetes/0: 52.4.92.78 (started)
    - docker/1: 52.6.104.142 (started)
      - flannel-docker/1: 52.6.104.142 (started)
      - kubernetes/1: 52.6.104.142 (started)
    - etcd/0: 52.5.216.210 (started) 4001/tcp
    - juju-gui/0: 52.5.205.174 (started) 80/tcp, 443/tcp
    - kubernetes-master/0: 52.6.19.238 (started) 8080/tcp

你可以使用`juju ssh`来访问任意单元:

   ` juju ssh kubernetes-master/0`


## 运行一些容器！

在主节点上你可以找到`kubectl`。我们使用ssh登入主节点来启动一些容器，当然你也可以设置`KUBERNETES_MASTER`为"kubernetes-master/0”的IP地址，从而在本地使用`kubectl`。

在启动容器前pods是不会存在的：

    kubectl get pods
    NAME             READY     STATUS    RESTARTS   AGE

    kubectl get replicationcontrollers
    CONTROLLER  CONTAINER(S)  IMAGE(S)  SELECTOR  REPLICAS

我们依照aws-coreos这个例子。创建一个pod的manifest: `pod.json`

```
json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "hello",
    "labels": {
      "name": "hello",
      "environment": "testing"
    }
  },
  "spec": {
    "containers": [{
      "name": "hello",
      "image": "quay.io/kelseyhightower/hello",
      "ports": [{
        "containerPort": 80,
        "hostPort": 80
      }]
    }]
  }
}
```

用kubectl建立pod:

    `kubectl create -f pod.json`


获取pod信息:

    `kubectl get pods`

让我们来测试一个hello应用。首先让我们找到这个容器运行在哪个节点上。Juju是个更好和容器交互的工具，我们可以用`juju run`和`juju status` 到到这个hello应用。

退出ssh并运行：
    juju run --unit kubernetes/0 "docker ps -n=1"
    ...
    juju run --unit kubernetes/1 "docker ps -n=1"
    CONTAINER ID        IMAGE                                  COMMAND             CREATED             STATUS              PORTS               NAMES
    02beb61339d8        quay.io/kelseyhightower/hello:latest   /hello              About an hour ago   Up About an hour                        k8s_hello....


我们可以看到容器运行在”kubernetes/1”上, 我们可以打开端口80:

    juju run --unit kubernetes/1 "open-port 80"
    juju expose kubernetes
    sudo apt-get install curl
    curl $(juju status --format=oneline kubernetes/1 | cut -d' ' -f3)

最后删除pod:

    juju ssh kubernetes-master/0
    kubectl delete pods hello


## 扩展集群
我们可以用如下命令来添加节点单元：

    juju add-unit docker # creates unit docker/2, kubernetes/2, docker-flannel/2

## 运行”k8petstore”示例应用

示例[k8petstore example](../../examples/k8petstore/)是一个现成的
[juju action](https://jujucharms.com/docs/devel/actions)。

    juju action do kubernetes-master/0

> 注意: 这个示例包括的curl自动生成"petstore”的商品交易，这些交易储存在Redia中，并在网页上显示交易流量
> 大小。

## 拆除集群

    ./kube-down.sh

或者拆除目前整个Juju环境(使用`juju env`命令):

    juju destroy-environment --force `juju env`


## 更多信息
你可以github.com的`kubernetes`项目里找到Kubernetes的charms和bundles：

 - [Bundle Repository](http://releases.k8s.io/HEAD/cluster/juju/bundles)
   * [Kubernetes master charm](../../cluster/juju/charms/trusty/kubernetes-master/)
   * [Kubernetes node charm](../../cluster/juju/charms/trusty/kubernetes/)
 - [More about Juju](https://jujucharms.com)


### 云环境的兼容性
Juju已经在不同的公有云测试过了。目前Juju测试过的云平台是[Amazon Web Service](https://jujucharms.com/docs/stable/config-aws),
[Windows Azure](https://jujucharms.com/docs/stable/config-azure),
[DigitalOcean](https://jujucharms.com/docs/stable/config-digitalocean),
[Google Compute Engine](https://jujucharms.com/docs/stable/config-gce),
[HP Public Cloud](https://jujucharms.com/docs/stable/config-hpcloud),
[Joyent](https://jujucharms.com/docs/stable/config-joyent),
[LXC](https://jujucharms.com/docs/stable/config-LXC), any
[OpenStack](https://jujucharms.com/docs/stable/config-openstack) deployment,
[Vagrant](https://jujucharms.com/docs/stable/config-vagrant), and
[Vmware vSphere](https://jujucharms.com/docs/stable/config-vmware).

如果你没有在列表里发现合适你的云服务提供商，许多云服务是可以通过[manual provisioning](https://jujucharms.com/docs/stable/config-manual)手动设置的。

Kubernetes
Kubernetes安装集成包已经在GCE和AWS上测试过了，版本1.0.0可以正常工作。
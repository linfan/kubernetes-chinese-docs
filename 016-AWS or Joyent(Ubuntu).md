# Juju上部署入门

Juju使通过配置部署Kubernetes，在集群中安装和配置所有系统更加简单。可通过一个命令增加集群尺寸来简单的扩展部署集群

## 内容列表

* 先决条件
    * Ubuntu上部署
    * Docker相关部署
* 运行Kubernetes集群
* 探索集群
* 运行多个容器！
* 扩展集群
* 运行“k8petsore”示例应用
* 拆除集群
* 更多信息
    * 云兼容

## 先决条件

注意：如果你运行kube-up，在Ubuntu上——所有的依赖会为您处理。你可以跳转到这个部分：运行Kubernetes集群

### Ubuntu上部署

在您的本地Ubuntu系统安装Juju客户端。

```
sudo add-apt-repository ppa:juju/stable
sudo apt-get update
sudo apt-get install juju-core juju-quickstart
```

### Docker相关部署

如果您不使用Ubuntu，而是使用Docker，您可以运行以下命令：

```
mkdir ~/.juju
sudo docker run -v ~/.juju:/home/ubuntu/.juju -ti jujusolutions/jujubox:latest
```

此时你可以在当前路径上获取```juju quickstart```命令。

为您所选择允许的云设置证书：

```
juju quickstart --constraints="mem=3.75G" -i```
容器flag是可选的，当请求一个新的虚拟机时，他改变Juju生成的虚拟机尺寸。大的虚拟机相比小虚拟机运行的更快，但是花费更多。

根据对话选择```save```和```use```。快速入门将启动Juju跟节点，根据用户接口设置Juju页面。

## 运行Kubernetes集群
启动集群之前需要导出环境变量```KUBERNETES_PROVIDER```。
```
export KUBERNETES_PROVIDER=juju
cluster/kube-up.sh```

如果这是第一次运行```kube-up.sh```脚本，它会安装Juju部署入门的所有依赖关系，另外它会根据通用配置运行一个窗口，允许你选择云服务商，输入适当的访问凭证。

下一步它会部署kubernetes master，etcd，2个带有flannel的nodes，flannel是基础软件定义网络(SDN)，它可以使在不同的主机上的容器互相通信。

## 探索集群
```juju status```命令提供了集群中每个单元的信息：
```
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
```
你可
以使用```juju ssh```去访问任何一个单元：
```
juju ssh kubernetes-master/0
```
## 运行多个容器！

在Kubernetes主节点```kubectl```是可用的。我们ssh登录去运行一些容器，但也可以通过设置```KUBERNETES_MASTER```为“kubernetes-master/0”的ip地址来使用本地```kubectl```。

在启动一个容器前无pods可获取
```
kubectl get pods
NAME             READY     STATUS    RESTARTS   AGE

kubectl get replicationcontrollers
CONTROLLER  CONTAINER(S)  IMAGE(S)  SELECTOR  REPLICAS
```
我们将跟随aws-coreos实例。创建一个pod清单：pod.json
```
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
}```

使用kubectl创建pod：
```
kubectl create -f pod.json```

获取pod信息：
```
kubectl get pods```

测试hello应用，我们需要定位容器所在的节点。使用Juju去作用于容器的更好的工具是在工作节点上，但是我们可以使用```juju run```和```juju status```去找到我们的hello应用。
```
juju run --unit kubernetes/0 "docker ps -n=1"
...
juju run --unit kubernetes/1 "docker ps -n=1"
CONTAINER ID        IMAGE                                  COMMAND             CREATED             STATUS              PORTS               NAMES
02beb61339d8        quay.io/kelseyhightower/hello:latest   /hello              About an hour ago   Up About an hour                        k8s_hello....```
我们看到“kubernetes/1”有我们的容器，我们可以打开80端口：
```
juju run --unit kubernetes/1 "open-port 80"
juju expose kubernetes
sudo apt-get install curl
curl $(juju status --format=oneline kubernetes/1 | cut -d' ' -f3)```

最后删除pod：
```
juju ssh kubernetes-master/0
kubectl delete pods hello```

##扩展集群
我们可以想这样增加节点单元：
```
juju add-unit docker # creates unit docker/2, kubernetes/2, docker-flannel/2```


## 运行“k8petsore”示例应用

[k8petstore示例](https://github.com/kubernetes/kubernetes/tree/master/examples/k8petstore)可以像一个[juju action](https://jujucharms.com/docs/devel/actions)获取到。
```
juju action do kubernetes-master/0```

注意：这个示例既包含curl状态来练习这个应用。这个应用自动生成“prestore”日志写到redis上，并且允许你在浏览器上可视化吞吐量。

## 拆除集群
```
./kube-down.sh```

或者破坏你当前的Juju环境（使用```juju env```命令）
```
juju destroy-environment --force `juju env````

## 更多信息
Kubernetes的分支和包可以在github.com的```kubernetes```项目中找到：
* [镜像包](http://releases.k8s.io/HEAD/cluster/juju/bundles)
	* [Kubernetes主节点分支](https://github.com/kubernetes/kubernetes/tree/master/cluster/juju/charms/trusty/kubernetes-master)
	* [Kubernetes节点分支](https://github.com/kubernetes/kubernetes/blob/master/cluster/juju/charms/trusty/kubernetes)
* [关于Juju的更多信息](https://jujucharms.com/)

###云兼容

Juju运行在本地和各种公共云提供商。Juju当前和[Amazon Web Service](https://jujucharms.com/docs/stable/config-aws)，[Windows Azure](https://jujucharms.com/docs/stable/config-azure)，[DigitalOcean](https://jujucharms.com/docs/stable/config-digitalocean)，[Google Compute Engine](https://jujucharms.com/docs/stable/config-gce)，[HP Public Cloud](https://jujucharms.com/docs/stable/config-hpcloud)，[Joyent](https://jujucharms.com/docs/stable/config-joyent)，[LXC](https://jujucharms.com/docs/stable/config-LXC)，任何[OpenStack](https://jujucharms.com/docs/stable/config-openstack)部署，[Vagrant](https://jujucharms.com/docs/stable/config-vagrant)，和 [Vmware vSphere](https://jujucharms.com/docs/stable/config-vmware)。

如果你没有在多个云列表看到你比较喜欢的云供应商，可以配置为[手动配置](https://jujucharms.com/docs/stable/config-manual)。

Kubernetes包已经在GCE和AWS上测试，且使用1.0.0版本









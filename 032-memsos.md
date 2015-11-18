# 如何在Mesos上运行Kubernetes

## 关于在Mesos上运行Kubernetes

Mesos允许Kubernetes和其他Mesos构架，例如：[Hadoop][1],[Spark][2]和[Chronos][3]来动态分享集群资源。

Mesos确保同一集群上的运行的不同架构上的应用之间相互隔离，和较公平资源的分配。

Mesos集群几乎可以部署在任何IaaS云服务上或是你自己的数据中心。当Kubernetes运行在Mesos上，你可以轻松将Kubernetes上运行的任务在任意云平台之间转移。

本指南将具体介绍如何架设在Mesos集群上运行Kubernetes。我们一步一步指导你怎么在Mesos集群上添加Kubernetes和如何启动你的第一个pod来运行nginx网站服务器。

**注意:** [现有实现上的已知问题][7]，集中式的监控和日志功能尚不支持。如果你发现以下步骤有什么问题，请[在kubernetes-mesos项目报告这个问题][8]。

你可以在[contrib directory][13]找到进一步的有关信息。

### 前提条件

* 理解[Apache Mesos][6]
* 一个运行在[Google Compute Engine的Mesos集群][5]
* 一个连接到这个集群[VPN连接][10]
* 一台在集群里并可以作为Kubernetes*主节点*的主机，这个主机需安装一下软件：
  * GoLang > 1.2
  * make (i.e. build-essential)
  * Docker

**注意**: 你*可以*, 但是你*不要*在运行Mesos master的主机上部署Kubernetes-Mesos。

### 部署Kubernetes-Mesos

用SSH登入Kubernetes*主节点*。参考以下命令行时需要替换其中的地址为正确的IP地址。

```bash
ssh jclouds@${ip_address_of_master_node}
```

编译Kubernetes-Mesos。

```bash
git clone https://github.com/kubernetes/kubernetes
cd kubernetes
export KUBERNETES_CONTRIB=mesos
make
```

设置环境变量。
可以用`hostname -i`来获得主节点的内部IP地址。

```bash
export KUBERNETES_MASTER_IP=$(hostname -i)
export KUBERNETES_MASTER=http://${KUBERNETES_MASTER_IP}:8888
```

请注意KUBERNETES_MASTER用来定义API的endpoint的。 如果`~/.kube/config`已存在并指向其他endpoint, 你需要在最后一步使用kubectl时加上`--server=${KUBERNETES_MASTER}`选项。

### 部署etcd

启动etcd并监测它是否正常运行：

```bash
sudo docker run -d --hostname $(uname -n) --name etcd \
  -p 4001:4001 -p 7001:7001 quay.io/coreos/etcd:v2.0.12 \
  --listen-client-urls http://0.0.0.0:4001 \
  --advertise-client-urls http://${KUBERNETES_MASTER_IP}:4001
```

```console
$ sudo docker ps
CONTAINER ID   IMAGE                        COMMAND   CREATED   STATUS   PORTS                NAMES
fd7bac9e2301   quay.io/coreos/etcd:v2.0.12  "/etcd"   5s ago    Up 3s    2379/tcp, 2380/...   etcd
```

最好检测一下你可以连接到你的etcd服务

```bash
curl -L http://${KUBERNETES_MASTER_IP}:4001/v2/keys/
```
如果连接没有问题，你可以看见所显示的etcd秘钥（如果有的话）。

### 启动Kubernetes-Mesos服务

设置正确的环境变量`PATH`以便访问Kubernetes-Mesos中的二进制运行文件：


```bash
export PATH="$(pwd)/_output/local/go/bin:$PATH"
```
确认你的Mesos master：根据具体Mesos的安装，Mesos master的地址可能会是`host:port`的样子，比如`mesos-master:5050`的`host:port`， 也可能是ZooKeeper的URL，比如`zk://zookeeper:2181/mesos`。在生产环境中，推荐使用ZooKeeper的URL，这样可避免Kubernetes在Mesos master的地址有变化的时候出现问题。

```bash
export MESOS_MASTER=<host:port or zk:// url>
```

在当前目录下新建一个云配置文件`mesos-cloud.conf`，文件内写入一下内容:

```console
$ cat <<EOF >mesos-cloud.conf
[mesos-cloud]
        mesos-master        = ${MESOS_MASTER}
EOF
```

现在启动kubernetes-mesos的API服务和主节点上的scheduler：

```console
$ km apiserver \
  --address=${KUBERNETES_MASTER_IP} \
  --etcd-servers=http://${KUBERNETES_MASTER_IP}:4001 \
  --service-cluster-ip-range=10.10.10.0/24 \
  --port=8888 \
  --cloud-provider=mesos \
  --cloud-config=mesos-cloud.conf \
  --secure-port=0 \
  --v=1 >apiserver.log 2>&1 &

$ km controller-manager \
  --master=${KUBERNETES_MASTER_IP}:8888 \
  --cloud-provider=mesos \
  --cloud-config=./mesos-cloud.conf  \
  --v=1 >controller.log 2>&1 &

$ km scheduler \
  --address=${KUBERNETES_MASTER_IP} \
  --mesos-master=${MESOS_MASTER} \
  --etcd-servers=http://${KUBERNETES_MASTER_IP}:4001 \
  --mesos-user=root \
  --api-servers=${KUBERNETES_MASTER_IP}:8888 \
  --cluster-dns=10.10.10.10 \
  --cluster-domain=cluster.local \
  --v=2 >scheduler.log 2>&1 &
```

使用Disown显示后台运行的任务。

```bash
disown -a
```

#### 验证KM服务

设置正确的环境变量`PATH`以便访问kubectl：

```bash
export PATH=<path/to/kubernetes-directory>/platforms/linux/amd64:$PATH
```

使用`kubectl`和kubernetes-mesos交互:

```console
$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
```

```console
# 注意: 显示的service IP有可能不同
$ kubectl get services
NAME             LABELS                                    SELECTOR   IP(S)          PORT(S)
k8sm-scheduler   component=scheduler,provider=k8sm         <none>     10.10.10.113   10251/TCP
kubernetes       component=apiserver,provider=kubernetes   <none>     10.10.10.1     443/TCP
```

最后, 登陆地址`http://<mesos-master-ip:port>`访问Kubernetes服务网页。确定你有有效的VPN连接。点击
｀Frameworks｀标签并寻找一个名叫"Kubernetes”的激活framework。

## 启动一个pod

在本地文件里写入描述pod的JSON:

```bash
$ cat <<EOPOD >nginx.yaml
```

```yaml
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
EOPOD
```

用`kubectl`CLI把这个pod的描述发送到Kubernetes:

```console
$ kubectl create -f ./nginx.yaml
pods/nginx
```
`dockerd`会从Internet下载相对应的镜像，这个过程大概持续一到两分钟的时间。
我们可以用`kubectl`CLI去观察pod的状态:

```console
$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          14s
```

在Mesos的网页上确认每一个pod都正常运行。点击Kubernetes的framework选项后，会显示在Kubernetes启动pod的Mesos任务。

## 启动kube-dns

Kube-dns是为Kubernetes集群提供基于DNS的发现服务的插件。参见[Kubernetes里的DNS][4]以获得更多信息。

kube-dns 插件运行在集群内的pod上。这个pod包涵三个运行在一起的容器:
 * 一个本地etc实例
 * DNS服务器[skydns][11]
 * 完成Kubernetes集群状态到skydns交互的kube2sky进程

skydns容器通过53端口向集群提供DNS服务。 etcd communication works via local 127.0.0.1 communication

我们假设 kube-dns会使用
 * service IP `10.10.10.10`
 * 和`cluster.local`域名。

注意这两个值已经在设置apiserver时使用了。

在[cluster/addons/dns/skydns-rc.yaml.in][11]目录下可以找到用一个replication controller建立一个有3个容器的pod的模版。为了获得一个正确的replication controller的yaml文件，以下这几步是必须的：

 * 将`{{ pillar['dns_replicas'] }}`替换为`1`
 * 将`{{ pillar['dns_domain'] }}`替换为`cluster.local.`
 * 将参数`--kube_master_url=${KUBERNETES_MASTER}`添加到kube2sky容器的命令行中

除此之外service模版[cluster/addons/dns/skydns-svc.yaml.in][12]需要一下变动：
 * 将`{{ pillar['dns_server'] }}`替换为`10.10.10.10`.

自动化这一步:

```bash
sed -e "s/{{ pillar\['dns_replicas'\] }}/1/g;"\
"s,\(command = \"/kube2sky\"\),\\1\\"$'\n'"        - --kube_master_url=${KUBERNETES_MASTER},;"\
"s/{{ pillar\['dns_domain'\] }}/cluster.local/g" \
  cluster/addons/dns/skydns-rc.yaml.in > skydns-rc.yaml
sed -e "s/{{ pillar\['dns_server'\] }}/10.10.10.10/g" \
  cluster/addons/dns/skydns-svc.yaml.in > skydns-svc.yaml
```

现在kube-dns pod和service已经启动了:

```bash
kubectl create -f ./skydns-rc.yaml
kubectl create -f ./skydns-svc.yaml
```

使用`kubectl get pods --namespace=kube-system`来检查pod上3个容器正常运行。请注意运行kube-dns的pod是在`kube-system`的命名空间下，不是在`default`下面。

为了监测新的DNS服务正常，我们启动一个pod运行busybox来查找DNS。第一步新建一个`busybox.yaml`文件：

```bash
cat <<EOF >busybox.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
EOF
```

之后启动pod:

```bash
kubectl create -f ./busybox.yaml
```

当pod启动和正常运行后，运行以下命令找到Kubernetes主节点的service的地址。通常情况下地址为10.10.10.1。

```bash
kubectl  exec busybox -- nslookup kubernetes
```

如果一切运行正常，你会看到一下显示:

```console
Server:    10.10.10.10
Address 1: 10.10.10.10

Name:      kubernetes
Address 1: 10.10.10.1
```

## 下一步?

试着运行一下[Kubernetes examples][9].

阅读在Mesos上运行Kubernetes的架构[contrib directory][13].

**注意:** 一些示例需要在集群上安装Kubernetes DNS。我们未来会在本指南里加入如何支持Kubernetes DNS的介绍的。

**注意:** 请注意这里有一些[known issues with the current Kubernetes-Mesos implementation][7].

[1]: http://mesosphere.com/docs/tutorials/run-hadoop-on-mesos-using-installer
[2]: http://mesosphere.com/docs/tutorials/run-spark-on-mesos
[3]: http://mesosphere.com/docs/tutorials/run-chronos-on-mesos
[4]: ../../cluster/addons/dns/README.md
[5]: http://open.mesosphere.com/getting-started/cloud/google/mesosphere/
[6]: http://mesos.apache.org/
[7]: ../../contrib/mesos/docs/issues.md
[8]: https://github.com/mesosphere/kubernetes-mesos/issues
[9]: ../../examples/
[10]: http://open.mesosphere.com/getting-started/cloud/google/mesosphere/#vpn-setup
[11]: ../../cluster/addons/dns/skydns-rc.yaml.in
[12]: ../../cluster/addons/dns/skydns-svc.yaml.in
[13]: ../../contrib/mesos/README.md
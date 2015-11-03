从本地环境起步
-----------------------

**目录**

- [依赖](#依赖)
    - [Linux](#linux)
    - [Docker](#docker)
    - [etcd](#etcd)
    - [go](#go)
- [启动集群](#启动集群)
- [运行容器](#运行容器)
- [运行自定义Pod](#运行自定义Pod)
- [问题定位](#问题定位)
    - [我无法获取服务的IP地址](#i-cannot-reach-service-ips-on-the-network)
    - [我无法创建副本数大于1的replication controller！怎么回事？](#i-cannot-create-a-replication-controller-with-replica-size-greater-than-1--what-gives)
    - [我修改了Kubernetes代码，如何运行？](#i-changed-kubernetes-code-how-do-i-run-it)
    - [kubectl claims to start a container but `get pods` and `docker ps` don't show it.kubectl显示已经启动容器但是`get pods`和`docker ps`都无法查到。](#kubectl-claims-to-start-a-container-but-get-pods-and-docker-ps-dont-show-it)
    - [Pods无法通过域名连接到服务](#the-pods-fail-to-connect-to-the-services-by-host-names)

### 依赖

#### Linux

没有Linux环境？考虑通过[Vagrant](vagrant.md)在本地虚拟机中运行Linux，或者选择一个云服务提供商，如[Google Compute Engine](gce.md)

#### Docker

最低版本[Docker](https://docs.docker.com/installation/#installation)
1.3+。确保Docker守护进程已经运行并可以访问（试试`docker
ps`）。一些Kubernetes组件需要root权限运行，一般情况docker中都没有问题。

#### etcd

你的可执行文件路径中需要有[etcd](https://github.com/coreos/etcd/releases)，请确保已经安装并在``$PATH``环境变量中.

#### go

你的可执行文件路径中需要有[go](https://golang.org/doc/install)，请确保已经安装并在``$PATH``环境变量中.

### 启动集群

In a separate tab of your terminal, run the following (since one needs sudo access to start/stop Kubernetes daemons, it is easier to run the entire script as root):

```sh
cd kubernetes
hack/local-up-cluster.sh
```

This will build and start a lightweight local cluster, consisting of a master
and a single node. Type Control-C to shut it down.

You can use the cluster/kubectl.sh script to interact with the local cluster. hack/local-up-cluster.sh will
print the commands to run to point kubectl at the local cluster.


### Running a container

Your cluster is running, and you want to start running containers!

You can now use any of the cluster/kubectl.sh commands to interact with your local setup.

```sh
cluster/kubectl.sh get pods
cluster/kubectl.sh get services
cluster/kubectl.sh get replicationcontrollers
cluster/kubectl.sh run my-nginx --image=nginx --replicas=2 --port=80


## begin wait for provision to complete, you can monitor the docker pull by opening a new terminal
  sudo docker images
  ## you should see it pulling the nginx image, once the above command returns it
  sudo docker ps
  ## you should see your container running!
  exit
## end wait

## introspect Kubernetes!
cluster/kubectl.sh get pods
cluster/kubectl.sh get services
cluster/kubectl.sh get replicationcontrollers
```


### Running a user defined pod

Note the difference between a [container](../user-guide/containers.md)
and a [pod](../user-guide/pods.md). Since you only asked for the former, Kubernetes will create a wrapper pod for you.
However you cannot view the nginx start page on localhost. To verify that nginx is running you need to run `curl` within the docker container (try `docker exec`).

You can control the specifications of a pod via a user defined manifest, and reach nginx through your browser on the port specified therein:

```sh
cluster/kubectl.sh create -f docs/user-guide/pod.yaml
```

Congratulations!

### Troubleshooting

#### I cannot reach service IPs on the network.

Some firewall software that uses iptables may not interact well with
kubernetes.  If you have trouble around networking, try disabling any
firewall or other iptables-using systems, first.  Also, you can check
if SELinux is blocking anything by running a command such as `journalctl --since yesterday | grep avc`.

By default the IP range for service cluster IPs is 10.0.*.* - depending on your
docker installation, this may conflict with IPs for containers.  If you find
containers running with IPs in this range, edit hack/local-cluster-up.sh and
change the service-cluster-ip-range flag to something else.

#### I cannot create a replication controller with replica size greater than 1!  What gives?

You are running a single node setup.  This has the limitation of only supporting a single replica of a given pod.  If you are interested in running with larger replica sizes, we encourage you to try the local vagrant setup or one of the cloud providers.

#### I changed Kubernetes code, how do I run it?

```sh
cd kubernetes
hack/build-go.sh
hack/local-up-cluster.sh
```

#### kubectl claims to start a container but `get pods` and `docker ps` don't show it.

One or more of the KUbernetes daemons might've crashed. Tail the logs of each in /tmp.

#### The pods fail to connect to the services by host names

The local-up-cluster.sh script doesn't start a DNS service. Similar situation can be found [here](http://issue.k8s.io/6667). You can start a manually. Related documents can be found [here](../../cluster/addons/dns/#how-do-i-configure-it)
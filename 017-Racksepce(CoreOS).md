#Racksapec上部署入门
内容列表
* 入门
* 先决条件
* 供应商：Racksapce
* 编译
* 集群
* 注意点：
* 网络设计
##入门

Supported Version: v0.18.1
In general, the dev-build-and-up.sh workflow for Rackspace is the similar to Google Compute Engine. The specific implementation is different due to the use of CoreOS, Rackspace Cloud Files and the overall network design.

These scripts should be used to deploy development environments for Kubernetes. If your account leverages RackConnect or non-standard networking, these scripts will most likely not work without modification.

NOTE: The rackspace scripts do NOT rely on saltstack and instead rely on cloud-init for configuration.

The current cluster design is inspired by:

corekube
Angus Lees
##先决条件

Python2.7
You need to have both nova and swiftly installed. It's recommended to use a python virtualenv to install these packages into.
Make sure you have the appropriate environment variables set to interact with the OpenStack APIs. See Rackspace Documentation for more details.
##供应商：Rackspace

To build your own released version from source use export KUBERNETES_PROVIDER=rackspace and run the bash hack/dev-build-and-up.sh
Note: The get.k8s.io install method is not working yet for our scripts.
To install the latest released version of Kubernetes use export KUBERNETES_PROVIDER=rackspace; wget -q -O - https://get.k8s.io | bash
##编译

The Kubernetes binaries will be built via the common build scripts in build/.
If you've set the ENV KUBERNETES_PROVIDER=rackspace, the scripts will upload kubernetes-server-linux-amd64.tar.gz to Cloud Files.
A cloud files container will be created via the swiftly CLI and a temp URL will be enabled on the object.
The built kubernetes-server-linux-amd64.tar.gz will be uploaded to this container and the URL will be passed to master/nodes when booted.
##集群

There is a specific cluster/rackspace directory with the scripts for the following steps:

1. A cloud network will be created and all instances will be attached to this network.
    * flanneld uses this network for next hop routing. These routes allow the containers running on each node to communicate with one another on this private network.
2. A SSH key will be created and uploaded if needed. This key must be used to ssh into the machines (we do not capture the password).
3. The master server and additional nodes will be created via the nova CLI. A cloud-config.yaml is generated and provided as user-data with the entire configuration for the systems.
4. We then boot as many nodes as defined via $NUM_MINIONS.

##注意点：

* 脚本设置eth2成为容器可通过它通信的云网络。
* config-default.sh中条目的数量可通过环境变量覆盖。 
* 旧版本请选择:
    * 使用git chechout V0.9 同步到V0.9
    * 下载V0.9的一个快照
    * 使用 git checkout V0.3 同步到v0.3
    * 下载v0.3的一个快照

##网络设计

* eth0 - servers/containers访问网络的公有接口
* eth1 - ServiceNet - 集群内部通信 (k8s, etcd, etc) 通过这个接口。cloud-confi文件使用特殊CoreOS标识符$ private_ipv4配置服务
* eth2 - Cloud Network - k8s pods使用这个和其他pods通信。服务代理通过这个接口传输流量。
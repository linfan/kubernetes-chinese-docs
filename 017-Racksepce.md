# **Racksapec上部署入门**
内容列表
* [入门介绍](#入门介绍)
* [部署需求](#部署需求)
* [供应商：Racksapce](#供应商：Racksapce)
* [编译](#编译)
* [集群](#集群)
* [注意点：](#注意点：)
* [网络设计](#网络设计)

# **入门介绍**

* 支持版本: v0.18.1

一般来说，Racksapce的dev-build-and-up.sh工作流程类似于Google Compute Engine。具体实现是不同的，因为Rackspace使用CoreOS， Rackspace云文件和整体网络设计。

这些脚本应该用于部署Kubernetes的开发环境。 如果你的账号利用RackConnect或者非标准的网络，若不修改这些脚本，这些脚本将无法使用。

注意: rackspace脚本不依赖```saltstack```而依赖于配置文件cloud-init。 

当前集群设计受如下启发：

* [corekube](https://github.com/metral/corekube)
* [Angus Lees](https://github.com/anguslees/kube-openstack)

# **部署需求**

1. Python2.7
2. 需要安装 ```nova```和```swiftly```。推荐使用python virtualenv来安装这些包。
3. 确认已经有与OpenStack APIs交互的合适的环境变量。获取更详细信息请查看 [Rackspace文档](http://docs.rackspace.com/servers/api/v2/cs-gettingstarted/content/section_gs_install_nova.html)。

# **供应商：Rackspace**

* 使用 ``` KUBERNETES_PROVIDER=rackspace```从源码编译自己的发布版本并且运行``` bash hack/dev-build-and-up.sh```
* 注意: 我们脚本中还没有提供get.k8s.io安装方法。
    * 请使用 ```export KUBERNETES_PROVIDER=rackspace; wget -q -O - https://get.k8s.io | bash```来安装Kubernetes最近发布版本

# **编译**

1. Kubernetes二进制文件将通过```build/```下的的通用编译脚本编译。
2. 如果已经设置了环境变量```ENV KUBERNETES_PROVIDER=rackspace```， 脚本将上传```kubernetes-server-linux-amd64.tar.gz```到云文件。
3. 通过swiftly创建一个云文件容器，且在这个对象内启用一个临时URL。
4. 编译过的```kubernetes-server-linux-amd64.tar.gz```将上传到这个容器内，当master/nodes启动时这个URL将传到master/nodes。

# **集群**

有一个特殊的```cluster/rackspace```脚本目录有如下步骤：
1. 创建一个云网络并且所有的实例附属于这个网络。
    * flanneld使用这个网络作为下一跳路由。这些路由使每个节点上的容器和这个私有网络内其他的容器之间通信。
2. 如果需要，将创建且上传一个SSH key。这个key必须用于ssh登录机器（我们不捕获密码）
3. 通过```nova``` CLI创建主节点server和额外节点。生成一个```cloud-config.yaml```作为户数据和整个系统的配置
4. 然后，我们通过```$NUM_MINIONS```定义的数量启动节点。

# **注意点**

* 脚本设置```eth2```成为云网络，容器可通过它通信。
* ```config-default.sh```中条目的数量可通过环境变量覆盖。 
* 旧版本请选择:
    * 使用```git chechout v0.9```同步到```v0.9```
    * 下载```snapshot of v0.9```
    * 使用```git checkout v0.3```同步到```v0.3```
    * 下载```snapshot of v0.3```

# **网络设计**

* eth0 - servers/containers访问网络的公有接口
* eth1 - ServiceNet - 集群内部通信 (k8s, etcd, etc) 通过这个接口。```cloud-config```文件使用特殊CoreOS标识符```$ private_ipv4```配置服务
* eth2 - Cloud Network - k8s pods使用这个和其他pods通信。服务代理通过这个接口传输流量。
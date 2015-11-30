#vSphere的入门指南

下面的示例使用了4个虚拟工作节点和1个虚拟主机（即集群中一共有5台虚拟机）来创建一个Kubernetes集群。集群是通过你的工作站（或任何你觉得方便的地方）来安装和控制的。

**内容列表**

- [前提条件](#prerequisites)
- [安装](#setup)
- [启动集群](#starting-a-cluster)
- [其他：部署失败调试](#extra-debugging-deployment-failure)

### 前提条件

1. 需要有一台ESXi机器或vCenter实例的管理员权限。 
2. 需要先安装Go（1.2或以上版本）。下载地址： [www.golang.org](http://www.golang.org).
3. 需要在环境变量中添加`GOPATH`并将`$GOPATH/bin` 添加到`PATH`中。

   ```sh
   export GOPATH=$HOME/src/go
   mkdir -p $GOPATH
   export PATH=$PATH:$GOPATH/bin
   ```

4. 安装govc工具来和ESXi/vCenter进行交互：

   ```sh
   go get github.com/vmware/govmomi/govc
   ```

5. 需要预先下载或编译[二进制版本](binary_release.md)

### Setup

下载一个预置了Debian 7.7 的VMDK，把它作为基础镜像来使用：

```sh
curl --remote-name-all https://storage.googleapis.com/govmomi/vmdk/2014-11-11/kube.vmdk.gz{,.md5}
md5sum -c kube.vmdk.gz.md5
gzip -d kube.vmdk.gz
```

将VMDK导入vSphere中：

```sh
export GOVC_URL='user:pass@hostname'
export GOVC_INSECURE=1 # If the host above uses a self-signed cert
export GOVC_DATASTORE='target datastore'
export GOVC_RESOURCE_POOL='resource pool or cluster with access to datastore'

govc import.vmdk kube.vmdk ./kube/
```

验证VMDK是否已经正确上传并扩展到~3GiB：

```sh
govc datastore.ls ./kube/
```

检查文件`cluster/vsphere/config-common.sh`是否已经配置了必填参数。该导入镜像的游客登录帐号为`kube:kube`。

### 启动集群

现在继续部署Kubernetes。整个过程需要大约10分钟。  

```sh
cd kubernetes # Extracted binary release OR repository root
export KUBERNETES_PROVIDER=vsphere
cluster/kube-up.sh
```

参见根目录下的README和《谷歌计算引擎入门指南》。一旦你成功到达了这一步，你的vSphere Kubernetes就可以像其他Kubernetes集群一样正常工作了。

**开始享受Kubernetes之旅吧！**

### 其他：部署失败调试

`kube-up.sh`输出可以查看部署集群中各个虚拟机的ip地址，你可以用`kube`账户登录到任何虚拟机上查看并找出到底发生了什么状况。（通过你的SSH密钥或密码'kube'来登录）

# **Service Accounts集群管理指南**

*这是对Service Accounts的集群管理指南，详情[Service Accounts用户指南](http://kubernetes.io/v1.1/docs/user-guide/service-accounts.html).*

*对用户授权和用户账户的支持正在计划中，还没有完全完成。有时候，不完整的一些特性，可以更好的描述Service Accounts。*

## **User Accounts和Service Accounts**

基于以下原因，Kubernetes区分了User Accounts和Service Accounts：
- 用户账户针对人，服务账户针对运行在Pod的进程；
- 用户账户是全局的，其名字必须在一个集群的所有Namespace中是唯一的。未来的用户资源将不被命令，但是服务账户是可以被命名的。
- 通常情况下，集群的用户账户可以从一个企业数据库同步。在企业数据库中，新建的账户需要特殊权限，而且绑定到复杂业务流程。新建服务账户可以更加轻量级，允许集群用户为特殊任务创建服务账户（比如，最小权限规则）。
- Auditing considerations for humans and service accounts may differ.
- 复杂系统的配置包包含对系统组件的各种服务账户的定义。因为服务账户可以创建ad-hoc，可以命名，配置是便携式的。

## **服务账户自动化**

三个独立的组件合作实现对服务账户的自动化。
- Service账户接入控制器
- Token控制器
- Service账户控制器

### **服务账户Admission Controller**

Pod的修改是[Admission Controller]()插件实现的，该插件是apiserver的一部分。插件的创建和更新，会同步修改Pod。当插件状态是active（大多版本中，默认是active），创建或者修改Pod会遵循以下流程：

1.	如果该Pod没有ServiceAccount集，将ServiceAccount设为default
2.	ServiceAccount必须有存在的Pod引用，否则拒绝该ServiceAccount
3.	如果该Pod不包含任何ImagePullSecrets，然后该ServiceAccount的ImagesPullSecrets会被加入Pod。
4.	添加一个volume到该Pod，包含API访问的令牌。
5.	添加一个volume到该Pod的每一个容器，挂载/var/run/secrets/kubernetes.io/serviceaccount

###** Token Controller**

TokenController做为controller-manager的一部分异步运行。
- 检查serviceAccount的创建，并且创建一个关联的Secret，允许API访问。
- 检查serviceAccount的删除，并且删除所有相关ServiceAccountToken Secrets
- 检查额外秘钥，and ensures the referenced ServiceAccount exists, and adds a token to the secret if needed
- 检查秘钥的删除，如果有必要，从相关的ServiceAccount中移除参考信息。

#### 创建额外的API标记

一个控制循坏要确保每一个服务账户存在一个API标记。为一个服务账户创建一个额外的API标记，类型是ServiceAccountToken，含有一个注释去引用到对应的服务账户。该控制器使用如下的标记去更新：
```Json
secret.json:
{
    "kind": "Secret",
    "apiVersion": "v1",
    "metadata": {
        "name": "mysecretname",
        "annotations": {
            "kubernetes.io/service-account.name": "myserviceaccount"
        }
    },
    "type": "kubernetes.io/service-account-token"
}

kubectl create -f ./secret.json
kubectl describe secret mysecretname
```
#### 删除、废弃一个服务账户标记
```
kubectl delete secret mysecretname
```
### **服务账户控制器**

Service Account Controller管理Namespace中的ServiceAccount，确保每一个“default”的ServiceAccount存在于每一个活动空间中。

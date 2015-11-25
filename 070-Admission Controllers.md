# **Admission Controller**
目录
- Admission Controller『参见以下内容』
 - Admission Controller是什么？『参见下文“什么是Admission Controller”』
 - 为什么使用Admission Controller？『参见下文“为什么使用Admission Controller”』
 - 如何使用接入控制插件？『参见下文“如何接入该插件”』
 - 每个插件的功能『参见下文“每个插件的功能是什么”』
 - AlwaysAdmit『参见下文“AlwaysAdmin插件”』
 - AlwaysDeny『参见下文“AlwaysDeny插件”』
 - DenyExecOnPrivileged (废弃)『参见下文“DenyExecOnPrivileged (废弃)插件”』
 - DenyEscalatingExec『参见下文“DenyEscalatingExec插件”』
 - ServiceAccount『参见下文“ServiceAccount插件”』
 - SecurityContextDeny『参见下文“SecurityContextDeny插件”』
 - ResourceQuota『参见下文“ResourceQuota插件”』
 - LimitRanger『参见下文“LimitRanger插件”』
 - NamespaceExists (废弃)『参见下文“NamespaceExists（废弃）插件”』
 - NamespaceAutoProvision (废弃)『参见下文“NamespaceAutoProvision (废弃)插件”』
 - NamespaceLifecycle『参见下文“NamespaceLifecycle插件”』
 - 是否有推荐的插件集合？『参见下文“是否有推荐的插件集合”』

## **什么是Admission Controller？**
Admission Controller插件是一段代码，其拦截Kubernetes API服务的请求早于对象的持久性，但是在请求的认证和授权之后。插件代码位于API服务进程中，会编译成二进制以便此时使用。

集群在接受一个请求之前，每一个Admission Controller插件都会按序运行。如果这个序列中的某个插件拒绝该请求，则整个的请求都会被立刻拒绝，返回一个错误给用户。

Admission Controller插件在某些情况下也许会改变传进来的对象，配置系统默认值。此外，Admission Controller插件也许会改变请求处理中的部分相关资源去做些事情，比如增量配额的使用。

## **为什么使用Admission Controller？**
Kubernetes中许多高级功能需要激活Admission Controller插件，以便更好的支持该功能。总之，没有正确配置Admission Controller插件的Kubernetes API服务是不完整的服务，很多用户期望的服务是不支持的。

## **如何接入该插件？**
Kubernetes API 服务器提供了一个参数，admission-control，用逗号分隔，在集群中修改对象之前，调用许可控制选项的有序列表。

## **每个插件的功能是什么？**
### ****AlwaysAdmin插件****
使用插件本身处理所有请求。

### ****AlwaysDeny插件****
拒绝所有请求，主要用于测试。

### ****DenyExecOnPrivileged (废弃)插件****
如果一个Pod有一个特权Container，该插件就会拦截所有的请求，在该Pod中执行一个命令。

如果你的集群支持特权Container，而且你想要限制终端用户在那些Container中执行命令的权限，我们强烈建议使用该插件。

该功能已经合并到DenyEscalatingExec插件『参见下文“DenyEscalatingExec插件”』)

### ****DenyEscalatingExec插件****
This plug-in will deny exec and attach commands to pods that run with escalated privileges that allow host access. This includes pods that run as privileged, have access to the host IPC namespace, and have access to the host PID namespace.
If your cluster supports containers that run with escalated privileges, and you want to restrict the ability of end-users to exec commands in those containers, we strongly encourage enabling this plug-in.

### ****ServiceAccount插件****
这个插件实现了[serviceAccounts](http://kubernetes.io/v1.1/docs/user-guide/service-accounts.html)的自动化。如果你打算使用Kubernetes ServiceAccount对象，我们强烈建议使用该插件。

### ****SecurityContextDeny插件****

[SecurityContext](http://kubernetes.io/v1.1/docs/user-guide/security-context.html)定义了一些不适用于Container的选项，这个插件将会拒绝任何含有该SecurityContext的Pod。

### ****ResourceQuota插件****

该插件会检查传入的请求，确保其不违反任何Namespace中ResourceQuota对象枚举的约束条件。如果你在Kubernetes开发中正在使用ResourceQuota对象，你必须使用该插件实现配额约束条件。

查看[resourceQuota设计文档](http://kubernetes.io/v1.1/docs/design/admission_control_resource_quota.html)和[Resource Quota示例](http://kubernetes.io/v1.1/docs/admin/resourcequota/README.html)了解更多细节。

强烈建议配置该插件在Admission Controller插件的序列中。This is so that quota is not prematurely incremented only for the request to be rejected later in admission control。

### ****LimitRanger插件****

该插件会检查传入的请求，确保其不违反任何Namespace中LimitRange对象枚举的约束条件。如果你在Kubernetes开发中正在使用LimitRange对象，你必须使用该插件实现约束条件。LimitRange也经常用于Pod中默认资源请求，不会指定哪一个请求。目前，默认LimitRange，在默认的Namespace中对所有的Pod，需要0.1CPU。

查阅[LimitRange设计文档](http://kubernetes.io/v1.1/docs/design/admission_control_limit_range.html)和[用例](http://kubernetes.io/v1.1/docs/admin/limitrange/)，了解更多详细信息。

### ****NamespaceExists（废弃）插件****
该插件会检查所有传入的请求，尝试在Kubernetes Namespace中创建资源，如果该Namespace不是当前创建的，该插件会拒绝这个请求。我们强烈建议使用该插件，确保数据的完整性。

Admission Controller的该功能已经并入NamespaceLifecycle插件。

### ****NamespaceAutoProvision (废弃)插件****

该插件会检查所有传入的请求，尝试在Kubernetes Namespace中创建资源，如果Namespace不存在，会创建一个新的Namespace。

我们强烈建议NamespaceExists优先级高于NamespaceAutoProvision。

### ****NamespaceLifecycle插件****
如果Namespace已经终止，则不能在其中创建新的Namespace，该插件会强制该操作。

删除一个Namespace会终止一系列操作，移除该Namespace中所有对象（Pod，服务等等）。为了加强该过程的完整性，我们建议使用该插件。

## **是否有推荐的插件集合？**

是的
Kubernetes1.0，我们强烈建议使用如下的许可控制插件集合（order matters）：
--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota

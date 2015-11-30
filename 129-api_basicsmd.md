
# **Kubernetes API**

[用户指南](https://github.com/kubernetes/kubernetes/blob/master/docs/user-guide/README.md)记录了主系统和API概念。

[API规范文档](https://github.com/kubernetes/kubernetes/blob/master/docs/devel/api-conventions.md)描述了API整体规范。

Kubernetes通过[Swagger](http://swagger.io/)记录API所有细节。Kubernetes apiserver (aka "master")提供了一个API接口用于获取Kubernetes API的[Swagger spec](https://github.com/swagger-api/swagger-spec/tree/master/schemas/v1.2)，默认在路径```/swaggerapi```下，```/swagger-ui```是可以使用浏览器查看API文档的UI
。我们会定期更新[静态生成UI](http://kubernetes.io/third_party/swagger-ui/)。

远程访问API在[访问文档](/kubernetes/kubernetes/blob/master/docs/admin/accessing-the-api.md)有讨论。

Kubernetes API是系统描述性配置的基础。 [Kubectl](/kubernetes/kubernetes/blob/master/docs/user-guide/kubectl/kubectl.md)命令行工具被用于创建，更新，删除，获取API对象。

Kubernete通过API资源存储自己序列化状态(现在存储在[etcd](https://coreos.com/docs/distributed-configuration/getting-started-with-etcd/))。

Kubernetes被分成多个组件，各部分通过API相互交互。 

##API更改

根据经验，任何成功的系统都需要成长和改变，可能是一个新的示例出现，也可能是对已有系统进行更改。因此，我们希望Kubernetes API也可以持续的改变和成长。在较长一段时间内，我们不打算中断已有客户端。一般情况，经常会有新的API资源和新的资源字段增加。删除资源或者字段会有一个跟踪废弃流程。删除功能有一个小的废弃策略叫做TBD，但是Kubernetes到达1.0版本时，将会有一个详细的策略代替。

如何构成一个兼容性的改变以及如何更改API的详细信息都在[API变化文档](/kubernetes/kubernetes/blob/master/docs/devel/api_changes.md)

##API版本

为了使删除字段或者重构资源表示更加容易，Kubernetes支持多个API版本。每一个版本都在不同API路径下，例如 ```/api/v1``` 或者```/apis/extensions/v1beta1```.

我们选择版本时时根据API而不是根据资源或者字段来确认的，API为系统资源和行为提供了一个清晰的，一致性的视图，并且可以控制访问生命周期结束和／或测试的APIs。


注意API版本控制和软件版本控制只能间接的相关联。文档[API and release versioning proposal](/kubernetes/kubernetes/blob/master/docs/design/versioning.md) 描述了API版本控制和软件版本控制两者之间的关系。

不同的API版本会有不同级别的稳定性和支持。每个级别的详细描述包含在文档[API Changes documentation](/kubernetes/kubernetes/blob/master/docs/devel/api_changes.md#alpha-beta-and-stable-versions)中。 内容主要概括如下：

* Alpha 级别:
    * 版本名称包含了 ```alpha``` (e.g. ```v1alpha1``).
    * 可能是有问题的。实现的功能可能隐含问题，功能默认是不可用的。
    * 支持的功能可能在没有通知的情况下随时删除。
    * API的更改可能存在冲突，但是在后续的软件发布中不会有任何通知。
    * 由于bugs风险的增加和缺乏长期的支持，推荐在短暂的集群测试中使用。
* Beta 级别:
    * 版本名称包含了 ```beta``` (e.g. ```v2beta3```).
    * 代码已经测试过。实现的功能认为是安全的，功能默认是可用的。
    * 所有已支持的功能不会被删除，细节可能会发生变化。
    * 对象的模式和／或定义在随后的beta版本或者稳定版本可能以不兼容的方式改变。在这种情况下，Kubernetes会提供合并到下一个版本的指南。这一过程中可能会要求删除，编译和重新创建API对象。编译过程中可能会要求一些选择。根据不同的功能可能会需要一些应用的下载时间。
    * 在随后的发布中存在一些非兼容性的更改，所已推荐在不重要的非商业化的情况下使用。如果你已经有多个集群，并且可以单独更新，你可以放宽这个限制。 
    * **请尝试我们的bete版本功能并且给出反馈！因为已经是beta版本，对于我们来说太多的更改可能不太实用 **
* Stable 级别:
    * 版本名称中包含 vX 这里的X是一个整数.
    * 稳定版本的功能在后续的版本发布中会一直存在。

##API群组

为了使扩展Kubernetes API更加简单，我们正在实现[API groups](/kubernetes/kubernetes/blob/master/docs/proposals/api-group.md). API群组是一些可以读和／或更改相同基础资源的简单的不同的接口。API群组被定义在一个REST路径下，在```apiVersion```的一个序列化对象的字段。

当前有两个API群组在使用：

1. "core"群组，在REST路径```/api/v1```下，但不是apiVersion字段的一部分。e.g. ```apiVersion: v1 ```。
2. "extensions"群组, 在REST路径```/apis/extensions/$VERSION```下, 并且使用```apiVersion: extensions/$VERSION ```(e.g. 当前是```apiVersion: extensions/v1beta1```).

未来我们希望有更多的API群组，并且所有的群组都在REST路径
```/apis/$API_GROUP```下，并且使用```apiVersion: $API_GROUP/$VERSION```。我们希望将来有一种方式支持三方可以创建自己的[API groups](design/extending-api.md)，并且可以避免命名冲突。

## 扩展资源

默认情况下Jobs, Ingress和HorizontalPodAutoscalers都是可用的。其他的扩展资源通过设置apiserver的runtime-config使其可用。runtime-config多个值可通过都逗号分开。例如：设置部署可用，jobs不可用命令，```--runtime-config=extensions/v1beta1/deployments=true,extensions/v1beta1/jobs=false```

##v1beta1, v1beta2, and v1beta3已经废弃；请转到v1 ASAP

2015年6月4日，Kubernetes v1 API已经默认可以使用。v1beta1和v1beta2 APIs在2015年6月1日删除。v1beta3计划会在2015年6月6日删除。

###v1转换技巧(从v1beta3)

我们已经将所有文档和例子转换成v1版本。并且写了一个简单的 [API转换工具](/kubernetes/kubernetes/blob/master/docs/admin/cluster-management.md#switching-your-config-files-to-a-new-api-version)来进行简单的转换。使用```kubectl create --validate```命令可以使你的json或者yaml按照Swagger spec规范有效。

v1beta3和v1版本之间最重要的不同是服务的改变。

* ```service.spec.portalIP```属性重命名为```service.spec.clusterIP```。
* ```service.spec.createExternalLoadBalancer```已经被删除。 定义了```service.spec.type: "LoadBalancer"```来代替，用于创建外部负载均衡。
* ```service.spec.publicIPs```属性已经被废弃，现在叫做```service.spec.deprecatedPublicIPs```。当v1beta3删除时，这个属性也会被完全删除。这个字段的多数用户使用这个字段来公开nodes上的services端口。现在这些用户应该使用```service.spec.type: "NodePort"```来代替。 获取更多信息可阅读[External Services](/kubernetes/kubernetes/blob/master/docs/user-guide/services.md#external-services)。如果这些还是不能满足你的需求，请提交issue或者联系@thockin。

v1beta3和v1版本之间其他的不同：

* ```pod.spec.containers[*].privileged```和```pod.spec.containers[*].capabilities```属性已经合并到```pod.spec.containers[*].securityContext```属性。可以查看文档[Security Contexts](/kubernetes/kubernetes/blob/master/docs/user-guide/security-context.md)。
* ```pod.spec.host```属性被重命名为```pod.spec.nodeName```。
* ```endpoints.subsets[*].addresses.IP```被重命名为```endpoints.subsets[*].addresses.ip```。
* ```pod.status.containerStatuses[*].state.termination```和```pod.status.containerStatuses[*].lastState.termination```属性被分别重命名为```pod.status.containerStatuses[*].state.terminated```和```pod.status.containerStatuses[*].lastState.terminated```。
* ```pod.status.Condition```属性被重命名为```pod.status.conditions```。
* ```status.details.id```属性被重命名为```status.details.name```。

v1beta3转换技巧(从v1beta1/2)

v1beta1/2和v1beta3重要不同点:

* 资源```id```现在称作``name```.
* ```name```，```labels```，```annotations```，和其他元数据现在内嵌在一个称作```metadata```的map中
* ```desiredState``` 现在称作```spec```，```currentState``` 现在称作```status``
* ```/minions``` 已经被移到```/nodes```, 且资源包括各种 ```Node```
* namesapce是必要条件(对于所有资源namesapce)且已经从一个URL参数改为路径：```/api/v1beta3/namespaces/{namespace}/{resource_collection}/{resource_name}```. 如果之前你没有使用过namespace，在这可以使用```default```.
* 所有资源收集器的名字已经小写 - 已经废弃 ```replicationControllers```，使用 ```replicationcontrollers``来代替。
* 查看某个资源的改变，可打开一个HTTP或者Websocket连接查询，并且提供```?watch=true ```请求参数和```resourceVersion```参数进行查询。
* ```labels```查询参数已经被重命名为```labelSelector```。
* ```fields```查询参数已经被重命名为```fieldSelector```。
* Container中的```entrypoint```已经被重命名为```command```，而```command```被重命名为```args```.
* Container, volume, and node资源的定义已经被内嵌到一个maps (e.g., ```resources{cpu:1}```)并没有作为独立的字段，并且资源数值支持[后缀](user-guide/compute-resources.html#specifying-resource-quantities)而不是固定的(e.g., milli-cores)。
* 重新启动策略简单的表示为字串(e.g., ```"Always"```)而不是作为一个内嵌的map(```always{}```).
* Pull策略从```PullAlways```, ```PullNever```，和```PullIfNotPresent```变成了```Always```，```Never```，和```IfNotPresent```。
* volume ```source```被内联到```volume```而非嵌套。
* 宿主机volumes已经从```hostDir```变成了```hostPath```，这样做的好处是，挂载的可以是一个文件也可以是一个路径。
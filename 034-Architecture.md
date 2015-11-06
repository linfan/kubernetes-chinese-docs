# Kubernetes架构


一个运行着的Kubernetes集群包含node agents(即kubelet)和主要的组件（API，调度器等等），其构建于分布式存储的基础之上。这个图显示了我们的理想的最终状态，尽管我们还在对一些部分做改动，如让kubelet自身运行在容器之内，和让调度器百分之百的插件式。
!p[架构图](architecture.png?raw=true "Architecture overview")



## Kubernetes Node（Kubernetes节点）


当研究系统的架构的时候，我们会将其划分成运行子worker节点上的service和那些组成集群级别控制面板的service。


Kubernetes Node拥有能运行应用容器的必要服务和从主系统管理的service。


每一个node当然运行着Docker。Docker负责处理下载镜像，运行容器的细节。

### `kubelet`


kubelet管理着pod和它们的容器，它们的镜像，它们的卷，等等。

### `kube-proxy`


每一个node也运行着一个简单的网络代理和负载均衡器。这反映出定义在Kubernetes API中的每个节点的service能做简单的TCP和UDP流转发（轮流的方式）到一组后端。
    

service的端点现在是通过DNS或者通过环境变量来发现的。这些变量解析到有service代理管理着的端口。
    

## Kubernetes控制面板


Kubernetes控制面板被分成一组组件。目前，它们都运行在一个单一的master节点上，但是这很快就会改变，以支持高可用的集群。这些节点一起工作提供了一个集群统一的视图。


### `etcd`


所有持久化的master状态保存在一个etcd的节点中。这给保存配置数据稳定性提供了很好的存储。有watch的支持，协作的组件可以在更改时很快的被通知到。

### Kubernetes API Server   



apiserver服务着Kubernetes API。其目标是作为简单的CRUD式的服务，然后绝大多数/所有的业务逻辑都在单独的组件或者插件里面实现。他主要处理REST操作，做数据验证，并更新对应的etcd里面的对象（然后最终其他的存储）。
### Scheduler


调度器通过bingding API将没有调度的pod绑定到node。 调度器是插件式的，我们期望支持多种集群的调度器，最终在未来甚至支持用户提供额的调度器。
### Kubernetes Controller Manager Server


所有的其他集群级别的功能目前是通过Controller Manager来执行的。例如Endpoint对象通过endpoint控制器偶来创建和更新，node是通过node控制器来发现管理和监控的。这些可能会最终被拆分到单独的组件，以让他们可以独立的插件化。

replication控制器是一个在简单API的上层的机制。我们最终计划将其移植成可插入式的机制，一旦有一个被实现了。

# 应用程序相关的故障排查

这个小节将帮助读者调试部署在Kubernetes的应用程序运行不正常的情况。本篇并不会包含如何调试集群组建过程中出现错误的问题，这些内容在文档的下一节中进行介绍。

## 内容提要

- 应用程序故障排查
	- 常见问题
	- 故障诊断
		- 排查Pods的故障
			- Pod始终处于Pending状态
			- Pod始终处于Waiting状态
			- Pod一直崩溃或运行不正常
			- Pod在运行但没有如预期工作
		- 排查Replication Controllers的故障
		- 排查Services的故障
			- 服务没有端点信息
			- 网络流量没有正确的转发
			- 其他问题

## 常见问题

一些经常被提到的问题都更新在Kubernetes项目GitHub的[常见问题](https://github.com/kubernetes/kubernetes/wiki/User-FAQ)页面。

## 故障诊断

故障排查的第一步是对故障进行分类。一般来说，应用程序的故障可以分为下面几个方面：

- Pods的故障
- ReplicationControllers的故障
- Services的故障

### 排查Pods的故障

检查Pod的问题首先应该了解Pod所处的状况。下面这个命令能够获得Pod当前的状态和近期的事件列表：

```
$ kubectl describe pods ${POD_NAME}
```

确认清楚在Pod以及其中每一个容器的状态，是否都处于`Runing`？通过观察容器的已运行时间判断它是否刚刚才重新启动过？

根据不同的运行状态，用户应该采取不同的调查措施。

#### Pod始终处于Pending状态

如果Pod保持在`Pending`的状态，这意味着它无法被正常的调度到一个节点上。通常来说，这是由于某种系统资源无法满足Pod运行的需求。观察刚才`kubectl describe`命令的输出内容，其中应该包括了能够判断错误原因的消息。常见的原因有以下这些：

- **系统没有足够的资源**：你也许已经用尽了集群中所有的CPU或内存资源。对于这种情况，你需要清理一些不在需要的Pod，调整它们所需的资源量，或者向集群中增加新的节点。在[计算资源文档](https://github.com/kubernetes/kubernetes/blob/master/docs/user-guide/compute-resources.md#my-pods-are-pending-with-event-message-failedscheduling)有更详细的说明。

- **用户指定了`hostPort`**：通过`hostPort`用户能够将服务暴露到指定的主机端口上，但这样会限制Pod能够被调度运行的节点。在大多数情况下，`hostPort`配置都是没有必要的，用户应该采用Service的方式暴露其对外的服务。如果用户确实必须使用`hostPort`的功能，那么此Pod最多只能部署到和集群节点相同的数目。

#### Pod始终处于Waiting状态

Pod处在`Waiting`的状态，说明它已经被调度分配到了一个工作节点，然而它无法在那个节点上运行。同样的，在刚才`kubectl describe`命令的输出内容中，应该包含有更详细的错误信息。最经常导致Pod始终`Waiting`的原因是无法下载所需的镜像（译者注：由于国内特殊的网络环境，这类问题出现得特别普遍）。用户可以从下面三个方面进行排查：

- 请确保正确书写了镜像的名称
- 请检查所需镜像是否已经推送到了仓库中
- 手工的在节点上运行一次`docker pull <镜像名称>`检测镜像能否被正确的拉取下来

#### Pod一直崩溃或运行不正常

首先，查看正在运行容器的日志。

```
$ kubectl logs <Pod名称> <Pod中的容器名称>
```

如果容器之前已经崩溃过，通过以下命令可以获得容器前一次运行的日志内容。

```
$ kubectl logs --previous <Pod名称> <Pod中的容器名称>
```

此外，还可以使用`exec`命令在指定的容器中运行任意的调试命令。

```
$ kubectl exec <Pod名称> -c <Pod中的容器名称> -- <任意命令> <命令参数列表...>
```

值得指出的是，对于只有一个容器的Pod情况，`-c <Pod中的容器名称>`这个参数是可以省略的。

例如查看一个运行中的Cassandra日志文件内容，可以参考下面这个命令：

```
$ kubectl exec cassandra -- cat /var/log/cassandra/system.log
```

要是上面的这些办法都不奏效，你也可以找到正在运行该Pod的主机地址，然后使用SSH登陆进去检测。但这是在确实迫不得已的情况下才会采用的措施，通常使用Kubernetes暴露的API应该足够获得所需的环境信息。因此，如果当你发现自己不得不登陆到主机上去获取必要的信息数据时，不妨到Kubernetes的GitHub页面中给我们提一个功能需求（Feature Request），在需求中附上详细的使用场景以及为什么当前Kubernetes所提供的工具不能够满足需要。

#### Pod在运行但没有如预期工作

如果Pod没有按照预期的功能运行，有可能是由于在Pod描述文件中（例如你本地机器的`mypod.yaml`文件）存在一些错误，这些配置中的错误在Pod时创建并没有引起致命的故障。这些错误通常包括Pod描述的某些元素嵌套层级不正确，或是属性的名称书写有误（这些错误属性在运行时会被忽略掉）。举例来说，如果你把`command`属性误写为了`commnd`，Pod仍然会启动，但用户所期待运行的命令则不会被执行。

对于这种情况，首先应该尝试删掉正在运行的Pod，然后使用`--validate`参数重新运行一次。继续之前的例子，当执行`kubectl create --validate -f mypod.yaml`命令时，被误写为`commnd`的`command`指令会导致下面这样的错误：

```
I0805 10:43:25.129850   46757 schema.go:126] unknown field: commnd
I0805 10:43:25.129973   46757 schema.go:129] this may be a false alarm, see https://github.com/kubernetes/kubernetes/issues/6842
pods/mypod
```

下一件事是检查当前apiserver运行Pod所使用的Pod描述文件内容是否与你想要创建的Pod内容（用户本地主机的那个yaml文件）一致。比如，执行`kubectl get pods/mypod -o yaml > mypod-on-apiserver.yaml`命令将正在运行的Pod描述文件内容导出来保存为`mypod-on-apiserver.yaml`，并将它和用户自己的Pod描述文件`mypod.yaml`进行对比。由于apiserver会尝试自动补全一些缺失的Pod属性，在apiserver导出的Pod描述文件中有可能比本地的Pod描述文件多出若干行，这是正常的，但反之如果本地的Pod描述文件比apiserver导出的Pod描述文件多出了新的内容，则很可能暗示着当前运行的Pod使用了不正确的内容。

### 排查Replication Controllers的故障

Replication Controllers的逻辑十分直白，它的作用只是创建新的Pod副本，仅仅可能出现的错误便是它无法创建正确的Pod，对于这种情况，应该参考上面的『排查Pods的故障』部分进行检查。

也可以使用`kubectl describe rc <控制器名称>`来显示与指定Replication Controllers相关的事件信息。

### 排查Services的故障

Service为一系列后端Pod提供负载均衡的功能。有些十分常见的故障都可能导致Service无法正常工作，以下将提供对调试Service相关问题的参考。

首先，检查Service连接的服务提供端点（endpoint），对于任意一个Service对象，apiserver都会为其创建一个端点资源（译者注：即提供服务的IP地址和端口号）。

这个命令可以查看到Service的端口资源：

```
$ kubectl get endpoints <Service名称>
```

请检查这个命令输出端点信息中的端口号与实际容器提供服务的端口号是否一致。例如，如果你的Service使用了三个Nginx容器的副本（replicas），这个命令应该输出三个不同的IP地址的端点信息。

#### 服务没有端点信息

如果刚刚的命令显示Service没有端点信息，请尝试通过Service的选择器找到具有相应标签的所有Pod。假设你的Service描述选择器内容如下：

```
...
spec:
  - selector:
     name: nginx
     type: frontend
```

可以使用以下命令找出相应的Pod：

```
$ kubectl get pods --selector=name=nginx,type=frontend
```

找到了符合标签的Pod后，首先确认这些被选中的Pod是正确，有无错选、漏选的情况。

如果被选中的Pod没有问题，则问题很可能出在这些Pod暴露的端口没有被正确的配置好。要是一个Service指定了`containerPort`，但被选中的Pod并没有在配置中列出相应的端口，它们就不会出现在端点列表中。

请确保所用Pod的`containerPort`与Service的`containerPort`配置信息是一致的。

#### 网络流量没有正确的转发

如果你能够连接到Service，但每次连接上就立即被断开，同时Service的端点列表内容是正确的，很可能是因为Kubernetes的kube-proxy服务无法连接到相应的Pod。

请检查以下几个方面：

- Pod是否在正常工作？从每个Pod的自动重启动次数可以作为有用的参考信息，前面介绍过的Pod错误排查也介绍了更详细的方法
- 能够直接连接到Pod提供的服务端口上吗？不妨获取到Pod的IP地址，然后尝试直接连接它，以验证Pod本身十分运行正确。
- 容器中的应用程序是否监听在Pod和Service中配置的那个端口？Kubernetes不会自动的映射端口号，因此如果应用程序监听在8080端口，务必保证Service和Pod的`containerPort`都配置为了8080。

#### 其他问题

如果上述的这些步骤还不足以解答你所遇到的问题，也就是说你已经确认了相应的Service正在运行，并且具有恰当的端点资源，相应的Pod能够提供正确的服务，集群的DNS域名服务没有问题，IPtable的防火墙配置没有问题，kube-proxy服务也都运转正常。

请参看本文档故障排除部分的其他小节，以获取更加全面的故障处理信息。









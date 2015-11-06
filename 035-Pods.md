# Pods


在Kubernetes中，与用采用单独的应用容器方式不同，pod是最小的部署单元，可以对其进行创建，调度和管理操作。



## _pod_是什么?

A _pod_ (as in a pod of whales or pea pod) corresponds to a colocated group of applications running with a shared context. Within that context, the applications may also have individual cgroup isolations applied. A pod models an application-specific "logical host" in a containerized environment. It may contain one or more applications which are relatively tightly coupled &mdash; in a pre-container world, they would have executed on the same physical or virtual host.

一个pod（正如鲸群中的群，或者豌豆荚中的荚）对应着处在一个共享情景的一组共存一组应用。在该上下文中，应用可能使用了单独的cgroup隔离。一个pod将处在一个容器化环境的以应用为中心的“逻辑主机“。它可能包含一个或者多个应用，它们之间很可能紧密耦合 - 在一容器之前的世界里，他们本应该在同样的物理或者虚拟主机上。

The context of the pod can be defined as the conjunction of several Linux namespaces:

* PID namespace (applications within the pod can see each other's processes)
* network namespace (applications within the pod have access to the same IP and port space)
* IPC namespace (applications within the pod can use SystemV IPC or POSIX message queues to communicate)
* UTS namespace (applications within the pod share a hostname)

pod的上下文可以被定义成Linux几种命名空间的的结合：

- PID命名空间（处在同一个pod中的应用可以看到彼此的进程）
- 网络命名空间（处在同一个pod中的应用能访问一样的IP和端口空间）
- IPC命名空间（处在同一个pod中的应用可以用SystemV IPC或者POSIX消息队列来进行通讯）
- UTS命名空间（处在同一个pod中的应用公用一个主机名）

Applications within a pod also have access to shared volumes, which are defined at the pod level and made available in each application's filesystem. Additionally, a pod may define top-level cgroup isolations which form an outer bound to any individual isolation applied to constituent applications.

In terms of [Docker](https://www.docker.com/) constructs, a pod consists of a colocated group of Docker containers with shared [volumes](volumes.md). PID namespace sharing is not yet implemented with Docker.

Like individual application containers, pods are considered to be relatively ephemeral rather than durable entities. As discussed in [life of a pod](pod-states.md), pods are scheduled to nodes and remain there until termination (according to restart policy) or deletion. When a node dies, the pods scheduled to that node are deleted. Specific pods are never rescheduled to new nodes; instead, they must be replaced (see [replication controller](replication-controller.md) for more details). (In the future, a higher-level API may support pod migration.)

处在同一个pod中的应用也能访问一样的的共享volume，卷是在pod的层级上定义的，并且在在每个应用的文件系统中被设为可用。另外，一个pod也可以定义顶级的cgroup隔离，这给任何一个应用到应用到单独的隔离构成了一个外围的约束。

至于Docker的构成，一个pod包含一组共存一组Docker容器，他们有共享的卷。PID的命名空间对于Docker来说还没有实现。

就如单独的应用容器一样，pod被认为是比较临时性的而不是持久性的实体。就如在一个pod的生命周期中讨论到的，pod被调度到node然后直到被结束或者删除前一直被保留在那里。当一个node挂掉时，被调度到该node的pod会被删除。特定的pod从来不会被调度到新的节点；他们必须被替换。（在以后，一个更高层次的API可能会支持pod的迁移。）


## Motivation for pods

### Resource sharing and communication

Pods facilitate data sharing and communication among their constituents.

The applications in  pod all use the same network namespace/IP and port space, and can find and communicate with each other using localhost. Each pod has an IP address in a flat shared networking namespace that has full communication with other physical computers and containers across the network. The hostname is set to the pod's Name for the application containers within the pod. [More details on networking](../admin/networking.md).

In addition to defining the application containers that run in the pod, the pod specifies a set of shared storage volumes. Volumes enable data to survive container restarts and to be shared among the applications within the pod.

### Management

Pods also simplify application deployment and management by providing a higher-level abstraction than the raw, low-level container interface. Pods serve as units of deployment and horizontal scaling/replication. Co-location (co-scheduling), fate sharing, coordinated replication, resource sharing, and dependency management are handled automatically.

## Pod的设计动机

### 资源共享与通讯

Pod有助于数据的在组成成员之间的共享和通讯。

一个pod中应用使用相同的网络命名空间/IP和端口空间，可以使用localhost找到彼此并进行通信。每一个pod在扁平的共享网络命名空间中有一个IP地址，能与网络上其他的物理计算机和容器进行完整的通讯。pod中的应用容器的主机名设置成pod的名称。

除了定义运行在pod中应用容器之外，pod指定了一组共享的存储卷。卷能让数据不受容器重启的影响，并且可以在pod的容器之间共享。



### 管理

通过提供更高层次的抽象，而不是提供一个原始，低层次的容器接口，pod也简化了应用的部署与管理。pod作为部署的单元，也是水平伸缩复制的单元。同地点安放，命运共享，协作的副本复制，资源贡献，和依赖管理都能自动的处理。


## Uses of pods

Pods can be used to host vertically integrated application stacks, but their primary motivation is to support co-located, co-managed helper programs, such as:

* content management systems, file and data loaders, local cache managers, etc.
* log and checkpoint backup, compression, rotation, snapshotting, etc.
* data change watchers, log tailers, logging and monitoring adapters, event publishers, etc.
* proxies, bridges, and adapters
* controllers, managers, configurators, and updaters

Individual pods are not intended to run multiple instances of the same application, in general.

For a longer explanation, see [The Distributed System ToolKit: Patterns for Composite Containers](http://blog.kubernetes.io/2015/06/the-distributed-system-toolkit-patterns.html).

## Alternatives considered

_Why not just run multiple programs in a single (Docker) container?_

1. Transparency. Making the containers within the pod visible to the infrastructure enables the infrastructure to provide services to those containers, such as process management and resource monitoring. This facilitates a number of conveniences for users.
2. Decoupling software dependencies. The individual containers may be rebuilt and redeployed independently. Kubernetes may even support live updates of individual containers someday.
3. Ease of use. Users don't need to run their own process managers, worry about signal and exit-code propagation, etc.
4. Efficiency. Because the infrastructure takes on more responsibility, containers can be lighter weight.

_Why not support affinity-based co-scheduling of containers?_

That approach would provide co-location, but would not provide most of the benefits of pods, such as resource sharing, IPC, guaranteed fate sharing, and simplified management.

## Durability of pods (or lack thereof)

Pods aren't intended to be treated as durable [pets](https://blog.engineyard.com/2014/pets-vs-cattle). They won't survive scheduling failures, node failures, or other evictions, such as due to lack of resources, or in the case of node maintenance.

In general, users shouldn't need to create pods directly. They should almost always use controllers (e.g., [replication controller](replication-controller.md)), even for singletons.  Controllers provide self-healing with a cluster scope, as well as replication and rollout management.

The use of collective APIs as the primary user-facing primitive is relatively common among cluster scheduling systems, including [Borg](https://research.google.com/pubs/pub43438.html), [Marathon](https://mesosphere.github.io/marathon/docs/rest-api.html), [Aurora](http://aurora.apache.org/documentation/latest/configuration-reference/#job-schema), and [Tupperware](http://www.slideshare.net/Docker/aravindnarayanan-facebook140613153626phpapp02-37588997).

Pod is exposed as a primitive in order to facilitate:

* scheduler and controller pluggability
* support for pod-level operations without the need to "proxy" them via controller APIs
* decoupling of pod lifetime from controller lifetime, such as for bootstrapping
* decoupling of controllers and services &mdash; the endpoint controller just watches pods
* clean composition of Kubelet-level functionality with cluster-level functionality &mdash; Kubelet is effectively the "pod controller"
* high-availability applications, which will expect pods to be replaced in advance of their termination and certainly in advance of deletion, such as in the case of planned evictions, image prefetching, or live pod migration [#3949](http://issue.k8s.io/3949)

The current best practice for pets is to create a replication controller with `replicas` equal to `1` and a corresponding service. If you find this cumbersome, please comment on [issue #260](http://issue.k8s.io/260).

## Termination of Pods

Because pods represent running processes on nodes in the cluster, it is important to allow those processes to gracefully terminate when they are no longer needed (vs being violently killed with a KILL signal and having no chance to clean up). Users should be able to request deletion and know when processes terminate, but also be able to ensure that deletes eventually complete. When a user requests deletion of a pod the system records the intended grace period before the pod is allowed to be forcefully killed, and a TERM signal is sent to the main process in each container. Once the grace period has expired the KILL signal is sent to those processes and the pod is then deleted from the API server. If the Kubelet or the container manager is restarted while waiting for processes to terminate, the termination will be retried with the full grace period.

An example flow:

1. User sends command to delete Pod, with default grace period (30s)
2. The Pod in the API server is updated with the time beyond which the Pod is considered "dead" along with the grace period.
3. Pod shows up as "Terminating" when listed in client commands
4. (simultaneous with 3) When the Kubelet sees that a Pod has been marked as terminating because the time in 2 has been set, it begins the pod shutdown process.
  1. If the pod has defined a [preStop hook](container-environment.md#hook-details), it is invoked inside of the pod. If the `preStop` hook is still running after the grace period expires, step 2 is then invoked with a small (2 second) extended grace period.
  2. The processes in the Pod are sent the TERM signal.
5. (simultaneous with 3), Pod is removed from endpoints list for service, and are no longer considered part of the set of running pods for replication controllers. Pods that shutdown slowly can continue to serve traffic as load balancers (like the service proxy) remove them from their rotations.
6. When the grace period expires, any processes still running in the Pod are killed with SIGKILL.
7. The Kubelet will finish deleting the Pod on the API server by setting grace period 0 (immediate deletion). The Pod disappears from the API and is no longer visible from the client.

By default, all deletes are graceful within 30 seconds. The `kubectl delete` command supports the `--grace-period=<seconds>` option which allows a user to override the default and specify their own value. The value `0` indicates that delete should be immediate, and removes the pod in the API immediately so a new pod can be created with the same name. On the node pods that are set to terminate immediately will still be given a small grace period before being force killed.

## Privileged mode for pod containers

From kubernetes v1.1, any container in a pod can enable privileged mode, using the `privileged` flag on the `SecurityContext` of the container spec. This is useful for containers that want to use linux capabilities like manipulating the network stack and accessing devices. Processes within the container get almost the same privileges that are available to processes outside a container. With privileged mode, it should be easier to write network and volume plugins as seperate pods that don't need to be compiled into the kubelet.

If the master is running kubernetes v1.1 or higher, and the nodes are running a version lower than v1.1, then new privileged pods will be accepted by api-server, but will not be launched. They will be pending state.
If user calls `kubectl describe pod FooPodName`, user can see the reason why the pod is in pending state. The events table in the describe command output will say:
`Error validating pod "FooPodName"."FooPodNamespace" from api, ignoring: spec.containers[0].securityContext.privileged: forbidden '<*>(0xc2089d3248)true'`


If the master is running a version lower than v1.1, then privileged pods cannot be created. If user attempts to create a pod, that has a privileged container, the user will get the following error:
`The Pod "FooPodName" is invalid.
spec.containers[0].securityContext.privileged: forbidden '<*>(0xc20b222db0)true'`

## API Object

Pod is a top-level resource in the kubernetes REST API. More details about the
API object can be found at: [Pod API
object](https://htmlpreview.github.io/?https://github.com/kubernetes/kubernetes/HEAD/docs/api-reference/v1/definitions.html#_v1_pod).

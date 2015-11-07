# 网络

Kubernetes网络需要解决下面四类不同的问题：

1. 高耦合容器和容器间通信
2. Pod和Pod间通信
3. Pod和Service间通信
4. 外部和内部通信


## 模型和动机
Kubernetes从默认的Docker网络模型中脱离出来（尽管和Docker 1.8的网络插件已经很接近了）。在一个共享网络命名空间的平面内，目标是一个pod一个IP，每个pod都可以和其它物理机或者容器实现跨网络通信。每个pod一个IP创建了一个简洁的，后向兼容的模型，在这个模型里，无论从哪个角度来看，包括端口分配，组网，命名，服务发现，负载均衡，应用配置和迁移，pod都可以被当作虚拟机或者物理主机。


Dynamic port allocation, on the other hand, requires supporting both static ports (e.g., for externally accessible services) and dynamically allocated ports, requires partitioning centrally allocated and locally acquired dynamic ports, complicates scheduling (since ports are a scarce resource), is inconvenient for users, complicates application configuration, is plagued by port conflicts and reuse and exhaustion, requires non-standard approaches to naming (e.g. consul or etcd rather than DNS), requires proxies and/or redirection for programs using standard naming/addressing mechanisms (e.g. web browsers), requires watching and cache invalidation for address/port changes for instances in addition to watching group membership changes, and obstructs container/pod migration (e.g. using CRIU). NAT introduces additional complexity by fragmenting the addressing space, which breaks self-registration mechanisms, among other problems.

另一方面，动态端口分配，要求同时支持静态端口（如对外部可接入的服务）和动态分配的端口，需要对中心型分配和本地获取动态端口做区分，复杂的调度（应为端口是稀缺资源）






























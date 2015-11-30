# 网络

Kubernetes网络需要解决下面四类不同的问题：

1. 高耦合容器和容器间通信
2. Pod和Pod间通信
3. Pod和Service间通信
4. 外部和内部通信

## 模型和动机
Kubernetes从默认的Docker网络模型中脱离出来（尽管和Docker 1.8的网络插件已经很接近了）。在一个共享网络命名空间的平面内，目标是一个pod一个IP，每个pod都可以和其它物理机或者容器实现跨网络通信。每个pod一个IP创建了一个简洁的，后向兼容的模型，在这个模型里，无论从哪个角度来看，包括端口分配，组网，命名，服务发现，负载均衡，应用配置和迁移，pod都可以被当作虚拟机或者物理主机。

另一方面，动态端口分配，要求同时支持静态端口（如对外部可接入的服务）和动态分配的端口；需要对中心型分配和本地获取动态端口做区分，复杂的调度（应为端口是稀缺资源）对用户来说是非常不方便的；复杂的应用配置，造成用户会被端口冲突、重用和耗尽所困扰；要求非标准的方式来命名（如consul和etcd而不是DNS）；需要代理或者重定向才能让程序使用标准的命名和寻址技术（如web浏览器）；如果还希望监控整个组成员的变化，并且阻碍容器或pod迁移（使用CRIU），就需要监控和缓存无效的地址和端口变化。其它问题中，NAT分隔地址空间引入了额外的复杂度，打破了自注册机制。

## 容器到容器

所有容器在一个pod中表现为它们在同一个主机上并且不受网络限制。它们可以通过端口在本地实现互相通信。这提供了简单（已知静态端口），安全（端口绑定在localhost，可以被pod中其他容器发现但不会被外部看到），还有性能提升。这同样减少了物理机获虚拟机上非容器化应用的迁移。人们运行应用都堆砌到统一个主机上，它已经解决了如何让端口不冲突和已经安排了如何让客户端去找到它。

这个方法确实减少了一个pod中容器的隔离性（端口可能冲突），并且可以没有容器私有化端口，但这些看起来都是未来才会面对的相对比较小的问题。另外，本地化pod是容器在pod中共享同样的资源（如volumes，cpu，内存等），所以希望并且可以容忍隔离性的减少。通常情况，用户可以控制哪些容器属于同一个pod，而不能控制哪些pod一起运行在一个主机上。

## Pod到Pod

因为每个pod都有一个“真”（非机器私有的）的IP地址，pods间可以相互通信，不依赖于代理和转换。Pod可以使用一个常见的端口号，还可以防止更高层次的服务发现系统像DNS-SD，Consul或者Etcd。

当任何容器调用ioctl（SIOCGIFADDR）（获取接口的地址），可以看到相同的IP，任何同级别的容器都可以看到这个IP，就是说每一个pod有自己的IP地址，并且这个IP地址其他pods也可以知道。在容器内外通过生成相同IP地址和端口，我们创造了一个非NAT模式，扁平化的地址空间。运行```ip addr show```应该可以看到期待的值。这会让所有现在的命名或发现机制到盒子外面去实现，包括自注册机制和应用分发IP地址。我们应该对pod间网络通信持乐观态度。在一个pod内，容器间更像是通过volumes（如tmpfs）或者IPC来通信。

这种方式和标准的Docker模式有所不同。在Docker模式下，每一个容器有一个IP，IP是在172.x.x.x的网段内，并且只能从SIOCGIFADDR看到172.x.x.x的地址。如果这些容器连接了另一个容器，那同级别容器会看到这个连接来自于一个不同的IP而不是容器自己知道的IP。简而言之，永远不能自注册同一个容器内的任何东西，因为一个容器不能被自己私有的IP获取到。

另一个解决方案我们考虑增加一个寻址层：每个容器一个中心化pod的IP。每个容器有自己本地的IP地址，这个IP只在pod内可见。这种方案可能会让在物理机或虚拟机上的容器化应用移动到pod中更容易，但会使实现变得更复杂（如需要每个pod一个桥，水平分割的DNS）。造成额外的地址转换，而且会破话自注册和IP分配机制。

像Docker，端口仍可以向主机节点的接口公开，但这样的需求应该从根本上减少。

## 实现

For the Google Compute Engine cluster configuration scripts, we use advanced routing rules and ip-forwarding-enabled VMs so that each VM has an extra 256 IP addresses that get routed to it. This is in addition to the 'main' IP address assigned to the VM that is NAT-ed for Internet access. The container bridge (called cbr0 to differentiate it from docker0) is set up outside of Docker proper.

例如，GCE的高级路由规则：
```
gcloud compute routes add "${MINION_NAMES[$i]}" \
  --project "${PROJECT}" \
  --destination-range "${MINION_IP_RANGES[$i]}" \
  --network "${NETWORK}" \
  --next-hop-instance "${MINION_NAMES[$i]}" \
  --next-hop-instance-zone "${ZONE}" &
```


GCE itself does not know anything about these IPs, though. This means that when a pod tries to egress beyond GCE's project the packets must be SNAT'ed (masqueraded) to the VM's IP, which GCE recognizes and allows.


### 其他实现

With the primary aim of providing IP-per-pod-model, other implementations exist to serve the purpose outside of GCE.

OpenVSwitch with GRE/VxLAN
Flannel
L2 networks ("With Linux Bridge devices" section)
Weave is yet another way to build an overlay network, primarily aiming at Docker integration.
Calico uses BGP to enable real container IPs.












































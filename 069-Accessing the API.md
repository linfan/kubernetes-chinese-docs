# **API Server端口配置**

本文档介绍Kubernetes apiserver服务的端口以及如何访问这些端口。观众都是集群管理员，都想定制他们自己的集群或者了解细节。

有关集群访问的更多问题，在[Accessing the cluster](http://kubernetes.io/v1.1/docs/user-guide/accessing-the-cluster.html)都有覆盖。

## **服务的Ports和IPs**
Kubernets apiserver进程提供Kuvernetes API。通常情况下，有一个进程运行在单一kubernetes-master节点上。

默认情况，Kubernetes API Server提供HTTP的两个端口：

 1.本地主机端口
 - HTTP服务
 - 默认端口8080，修改标识--insecure-port
 - 默认IP是本地主机，修改标识—insecure-bind-address
 - 在HTTP中没有认证和授权检查
 - 主机访问受保护
 
2.安全端口
 - 默认端口6443，修改标识—secure-port
 - 默认IP是首个非本地主机的网络接口，修改标识—bind-address
 - HTTPS服务。设置证书和秘钥的标识，--tls-cert-file，--tls-private-key-file
 - [认证方式](http://kubernetes.io/v1.1/docs/admin/authentication.html)，令牌文件或者客户端证书
 - 使用基于策略的[授权方式](http://kubernetes.io/v1.1/docs/admin/authorization.html)

3.移除：只读端口
 - 基于安全考虑，会移除只读端口，使用[Service Account](http://kubernetes.io/v1.1/docs/user-guide/service-accounts.html)代替。

## **代理和防火墙规则**
此外，在某些配置文件中有一个代理（nginx）作为API Server进程运行在同一台机器上。该代理是HTTPS服务，认证端口是443，访问API Server是本地主机8080端口。在这些配置文件里，Secure Port通常设置为6443。

防火墙规则，通常配置运行外部HTTPS通过443端口访问。

上面的都是默认配置，反应了Kubernetes使用kube-up.sh如何部署到Google Compute Engine。

## **用例和IP:Ports**
有关服务端口，有三种不同的配置，有各自的应用场景。
1. Kubernetes集群之外的客户端，例如在台式机上运行kubectl命令的人员。目前，通过运行在kubernetes-master机器上面的代理（nginx）访问本地主机端口。该代理可以使用证书认证或者Token认证方式。
2. 运行在Kuvernetes的Container里面的进程需要从API Server中读取。目前，这些进程都是用[Service Account](http://kubernetes.io/v1.1/docs/user-guide/service-accounts.html)
3. 调度器和Controller管理进程，需要对API做读写操作。目前，这些都必须运行在API Server同样的主机上面，使用本地主机。未来，这些进程将会使用Service Account服务，避免共存的必要。
4. Kubelets，需要对API做读写操作，并且同API Server相比，它必须运行在不同的机器上面。Kubelet使用安全端口获取Pod，发现Pod可以看到的服务，并且记录这些事件。在集群启动事件内，分布设置Kubelet凭证。Kubelet和Kube-proxy可以使用证书认证和Token认证方式。

## **预期变化**
- Policy会限制Kubelet通过身份认证端口实行的一些操作。
- 调度器和Controller管理也会使用Secure Port。他们可以运行在不同的机器上。
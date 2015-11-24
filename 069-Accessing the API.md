# **API Server端口配置**

本文档介绍Kubernetes apiserver服务的端口以及如何访问这些端口。观众都是集群管理员，都想定制他们自己的集群或者了解细节。

有关集群访问的更多问题，在[Accessing the cluster](http://kubernetes.io/v1.1/docs/user-guide/accessing-the-cluster.html)都有覆盖。

## **服务的Ports和IPs**
Kubernets apiserver进程提供Kuvernetes API。通常情况下，有一个进程运行在单一kubernetes-master节点上。

默认情况，Kubernetes APIserver提供HTTP上的两个端口：

 1.本地host端口（Localhost Port）
 - HTTP服务
 - 默认端口8080，修改标--insecure-port
 - 默认IP，localhost，修改标识—insecure-bind-address
 - 在HTTP中没有认证和授权检查
 - 主机访问受保护
 
2.安全端口（Secure Port）
 - 默认端口6443，修改标识—secure-port
 - 默认IP是首个非localhost网络接口，修改标识—bind-address
 - HTTPS服务。设置证书和秘钥的标识，--tls-cert-file，--tls-private-key-file
 - 认证方式，令牌文件或者客户端证书
 - 使用基于策略的授权方式

3.移除：只读端口（Removed：ReadOnly Port）
 - 基于安全考虑，会移除只读端口，使用服务账户（service account）代替。

## **代理和防火墙规则**

# **Kube-apiserver**

## **概要**

Kubernetes API服务器为API对象验证和配置数据，这些对象包含Pod，Service，Replication Controller等等。API Server提供REST操作以及前端到集群的共享状态，所有其它组件可以通过这些共享状态交互。
```
kube-apiserver
```

## **选项**
```
  --admission-control="AlwaysAdmit"：集群中资源的Admission Controller的插件的有序列表，分别使用逗号分隔，AlwaysAdmit, AlwaysDeny, DenyEscalatingExec, DenyExecOnPrivileged, InitialResources, LimitRanger, NamespaceAutoProvision, NamespaceExists, NamespaceLifecycle, ResourceQuota, SecurityContextDeny, ServiceAccount。
  --admission-control-config-file="": Admission Controller配置文件。
  --advertise-address=<nil>:广播API Server给所有集群成员的IP地址。
  --allow-privileged[=false]:
  --authorization-mode="AlwaysAllow":
  --authorization-policy-file="":
  --basic-auth-file="":
  --bind-address=0.0.0.0:
  --cert-dir="/var/run/kubernetes":
  --client-ca-file="":
  --cloud-config="":
  --cloud-provider="": 
  --cluster-name="kubernetes":
  --cors-allowed-origins=[]:
  --etcd-config="":
  --etcd-prefix="/registry": 
  --etcd-servers=[]:
  --etcd-servers-overrides=[]:
  --event-ttl=1h0m0s: 保留事件的时间值，默认1小时。
  --experimental-keystone-url="": 如果Passed，激活Keystone认证插件。
  --external-hostname="": 为Master生成外部URLs使用的主机名。
  --google-json-key="": 用户Google Cloud Platform Service Account JSON Key认证。
  --insecure-bind-address=127.0.0.1：非安全端口（所有接口都设置为0.0.0.0）的服务IP地址。默认是本地地址。
  --insecure-port=8080: 不安全且没有认证的进程访问端口，默认8080。假设防火墙规则设置该端口从集群外部禁止访问，并且在集群的公共地址区，443端口是该端口的代理。这是nginx的默认配置。
  --kubelet-certificate-authority="": 证书路径。证书授权文件。
  --kubelet-client-certificate="": TLS客户端证书文件路径。
  --kubelet-client-key="": TLS客户端秘钥文件路径。
  --kubelet-https[=true]: 使用https建立Kubelet连接。
  --kubelet-port=10250: Kubelet端口
  --kubelet-timeout=5s: Kubelet操作Timeout
  --log-flush-frequency=5s: 日志缓冲秒数的最大值
  --long-running-request-regexp="(/|^)((watch|proxy)(/|$)|(logs?|portforward|exec|attach)/?$)": 匹配长时间运行请求的正则表达式，该请求不属于最大机请求处理。
  --master-service-namespace="default": Namespace，该Namespace的Kubernetes主服务应该注入Pod。
  --max-connection-bytes-per-sec=0: 如果非零，表示每个用户连接的最大值，字节数/秒，当前只适用于长时间运行的请求。
  --max-requests-inflight=400: 给定时间内运行的请求的最大值。如果超过最大值，该请求就会被拒绝。零表示没有限制。
  --min-request-timeout=1800: 这是个可选字段，表示一个请求处理的最短时间，单位是秒。在超时之前，这个请求必须是激活的。目前，请求处理程序会选择一个高于该数值的随机值作为连接超时的值进行分散负载。（An optional field indicating the minimum number of seconds a handler must keep a request open before timing it out. Currently only honored by the watch request handler, which picks a randomized value above this number as the connection timeout, to spread out load.-----翻译有待考虑）
  --oidc-ca-file="": 如果设置该选项，Oidc-ca-file中的相关机构会验证OpenID服务的证书。否则，会使用主机的根证书。
  --oidc-client-id="": 如果设置了oidc-issuer-url字段，该字段，OpenID连接客户端的客户ID也必须设置。
  --oidc-issuer-url="": OpenID发行的URL，只接受HTTPS协议。如果设置该字段，将被用来验证OIDC JSON Web Token（JWT）。
  --oidc-username-claim="sub": 。默认值之外的那些值，可能是不唯一的，可变的。这个标志还在尝试中，详情请参考验证部分的文档。
  --profiling[=true]: 通过web接口进行分析 host:port/debug/pprof/
  --runtime-config=: key=value键值对集，描述运行时配置，也会回传输到apiserver。apis/<groupVersion>键值用于打开或者关闭指定的api版本。apis/<groupVersion>/<resource>用于打开、关闭指定的资源。api/all和api/legacy是特殊的值，分别控制所有和遗留的api版本。
  --secure-port=6443: 用于HTTPS的认证和授权。0表示不支持HTTPS服务。
  --service-account-key-file="": 该文件包含RPM-encoded x509 RSA的私钥和公钥，用于验证ServiceAccount的Token。如果没有指定，会使用该配置--tls-private-key-file。
  --service-account-lookup[=false]: If true, validate ServiceAccount tokens exist in etcd as part of authentication.
  --service-cluster-ip-range=<nil>: A CIDR notation IP range from which to assign service cluster IPs. This must not overlap with any IP ranges assigned to nodes for pods.
  --service-node-port-range=: NodePort可见性服务的端口范围，包含范围的两端。如'30000-32767'，包含30000和32767端口。
  --ssh-keyfile="": 如果非空，使用安全SSH代理到该节点，用该秘钥文件。
  --ssh-user="": 如果非空，使用安全SSH代理到该节点，用该用户名。
  --storage-versions="extensions/v1beta1,v1": 存储资源的版本。不同的组存储在不同的版本里面，指定格式"group1/version1,group2/version2..."。该标志预计出注册在服务器中的所有组的存储版本的完整列表。它默认是所有注册组的首选版本列表，来自KUBE_API_VERSIONS变量。
  --tls-cert-file="": 该文件包含HTTPS的x509证书。(CA cert, if any, concatenated after server cert)。如果支持HTTPS，且没有配置--tls-cert-file和 --tls-private-key-file，会给公共地址生成一个自签的证书和对应的秘钥，保存在/var/run/kubernetes。
  --tls-private-key-file="": 该文件包含x509私钥匹配项--tls-cert-file.
  --token-auth-file="": 该文件使用Token验证保护API Server的安全端口。
  --watch-cache[=true]: 可以在apiserver看缓存。
```
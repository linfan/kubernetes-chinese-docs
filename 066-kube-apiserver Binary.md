# **Kube-apiserver**

## **概要**

Kubernetes API服务器为API对象验证和配置数据，这些对象包含Pod，Service，Replication Controller等等。API Server提供REST操作以及前端到集群的共享状态，所有其它组件可以通过这些共享状态交互。
```
kube-apiserver
```

## **选项**
```
  --admission-control="AlwaysAdmit"：
  --admission-control-config-file="":
  --advertise-address=<nil>:
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
      --kubelet-certificate-authority="": Path to a cert. file for the certificate authority.
      --kubelet-client-certificate="": Path to a client cert file for TLS.
      --kubelet-client-key="": Path to a client key file for TLS.
      --kubelet-https[=true]: Use https for kubelet connections
      --kubelet-port=10250: Kubelet port
      --kubelet-timeout=5s: Timeout for kubelet operations
      --log-flush-frequency=5s: Maximum number of seconds between log flushes
      --long-running-request-regexp="(/|^)((watch|proxy)(/|$)|(logs?|portforward|exec|attach)/?$)": A regular expression matching long running requests which should be excluded from maximum inflight request handling.
      --master-service-namespace="default": The namespace from which the kubernetes master services should be injected into pods
      --max-connection-bytes-per-sec=0: If non-zero, throttle each user connection to this number of bytes/sec.  Currently only applies to long-running requests
      --max-requests-inflight=400: The maximum number of requests in flight at a given time.  When the server exceeds this, it rejects requests.  Zero for no limit.
      --min-request-timeout=1800: An optional field indicating the minimum number of seconds a handler must keep a request open before timing it out. Currently only honored by the watch request handler, which picks a randomized value above this number as the connection timeout, to spread out load.
      --oidc-ca-file="": If set, the OpenID server's certificate will be verified by one of the authorities in the oidc-ca-file, otherwise the host's root CA set will be used
      --oidc-client-id="": The client ID for the OpenID Connect client, must be set if oidc-issuer-url is set
      --oidc-issuer-url="": The URL of the OpenID issuer, only HTTPS scheme will be accepted. If set, it will be used to verify the OIDC JSON Web Token (JWT)
      --oidc-username-claim="sub": The OpenID claim to use as the user name. Note that claims other than the default ('sub') is not guaranteed to be unique and immutable. This flag is experimental, please see the authentication documentation for further details.
      --profiling[=true]: Enable profiling via web interface host:port/debug/pprof/
      --runtime-config=: A set of key=value pairs that describe runtime configuration that may be passed to apiserver. apis/<groupVersion> key can be used to turn on/off specific api versions. apis/<groupVersion>/<resource> can be used to turn on/off specific resources. api/all and api/legacy are special keys to control all and legacy api versions respectively.
      --secure-port=6443: The port on which to serve HTTPS with authentication and authorization. If 0, don't serve HTTPS at all.
      --service-account-key-file="": File containing PEM-encoded x509 RSA private or public key, used to verify ServiceAccount tokens. If unspecified, --tls-private-key-file is used.
      --service-account-lookup[=false]: If true, validate ServiceAccount tokens exist in etcd as part of authentication.
      --service-cluster-ip-range=<nil>: A CIDR notation IP range from which to assign service cluster IPs. This must not overlap with any IP ranges assigned to nodes for pods.
      --service-node-port-range=: A port range to reserve for services with NodePort visibility.  Example: '30000-32767'.  Inclusive at both ends of the range.
      --ssh-keyfile="": If non-empty, use secure SSH proxy to the nodes, using this user keyfile
      --ssh-user="": If non-empty, use secure SSH proxy to the nodes, using this user name
      --storage-versions="extensions/v1beta1,v1": The versions to store resources with. Different groups may be stored in different versions. Specified in the format "group1/version1,group2/version2...". This flag expects a complete list of storage versions of ALL groups registered in the server. It defaults to a list of preferred versions of all registered groups, which is derived from the KUBE_API_VERSIONS environment variable.
      --tls-cert-file="": File containing x509 Certificate for HTTPS.  (CA cert, if any, concatenated after server cert). If HTTPS serving is enabled, and --tls-cert-file and --tls-private-key-file are not provided, a self-signed certificate and key are generated for the public address and saved to /var/run/kubernetes.
      --tls-private-key-file="": File containing x509 private key matching --tls-cert-file.
      --token-auth-file="": If set, the file that will be used to secure the secure port of the API server via token authentication.
      --watch-cache[=true]: Enable watch caching in the apiserver
```
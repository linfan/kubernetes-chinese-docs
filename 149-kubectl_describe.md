## kubectl describe
`译者：hurf` `校对：无`


输出指定的一个/多个资源的详细信息。

### 摘要

输出指定的一个/多个资源的详细信息。

此命令组合调用多条API，输出指定的一个或者一组资源的详细描述。

$ kubectl describe TYPE NAME_PREFIX

首先检查是否有精确匹配TYPE和NAME_PREFIX的资源，如果没有，将会输出所有名称以NAME_PREFIX开头的资源详细信息。

支持的资源包括但不限于（大小写不限）：pods (po)、services (svc)、
replicationcontrollers (rc)、nodes (no)、events (ev)、componentstatuses (cs)、
limitranges (limits)、persistentvolumes (pv)、persistentvolumeclaims (pvc)、
resourcequotas (quota)和secrets。

```
kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | -l label] | TYPE/NAME)
```

### 示例

```
# 描述一个node
$ kubectl describe nodes kubernetes-minion-emt8.c.myproject.internal

# 描述一个pod
$ kubectl describe pods/nginx

# 描述pod.json中的资源类型和名称指定的pod
$ kubectl describe -f pod.json

# 描述所有的pod
$ kubectl describe pods

# 描述所有包含label name=myLabel的pod
$ kubectl describe po -l name=myLabel

# 描述所有被replication controller “frontend”管理的pod（rc创建的pod都以rc的名字作为前缀）
$ kubectl describe pods frontend
```

### 选项

```
  -f, --filename=[]: 用来指定待描述资源的文件名，目录名或者URL。
  -l, --selector="": 用于过滤资源的Label。
```

### 继承自父命令的选项
```
      --alsologtostderr[=false]: 同时输出日志到标准错误控制台和文件。
      --api-version="": 和服务端交互使用的API版本。
      --certificate-authority="": 用以进行认证授权的.cert文件路径。
      --client-certificate="": TLS使用的客户端证书路径。
      --client-key="": TLS使用的客户端密钥路径。
      --cluster="": 指定使用的kubeconfig配置文件中的集群名。
      --context="": 指定使用的kubeconfig配置文件中的环境名。
      --insecure-skip-tls-verify[=false]: 如果为true，将不会检查服务器凭证的有效性，这会导致你的HTTPS链接变得不安全。
      --kubeconfig="": 命令行请求使用的配置文件路径。
      --log-backtrace-at=:0: 当日志长度超过定义的行数时，忽略堆栈信息。
      --log-dir="": 如果不为空，将日志文件写入此目录。
      --log-flush-frequency=5s: 刷新日志的最大时间间隔。
      --logtostderr[=true]: 输出日志到标准错误控制台，不输出到文件。
      --match-server-version[=false]: 要求服务端和客户端版本匹配。
      --namespace="": 如果不为空，命令将使用此namespace。
      --password="": API Server进行简单认证使用的密码。
  -s, --server="": Kubernetes API Server的地址和端口号。
      --stderrthreshold=2: 高于此级别的日志将被输出到错误控制台。
      --token="": 认证到API Server使用的令牌。
      --user="": 指定使用的kubeconfig配置文件中的用户名。
      --username="": API Server进行简单认证使用的用户名。
      --v=0: 指定输出日志的级别。
      --vmodule=: 指定输出日志的模块，格式如下：pattern=N，使用逗号分隔。
```

### 参见

* [kubectl](kubectl.md)	 - 使用kubectl来管理Kubernetes集群。
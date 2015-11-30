## kubectl delete
`译者：hurf` `校对：无`


通过文件名、控制台输入、资源名或者label selector删除资源。

### 摘要

通过文件名、控制台输入、资源名或者label selector删除资源。
接受JSON和YAML格式的描述文件。

只能指定以下参数类型中的一种：文件名、资源类型和名称、资源类型和label selector。
注意：delete命令不检查资源版本，如果有人在你进行删除操作的同时进行更新操作，他所做的更新将随资源同时被删除。

```
kubectl delete ([-f FILENAME] | TYPE [(NAME | -l label | --all)])
```

### 示例

```
# 通过pod.json文件中指定的资源类型和名称删除一个pod
$ kubectl delete -f ./pod.json

# 通过控制台输入的JSON所指定的资源类型和名称删除一个pod
$ cat pod.json | kubectl delete -f -

# 删除所有名为“baz”和“foo”的pod和service
$ kubectl delete pod,service baz foo

# 删除所有带有lable name=myLabel的pod和service
$ kubectl delete pods,services -l name=myLabel

# 删除UID为1234-56-7890-234234-456456的pod
$ kubectl delete pod 1234-56-7890-234234-456456

# 删除所有的pod
$ kubectl delete pods --all
```

### 选项

```
      --all[=false]: 使用[-all]选择所有指定的资源。
      --cascade[=true]: 如果为true，级联删除指定资源所管理的其他资源（例如：被replication controller管理的所有pod）。默认为true。
  -f, --filename=[]: 用以指定待删除资源的文件名，目录名或者URL。
      --grace-period=-1: 安全删除资源前等待的秒数。如果为负值则忽略该选项。
      --ignore-not-found[=false]: 当待删除资源未找到时，也认为删除成功。如果设置了--all选项，则默认为true。
  -o, --output="": 输出格式，使用“-o name”来输出简短格式（资源类型/资源名）。
  -l, --selector="": 用于过滤资源的Label。
      --timeout=0: 删除资源的超时设置，0表示根据待删除资源的大小由系统决定。
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
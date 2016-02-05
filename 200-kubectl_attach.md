## kubectl attach
`译者：hurf` `校对：无`


连接到一个正在运行的容器。

### 摘要

连接到现有容器中一个正在运行的进程。

```
kubectl attach POD -c CONTAINER
```

### 示例

```
# 获取正在运行中的pod 123456-7890的输出，默认连接到第一个容器
$ kubectl attach 123456-7890

# 获取pod 123456-7890中ruby-container的输出
$ kubectl attach 123456-7890 -c ruby-container date

# 切换到终端模式，将控制台输入发送到pod 123456-7890的ruby-container的“bash”命令，并将其输出到控制台/
# 错误控制台的信息发送回客户端。
$ kubectl attach 123456-7890 -c ruby-container -i -t
```

### 选项

```
  -c, --container="": 容器名。
  -i, --stdin[=false]: 将控制台输入发送到容器。
  -t, --tty[=false]: 将标准输入控制台作为容器的控制台输入。
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
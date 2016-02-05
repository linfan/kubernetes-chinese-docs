## kubectl edit
`译者：hurf` `校对：无`


编辑服务端的资源。

### 摘要

使用系统默认编辑器编辑服务端的资源。

edit命令允许你直接编辑使用命令行工具获取的任何资源。此命令将打开你通过KUBE_EDITOR，GIT_EDITOR
或者EDITOR环境变量定义的编辑器，或者直接使用“vi”。你可以同时编辑多个资源，但所有的变更都只会一次性
提交。除了命令行参数外，此命令也接受文件名，但所直指定的文件必须使用早于当前的资源版本。

所编辑的文件将使用默认的API版本输出，或者通过--output-version选项显式指定。默认的输出格式是YAML，
如果你需要使用JSON格式编辑，使用-o json选项。

如果当更新资源的时候出现错误，将会在磁盘上创建一个临时文件，记录未成功的更新。更新资源时最常见的错误
是其他人也在更新服务端的资源。当这种情况发生时，你需要将你所作的更改应用到最新版本的资源上，或者编辑
保存的临时文件，使用最新的资源版本。

```
kubectl edit (RESOURCE/NAME | -f FILENAME)
```

### 示例

```
  # 编辑名为“docker-registry”的service
  $ kubectl edit svc/docker-registry

  # 使用一个不同的编辑器
  $ KUBE_EDITOR="nano" kubectl edit svc/docker-registry

  # 编辑名为“docker-registry”的service，使用JSON格式、v1 API版本
  $ kubectl edit svc/docker-registry --output-version=v1 -o json
```

### 选项

```
  -f, --filename=[]: 用来指定待编辑资源的文件名，目录名或者URL。
  -o, --output="yaml": 输出格式，可选yaml或者json中的一种。
      --output-version="": 输出资源使用的API版本（默认使用api-version）。
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
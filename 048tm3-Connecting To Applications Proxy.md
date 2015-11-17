
# 应用：kubectl proxy和apiserver proxy

你已经看过了kubectl proxy和apiserver proxy的基本知识。本页将为你展示如何一起使用它们去访问运行在你的工作站上的Kubernetes集群提供的服务（kube-ui）。


# 获取kube-ui的apiserver proxy URL


Kube-ui是作为一个集群的插件进行部署的。想要找到他的apiserver proxy URL：
```
$ kubectl cluster-info | grep "KubeUI"
KubeUI is running at https://173.255.119.104/api/v1/proxy/namespaces/kube-system/services/kube-ui

```
如果这条命令不能找到URL，试试这些步骤。

# 从你的本地工作站连接到kube-ui服务

上面说的proxy URL是由apiserver提供来访问kube-ui服务的。你要是在本地想访问它仍然需要用apiserver认证。Kubectl proxy可以处理认证。
```
$ kubectl proxy --port=8001
Starting to serve on localhost:8001

```
现在你就可以在你的本地工作站访问kube-ui服务了，使用如下地址
http://localhost:8001/api/v1/proxy/namespaces/kube-system/services/kube-ui

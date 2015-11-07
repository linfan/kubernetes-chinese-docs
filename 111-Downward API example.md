#Downward API 范例

接下来的例子，你会创建一个pod, 它包含了一个通过访问[downward API](http://kubernetes.io/v1.0/docs/user-guide/downward-api.html)来使用这个pod名字和命名空的容器

#步骤 0: 前提条件

这个例子会假设你有一个安装好并正在运行的Kubernetes集群，也已经在系统的某个路径下安装好了kubectl命令行工具。具体安装步骤请在[安装入门](http://kubernetes.io/v1.0/docs/getting-started-guides/)中找到与你平台对应的安装说明。

#步骤 1: 创建一个pod

容器可以通过环境变量来消费downward API，而且downward API允许容器通过被注入的方来式使用所在pod的name和namespace等信息

Use the examples/downward-api/dapi-pod.yaml file to create a Pod with a container that consumes the downward API.

我们使用examples/downward-api/dapi-pod.yaml来创建

$ kubectl create -f docs/user-guide/downward-api/dapi-pod.yaml

#检查日志

This pod runs the env command in a container that consumes the downward API. You can grep through the pod logs to see that the pod was injected with the correct values:

$ kubectl logs dapi-test-pod | grep POD_
2015-04-30T20:22:18.568024817Z POD_NAME=dapi-test-pod
2015-04-30T20:22:18.568087688Z POD_NAMESPACE=default

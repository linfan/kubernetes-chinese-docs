#Downward API 范例

接下来的例子，你会创建一个pod, 它包含了一个通过访问[downward API](http://kubernetes.io/v1.0/docs/user-guide/downward-api.html)来使用这个pod名字和命名空的容器

#步骤 0: 前提条件

This example assumes you have a Kubernetes cluster installed and running, and that you have installed the kubectl command line tool somewhere in your path. Please see the [getting started]() for installation instructions for your platform.

#步骤 1: 创建一个pod

Containers consume the downward API using environment variables. The downward API allows containers to be injected with the name and namespace of the pod the container is in.

Use the examples/downward-api/dapi-pod.yaml file to create a Pod with a container that consumes the downward API.

$ kubectl create -f docs/user-guide/downward-api/dapi-pod.yaml

#检查日志

This pod runs the env command in a container that consumes the downward API. You can grep through the pod logs to see that the pod was injected with the correct values:

$ kubectl logs dapi-test-pod | grep POD_
2015-04-30T20:22:18.568024817Z POD_NAME=dapi-test-pod
2015-04-30T20:22:18.568087688Z POD_NAMESPACE=default

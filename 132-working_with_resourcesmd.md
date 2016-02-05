
`译者：安雪艳` `校对：无`

# **资源管理**

本篇文章的目的是为了那些已经使用过一些示例并且想学习更多关于kubectl管理资源pod或者services的用户。想直接通过REST API访问的用户和想扩展Kubernetes API的开发者都应该参考文档 [api conventions](../devel/api-conventions.html)和[api document](../api.html).

# **资源自动更新**

当创建一个资源例如pod，随后会收到被创建和资源已添加的的多个字段。可以参考下面的工作示例：
```
$ cat > /tmp/original.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: foo
spec:
  containers:
    - name: foo
      image: busybox
  restartPolicy: Never
EOF
$ kubectl create -f /tmp/original.yaml
pods/original
$ kubectl get pods/original -o yaml > /tmp/current.yaml
pods/original
$ wc -l /tmp/original.yaml /tmp/current.yaml
      51 /tmp/current.yaml
       9 /tmp/original.yaml
      60 total
```
我们提交的资源只有9行，但是我们收到了51行。如果你执行```diff -u /tmp/original.yaml /tmp/current.yaml```，你可以看到pod增加的字段。系统通过如下几种方式增加字段：
* 一些字段时在资源创建时同步设置的，一些异步设置的。
    * 例如：```metadata.uid```同步设置. (获取更多信息请看 [metadata](../devel/api-conventions.html#metadata)).
    * 例如，```status.hostIP```是在pod被调度后设置。这些谁都发生的比较快。但是你可以注意到还没有设置的pods。这叫做初始化。 (获取更多信息请看[status](../devel/api-conventions.html#spec-and-status)和[late initialization](../devel/api-conventions.html#late-initialization) )。
* 一些字段被设置了默认值。一些默认值是根据cluster设置，一些默认值是API特定版本固定的。(获取更多信息请看 [defaulting](../devel/api-conventions.html#late-initialization)).
    * 例如， ```spec.containers[0].imagePullPolicy``` 在api v1版本的默认值一直时```IfNotPresent```。
    * 例如，```spec.containers[0].resources.limits.cpu``` 在一些cluster可能默认设置为```100m```，其他clueter中是其他的值，并且不是在所有的clueter中都默认设置。API一般不会改变你已经设置的字段；它只会设置那些你没有定义的字段。
    
# **资源文档**

你可以在[project website](http://kubernetes.io/v1.1/api-ref.html)或者[github](https://releases.k8s.io/release-1.1/docs/api-reference)上浏览自动生成的API问文档.


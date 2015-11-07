# **Storm 示例**

接下来的例子中，你将会使用Kubernetes和[Docker](http://docker.io/)来创建一个多功能的[Apache Storm](http://storm.apache.org/)集群。

你将会设置一个[Apache ZooKeeper](http://zookeeper.apache.org/)服务，一个Storm master服务（又名Nimbus主机），以及一个Storm工作者集合（又名监管者）。

如果你已经熟悉这个部分，请直接跳到[tl;dr](http://kubernetes.io/v1.0/examples/storm/README.html#tldr)章节。

### 资源

可以自由获取这些资源文件：

* Docker镜像 - [https://github.com/mattf/docker-storm](https://github.com/mattf/docker-storm)
* Docker受信的构建文件 - [https://registry.hub.docker.com/search?q=mattf/storm](https://registry.hub.docker.com/search?q=mattf/storm)


## **步骤零：前期准备**

这个示例假设你已经安装运行了一个Kubernetes集群，环境路径下已经安装了`kubectl`命令行工具。请查看不同平台的安装说明[开始](http://kubernetes.io/v1.0/docs/getting-started-guides/)。

## **步骤一：启动ZooKeeper服务**

ZooKeeper是一个分布式协调者[服务](http://kubernetes.io/v1.0/docs/user-guide/services.html)，Storm使用它来作为引导程序和存储运行转态数据。

使用这个[examples/storm/zookeeper.json](http://kubernetes.io/v1.0/examples/storm/zookeeper.json)文件来创建一个运行zookeeper的pod。



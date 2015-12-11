#Meteor on Kuberenetes
`译者：贾澜鹏` `校对：无`


这个例子将会向你展示如何在Kubernetes上打包运行一个[Meteor](https://www.meteor.com/) app。

##从谷歌的计算引擎开始
Meteor使用MongoDB，并且我们使用GCEPersistentDisk类型的卷作为永久存储介质。所以，这个实例只使用于[谷歌的计算引擎](https://cloud.google.com/compute/)。如果想选用其它的方式，可以去查看一下[卷使用文档](http://kubernetes.io/v1.1/docs/user-guide/volumes.html)。

首先，你需要完成如下操作：

1. [建立](https://cloud.google.com/compute/docs/quickstart)一个谷歌[云平台](https://cloud.google.com/)的项目。

2. 启用Google的[付费API](https://developers.google.com/console/help/new/#billing)？？

3. 安装[谷歌云的SDK](https://cloud.google.com/sdk/)。

认证谷歌云并且将谷歌云的默认项目名称指向你希望部署Kubernetes集群的项目：

	gcloud auth login
	gcloud config set project <project-name>

之后，开启一个Kubernetes集群

	wget -q -O - https://get.k8s.io | bash

所有细节和其它方式下启动集群的方法，请参考[谷歌GCE平台的Kubernetes入门指导](http://kubernetes.io/v1.1/docs/getting-started-guides/gce.html)。

## 为你的Meteor APP创建一个容器

为了使一个Meteor APP能运行再Kubernetes上，你首先需要建立一个Docker容器。在建立之前你需要安装[DOCKER](https://www.docker.com/)。一旦这些都具备了，你需要将`Dockerfile`和`.dockerignore`这2个文件添加到当前的Meteor工程中。

Dockerfile文件中应该包含下面的语句。其中的`ROOT_URL`你应该替换为当前APP的主机名。

	FROM chees/meteor-kubernetes
	ENV ROOT_URL http://myawesomeapp.com

对于`.dockerignore`文件，应该包含下面的语句。它是为了告诉Docker，再建立你的容器时忽略以下指定路径的文件。

	.meteor/local
	packages/*/.build*

你可以在下面的链接中看到一个已经建立起来的Meteor工程：[meteor-gke-example](https://github.com/Q42/meteor-gke-example)。你可以随意的使用这个例子中的APP。
如果你已经在你的Meteor工程中添加了移动平台，那么下面的步骤将不起作用。所以替换你的平台为 `meteor list-platforms`。

现在，你就可以在你的Meteor工程中运行下面的语句来建立一个容器：

	docker build -t my-meteor .

## 推送一个注册表

对于[Docker Hub](https://hub.docker.com/)，你需要利用下面的命令向Hub推送一个携带了你的用户名的APP图片，请注意替换`<username>`为你对应的Hub 用户名。命令如下：

	docker tag my-meteor <username>/my-meteor
	docker push <username>/my-meteor

对于[Google Container Registry](https://cloud.google.com/tools/container-registry/)，你需要向GCR推送一个携带你的工程ID的APP图片，同时，请注意替换`<project>`为你的对应工程ID。

	docker tag my-meteor gcr.io/<project>/my-meteor
	gcloud docker push gcr.io/<project>/my-meteor

## 运行

现在，你已经容器化了你的Meteor APP，是时候开始建立你的集群了。编辑`meteor-controller.json` 同时确保`image:`与你刚刚推送到Docker Hub或者GCR的容器是相对应的。
我们将需要一个MongoDB作为一个永久的Kubernetes卷来存放数据。相关的选项可以参考[volumes documentation](http://kubernetes.io/v1.1/docs/user-guide/volumes.html)。我们将利用谷歌的计算引擎永久磁盘。创建MongoDB磁盘的命令如下：

	gcloud compute disks create --size=200GB mongo-disk

你可以开启Mongo去使用那些磁盘，命令如下：

	kubectl create -f examples/meteor/mongo-pod.json
	kubectl create -f examples/meteor/mongo-service.json

等待Mongo完全启动，之后你就可以开启自己的Meteor APP:

	kubectl create -f examples/meteor/meteor-service.json
	kubectl create -f examples/meteor/meteor-controller.json

需要注意的是，`meteor-service.json`创建了一个负载均衡器，所以你的APP要求能在Meteor Pods启动时，有效的通过负载均衡器的IP。我们会再建立RC之前建立一个服务，该服务会提供反向关联（或者其它的什么）用以帮助调度程序去排列pod，从而帮助调度程序匹配对应的pods。你可以通过下面的命令获得你的负载均衡器的IP：

	kubectl get service meteor --template="{{range .status.loadBalancer.ingress}} {{.ip}} {{end}}"

之后，你需要在你的环境上开启你的80号端口。如果是使用谷歌计算引擎的用户，你需要运行下面的命令：

	gcloud compute firewall-rules create meteor-80 --allow=tcp:80 --target-tags kubernetes-minion

## 接下来呢？

首先，在`Dockerfile`中的`FROM chees/meteor-kubernetes`语句会指定Meteor APP的基础镜像。这个镜像的构建代码放置在示例代码工程的`dockerbase/`子目录下。你可以通过阅读`Dockerfile`文件中的代码来了解`docker build`的步骤。这个镜像是基于Node.js官方镜像构建的。它会安装Meteor并将用户的程序拷贝进去。其中的最后一行的命令指出了你的app需要通过怎样的命令在容器中运行起来。
	
	ENTRYPOINT MONGO_URL=mongodb://$MONGO_SERVICE_HOST:$MONGO_SERVICE_PORT /usr/local/bin/node main.js

在上面的命令中，我们能看到传递到Meteor App的MongoDB主机和端口信息。`MONGO_SERVICE...`这个环境变量是由Kubernetes设置的，同时，由`mongo-service.json`指定了名为`mongo`的服务器的详细信息。更多细节可以参考[environment documentation](http://kubernetes.io/v1.1/docs/user-guide/container-environment.html)。

也许你已经了解，Meteor需要使用TCP长连接，以及持续性的Session（Sticky Sessions）。通过Kubernetes用户能够轻松的在集群Scale out时保持节点的Session相关性。在meteor-service.json文件中所包含的`"sessionAffinity"* : *"ClientIP"`，就向我们提供了这些。更多的信息请参考 [service documentation](http://kubernetes.io/v1.1/docs/user-guide/services.html#virtual-ips-and-service-proxies)。

就向之前所提到的，mongo容器使用了一个被Kubernetes映射到永久磁盘的卷。在`mongo-pod.json`中，容器对指定的卷进行了划分：

	{
        "volumeMounts": [
          {
            "name": "mongo-disk",
            "mountPath": "/data/db"
          }

mongo-disk是指超出容器范围的卷：

	{
    "volumes": [
      {
        "name": "mongo-disk",
        "gcePersistentDisk": {
          "pdName": "mongo-disk",
          "fsType": "ext4"
        }
      }
    ],
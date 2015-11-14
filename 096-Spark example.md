#Spark例子
在下面的例子中，你会学习使用Kubernetes和[Docker](http://docker.io/)创建一个能够使用的阿帕奇[Spark](http://spark.apache.org/)集群。我们使用Spark的[单例模式](http://spark.apache.org/docs/latest/spark-standalone.html)创建一个Spark master节点服务和一系列Spark workers节点。

对于不耐心的专家，直接跳到[这个部分](http://kubernetes.io/v1.0/examples/spark/README.html#tldr)

##资源
Docker镜像很重，基于[https://github.com/mattf/docker-spark](https://github.com/mattf/docker-spark)

##步骤0:预备知识
下面的例子假设你已经安装和运行一个Kubernetes集群，并且你也下载好**kubectl**命令行工具以及配置在你的path中。请先阅读[入门指南](http://kubernetes.io/v1.0/docs/getting-started-guides/)获取在你平台上安装Kubernetes的指令。

##步骤一：启动Master服务
Master[服务](http://kubernetes.io/v1.0/docs/user-guide/services.html)是一个Spark集群的主服务（或头服务）。

使用**[examples/spark/spark-master.json](http://kubernetes.io/v1.0/examples/spark/spark-master.json)**文件创建运行在Master服务中的[pod](http://kubernetes.io/v1.0/docs/user-guide/pods.html)。
```language
$ kubectl create -f examples/spark/spark-master.json
```

然后使用**[examples/spark/spark-master-service.json](http://kubernetes.io/v1.0/examples/spark/spark-master-service.json)**文件创建一个逻辑服务端点供Spark workers节点使用连接Matser pod。
```language
$ kubectl create -f examples/spark/spark-master-service.json
```

###检查Master节点使用运行并且能够连接
```language
$ kubectl get pods
NAME                                           READY     STATUS    RESTARTS   AGE
[...]
spark-master                                   1/1       Running   0          25s
```
检查日志查看master节点的状态：
```language
$ kubectl logs spark-master

starting org.apache.spark.deploy.master.Master, logging to /opt/spark-1.4.0-bin-hadoop2.6/sbin/../logs/spark--org.apache.spark.deploy.master.Master-1-spark-master.out
Spark Command: /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java -cp /opt/spark-1.4.0-bin-hadoop2.6/sbin/../conf/:/opt/spark-1.4.0-bin-hadoop2.6/lib/spark-assembly-1.4.0-hadoop2.6.0.jar:/opt/spark-1.4.0-bin-hadoop2.6/lib/datanucleus-api-jdo-3.2.6.jar:/opt/spark-1.4.0-bin-hadoop2.6/lib/datanucleus-rdbms-3.2.9.jar:/opt/spark-1.4.0-bin-hadoop2.6/lib/datanucleus-core-3.2.10.jar -Xms512m -Xmx512m -XX:MaxPermSize=128m org.apache.spark.deploy.master.Master --ip spark-master --port 7077 --webui-port 8080
========================================
15/06/26 14:01:49 INFO Master: Registered signal handlers for [TERM, HUP, INT]
15/06/26 14:01:50 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
15/06/26 14:01:51 INFO SecurityManager: Changing view acls to: root
15/06/26 14:01:51 INFO SecurityManager: Changing modify acls to: root
15/06/26 14:01:51 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users with view permissions: Set(root); users with modify permissions: Set(root)
15/06/26 14:01:51 INFO Slf4jLogger: Slf4jLogger started
15/06/26 14:01:51 INFO Remoting: Starting remoting
15/06/26 14:01:52 INFO Remoting: Remoting started; listening on addresses :[akka.tcp://sparkMaster@spark-master:7077]
15/06/26 14:01:52 INFO Utils: Successfully started service 'sparkMaster' on port 7077.
15/06/26 14:01:52 INFO Utils: Successfully started service on port 6066.
15/06/26 14:01:52 INFO StandaloneRestServer: Started REST server for submitting applications on port 6066
15/06/26 14:01:52 INFO Master: Starting Spark master at spark://spark-master:7077
15/06/26 14:01:52 INFO Master: Running Spark version 1.4.0
15/06/26 14:01:52 INFO Utils: Successfully started service 'MasterUI' on port 8080.
15/06/26 14:01:52 INFO MasterWebUI: Started MasterWebUI at http://10.244.2.34:8080
15/06/26 14:01:53 INFO Master: I have been elected leader! New state: ALIVE
```

##步骤二：启动Spark workers
在Spark集群中Spark workers做繁重的工作。它们为你的程序提供计算资源和数据缓存的能力。

Spark workers需要Master服务支持才能运行。

使用**[examples/spark/spark-worker-controller.json](http://kubernetes.io/v1.0/examples/spark/spark-worker-controller.json)**文件创建[复制控制器](http://kubernetes.io/v1.0/docs/user-guide/replication-controller.html)管理workers pod。
```language
$ kubectl create -f examples/spark/spark-worker-controller.json
```

###检查workers节点是否运行
```language
$ kubectl get pods
NAME                                            READY     STATUS    RESTARTS   AGE
[...]
spark-master                                    1/1       Running   0          14m
spark-worker-controller-hifwi                   1/1       Running   0          33s
spark-worker-controller-u40r2                   1/1       Running   0          33s
spark-worker-controller-vpgyg                   1/1       Running   0          33s

$ kubectl logs spark-master
[...]
15/06/26 14:15:43 INFO Master: Registering worker 10.244.2.35:46199 with 1 cores, 2.6 GB RAM
15/06/26 14:15:55 INFO Master: Registering worker 10.244.1.15:44839 with 1 cores, 2.6 GB RAM
15/06/26 14:15:55 INFO Master: Registering worker 10.244.0.19:60970 with 1 cores, 2.6 GB RAM
```

##步骤三：使用集群做点什么
获取Master服务的地址和端口。
```language
$ kubectl get service spark-master
NAME           LABELS              SELECTOR            IP(S)          PORT(S)
spark-master   name=spark-master   name=spark-master   10.0.204.187   7077/TCP
```

使用SSH连接集群中的一个节点，在GCE/GKE（【译者注】指云服务谷歌计算引擎和谷歌Kubernetes引擎）上，你可以使用[开发者控制台](https://console.developers.google.com/)（更多细节点击[这里](https://cloud.google.com/compute/docs/ssh-in-browser)）或者运行**gcloud compute ssh <name>**，name可以通过**kubectl get nodes**命令获得（更多细节[在这](https://cloud.google.com/compute/docs/gcloud-compute/#connecting)）。
```language
$ kubectl get nodes
NAME                     LABELS                                          STATUS
kubernetes-minion-5jvu   kubernetes.io/hostname=kubernetes-minion-5jvu   Ready
kubernetes-minion-6fbi   kubernetes.io/hostname=kubernetes-minion-6fbi   Ready
kubernetes-minion-8y2v   kubernetes.io/hostname=kubernetes-minion-8y2v   Ready
kubernetes-minion-h0tr   kubernetes.io/hostname=kubernetes-minion-h0tr   Ready

$ gcloud compute ssh kubernetes-minion-5jvu --zone=us-central1-b
Linux kubernetes-minion-5jvu 3.16.0-0.bpo.4-amd64 #1 SMP Debian 3.16.7-ckt9-3~deb8u1~bpo70+1 (2015-04-27) x86_64

=== GCE Kubernetes node setup complete ===

me@kubernetes-minion-5jvu:~$
```
一旦登陆成功就可以使用Spark基础镜像了。在镜像中有一个脚本用来设置基于Master的IP和端口环境。
```language
cluster-node $ sudo docker run -it gcr.io/google_containers/spark-base
root@f12a6fec45ce:/# . /setup_client.sh 10.0.204.187 7077
root@f12a6fec45ce:/# pyspark
Python 2.7.9 (default, Mar  1 2015, 12:57:24) 
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
15/06/26 14:25:28 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 1.4.0
      /_/
Using Python version 2.7.9 (default, Mar  1 2015 12:57:24)
SparkContext available as sc, HiveContext available as sqlContext.
>>> import socket
>>> sc.parallelize(range(1000)).map(lambda x:socket.gethostname()).distinct().collect()
['spark-worker-controller-u40r2', 'spark-worker-controller-hifwi', 'spark-worker-controller-vpgyg']
```

##结果
现在你已经为你的Spark master节点和Spark workers节点创建了服务、复制控制器和pods。你可以在下一篇文档中继续使用这个例子以及使用这个Spark集群。点击[Spark文档](https://spark.apache.org/documentation.html)获取更多信息。

##tl；dr
```language
kubectl create -f spark-master.json

kubectl create -f spark-master-service.json

Make sure the Master Pod is running (use: kubectl get pods).

kubectl create -f spark-worker-controller.json
```

[原文连接](http://kubernetes.io/v1.0/examples/spark/README.html)

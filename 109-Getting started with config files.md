#配置文件入门.

>除了其他地方所描述的命令行式风格的命令以外，Kubernetes还支持以YAML或者JSON格式的配置方式，很多时候，配置文件是优于纯命令方式的，一旦他们能够被签入版本管理，文件的变化也能够像代码文件一样被回顾和管理，就能生产处更健壮，可靠和可存档系统。


##通过pod的配置文件来运营容器

```
$ cd kubernetes
$ kubectl create -f ./pod.yaml
Where pod.yaml contains something like:

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```   

通过以下命令来查看集群中pod的信息:

```
$ kubectl get pods
and delete the pod you just created:

$ kubectl delete pods nginx
```

##通过配置文件来运行容器的复制集

To run replicated containers, you need a Replication Controller. A replication controller is responsible for ensuring that a specific number of pods exist in the cluster.

要运行复制的容器，你需要一个Controller的复制品。一个Controller的复制品负责保证规定数量的pod会一直存在于集群当中

```
$ cd kubernetes
$ kubectl create -f ./replication.yaml
```

Where replication.yaml contains:
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        
```

To delete the replication controller (and the pods it created):
删除controller的复制品

```
$ kubectl delete rc nginx
```
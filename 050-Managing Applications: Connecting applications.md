#管理应用：连接应用
##Kubernetes容器连接模型

既然已经有了一个可持续运行的、可复制的应用，现在就可以在网络中将它暴露出来了。在讨论Kubernetes的网络连接方式之前，很值得和Docker的常规网络连接方式做个对比。

Dokcer默认使用私有网络连接方式，所以只有在同一台物理机器上的容器之前才可以通信。为了能让Docker容器可以跨节点通信，必须要给机器的IP地址分配端口号，这个端口之后会被用来转发或者路由给容器。很明显，这意味着容器要么很小心地协调使用端口，要么有动态分配地端口。

在一定的规模下，为多个开发者协调端口号非常困难。这也会把集群级别的问题暴露给用户，这是在用户的控制之外的。Kubernetes假定pods之间是可以通信的，不管它们落到哪个主机上。我们给每个pod指定集群私有的IP地址（cluster-private-IP address），所以不需要显示地创建pod之间的链接，也不需要映射容器的端口到主机的端口。这意味着pod里的容器可以在本机（localhost）上访问各自的端口，而且在没有NAT的情况下，集群中所有的pod也可以互相可见的。本文剩下的内容将会详细阐述如何在这样的网络模型中运行可靠的服务。

这个指南中用了一个简单的nginx服务来演示验证这个概念（proof of concept）。同样的原理也在一个更完整的[Jenkins CI 应用](http://blog.kubernetes.io/2015/07/strong-simple-ssl-for-kubernetes.html)中体现了。

###Exposing pods to the cluster

在前面的例子中已经演示过，让我们把注意力集中在网络的视角在来一次。创建一个nginx的pod，请注意它定义了容器的端口：

```
$ cat nginxrc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
This makes it accessible from any node in your cluster. Check the nodes the pod is running on:
```
$ kubectl create -f ./nginxrc.yaml
$ kubectl get pods -l app=nginx -o wide
my-nginx-6isf4   1/1       Running   0          2h        e2e-test-beeps-minion-93ly
my-nginx-t26zt   1/1       Running   0          2h        e2e-test-beeps-minion-93ly
```
Check your pods ips:
```
$ kubectl get pods -l app=nginx -o json | grep podIP
                "podIP": "10.245.0.15",
                "podIP": "10.245.0.14",
```
You should be able to ssh into any node in your cluster and curl both ips. Note that the containers are not using port 80 on the node, nor are there any special NAT rules to route traffic to the pod. This means you can run multiple nginx pods on the same node all using the same containerPort and access them from any other pod or node in your cluster using ip. Like Docker, ports can still be published to the host node's interface(s), but the need for this is radically diminished because of the networking model.

You can read more about [how we achieve this](http://kubernetes.io/v1.0/docs/admin/networking.html#how-to-achieve-this) if you’re curious.
##Creating a Service
So we have pods running nginx in a flat, cluster wide, address space. In theory, you could talk to these pods directly, but what happens when a node dies? The pods die with it, and the replication controller will create new ones, with different ips. This is the problem a Service solves.

A Kubernetes Service is an abstraction which defines a logical set of Pods running somewhere in your cluster, that all provide the same functionality. When created, each Service is assigned a unique IP address (also called clusterIP). This address is tied to the lifespan of the Service, and will not change while the Service is alive. Pods can be configured to talk to the Service, and know that communication to the Service will be automatically load-balanced out to some pod that is a member of the Service.

You can create a Service for your 2 nginx replicas with the following yaml:
```
$ cat nginxsvc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```
This specification will create a Service which targets TCP port 80 on any Pod with the **app=nginx** label, and expose it on an abstracted Service port (**targetPort**: is the port the container accepts traffic on, **port**: is the abstracted Service port, which can be any port other pods use to access the Service). View [service API object](https://htmlpreview.github.io/?https://github.com/GoogleCloudPlatform/kubernetes/v1.0.1/docs/api-reference/definitions.html#_v1_service) to see the list of supported fields in service definition. Check your Service:
```
$ kubectl get svc
NAME         LABELS        SELECTOR    IP(S)          PORT(S)
nginxsvc     app=nginx     app=nginx   10.0.116.146   80/TCP

```
As mentioned previously, a Service is backed by a group of pods. These pods are exposed through **endpoints**. The Service's selector will be evaluated continuously and the results will be POSTed to an Endpoints object also named **nginxsvc**. When a pod dies, it is automatically removed from the endpoints, and new pods matching the Service’s selector will automatically get added to the endpoints. Check the endpoints, and note that the ips are the same as the pods created in the first step:
```
$ kubectl describe svc nginxsvc
Name:          nginxsvc
Namespace:     default
Labels:            app=nginx
Selector:      app=nginx
Type:          ClusterIP
IP:            10.0.116.146
Port:          <unnamed> 80/TCP
Endpoints:     10.245.0.14:80,10.245.0.15:80
Session Affinity:  None
No events.

$ kubectl get ep
NAME         ENDPOINTS
nginxsvc     10.245.0.14:80,10.245.0.15:80
```
You should now be able to curl the nginx Service on **10.0.116.146:80** from any node in your cluster. Note that the Service ip is completely virtual, it never hits the wire, if you’re curious about how this works you can read more about the [service proxy](http://kubernetes.io/v1.0/docs/user-guide/services.html#virtual-ips-and-service-proxies).
##Accessing the Service
Kubernetes supports 2 primary modes of finding a Service - environment variables and DNS. The former works out of the box while the latter requires the [kube-dns cluster addon](http://releases.k8s.io/v1.0.6/cluster/addons/dns/README.md).
###Environment Variables
When a Pod is run on a Node, the kubelet adds a set of environment variables for each active Service. This introduces an ordering problem. To see why, inspect the environment of your running nginx pods:
```
$ kubectl exec my-nginx-6isf4 -- printenv | grep SERVICE
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
```
Note there’s no mention of your Service. This is because you created the replicas before the Service. Another disadvantage of doing this is that the scheduler might put both pods on the same machine, which will take your entire Service down if it dies. We can do this the right way by killing the 2 pods and waiting for the replication controller to recreate them. This time around the Service exists before the replicas. This will given you scheduler level Service spreading of your pods (provided all your nodes have equal capacity), as well as the right environment variables:
```
$ kubectl scale rc my-nginx --replicas=0; kubectl scale rc my-nginx --replicas=2;
$ kubectl get pods -l app=nginx -o wide
NAME             READY   STATUS     RESTARTS   AGE   NODE
my-nginx-5j8ok   1/1     Running       0         2m    node1
my-nginx-90vaf   1/1     Running   0          2m    node2

$ kubectl exec my-nginx-5j8ok -- printenv | grep SERVICE
KUBERNETES_SERVICE_PORT=443
NGINXSVC_SERVICE_HOST=10.0.116.146
KUBERNETES_SERVICE_HOST=10.0.0.1
NGINXSVC_SERVICE_PORT=80
```
###DNS
Kubernetes offers a DNS cluster addon Service that uses skydns to automatically assign dns names to other Services. You can check if it’s running on your cluster:
```
$ kubectl get services kube-dns --namespace=kube-system
NAME       LABELS       SELECTOR             IP(S)       PORT(S)
kube-dns   <none>       k8s-app=kube-dns     10.0.0.10   53/UDP
                                                         53/TCP
```
If it isn’t running, you can [enable it](http://releases.k8s.io/v1.0.6/cluster/addons/dns/README.md#how-do-i-configure-it). The rest of this section will assume you have a Service with a long lived ip (nginxsvc), and a dns server that has assigned a name to that ip (the kube-dns cluster addon), so you can talk to the Service from any pod in your cluster using standard methods (e.g. gethostbyname). Let’s create another pod to test this:
```
$ cat curlpod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: curlpod
spec:
  containers:
  - image: radial/busyboxplus:curl
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: curlcontainer
  restartPolicy: Always
```
And perform a lookup of the nginx Service
```
$ kubectl create -f ./curlpod.yaml
default/curlpod
$ kubectl get pods curlpod
NAME      READY     STATUS    RESTARTS   AGE
curlpod   1/1       Running   0          18s

$ kubectl exec curlpod -- nslookup nginxsvc
Server:    10.0.0.10
Address 1: 10.0.0.10
Name:      nginxsvc
Address 1: 10.0.116.146
```
##Securing the Service
Till now we have only accessed the nginx server from within the cluster. Before exposing the Service to the internet, you want to make sure the communication channel is secure. For this, you will need:
* Self signed certificates for https (unless you already have an identitiy certificate)
* An nginx server configured to use the cretificates
* A [secret](http://kubernetes.io/v1.0/docs/user-guide/secrets.html) that makes the certificates accessible to pods

You can acquire all these from the [nginx https example](http://kubernetes.io/v1.0/examples/https-nginx/README.html), in short:
```
$ make keys secret KEY=/tmp/nginx.key CERT=/tmp/nginx.crt SECRET=/tmp/secret.json
$ kubectl create -f /tmp/secret.json
secrets/nginxsecret
$ kubectl get secrets
NAME                  TYPE                                  DATA
default-token-il9rc   kubernetes.io/service-account-token   1
nginxsecret           Opaque  
```
Now modify your nginx replicas to start a https server using the certificate in the secret, and the Service, to expose both ports (80 and 443):
```
$ cat nginx-app.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
  labels:
    app: nginx
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
  selector:
    app: nginx
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: secret-volume
        secret:
          secretName: nginxsecret
      containers:
      - name: nginxhttps
        image: bprashanth/nginxhttps:1.0
        ports:
        - containerPort: 443
        - containerPort: 80
        volumeMounts:
        - mountPath: /etc/nginx/ssl
          name: secret-volume
```
Noteworthy points about the nginx-app manifest:
* It contains both rc and service specification in the same file
* The [nginx server](http://kubernetes.io/v1.0/examples/https-nginx/default.conf) serves http traffic on port 80 and https traffic on 443, and nginx Service exposes both ports.
* Each container has access to the keys through a volume mounted at /etc/nginx/ssl. This is setup before the nginx server is started.

```
$ kubectl delete rc,svc -l app=nginx; kubectl create -f ./nginx-app.yaml
replicationcontrollers/my-nginx
services/nginxsvc
services/nginxsvc
replicationcontrollers/my-nginx
```
At this point you can reach the nginx server from any node.
```
$ kubectl get pods -o json | grep -i podip
    "podIP": "10.1.0.80",
node $ curl -k https://10.1.0.80
...
<h1>Welcome to nginx!</h1>
```
Note how we supplied the -k parameter to curl in the last step, this is because we don't know anything about the pods running nginx at certificate generation time, so we have to tell curl to ignore the CName mismatch. By creating a Service we linked the CName used in the certificate with the actual DNS name used by pods during Service lookup. Lets test this from a pod (the same secret is being reused for simplicity, the pod only needs nginx.crt to access the Service):
```
$ cat curlpod.yaml
vapiVersion: v1
kind: ReplicationController
metadata:
  name: curlrc
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: curlpod
    spec:
      volumes:
      - name: secret-volume
        secret:
          secretName: nginxsecret
      containers:
      - name: curlpod
        command:
        - sh
        - -c
        - while true; do sleep 1; done
        image: radial/busyboxplus:curl
        volumeMounts:
        - mountPath: /etc/nginx/ssl
          name: secret-volume

$ kubectl create -f ./curlpod.yaml
$ kubectl get pods
NAME             READY     STATUS    RESTARTS   AGE
curlpod          1/1       Running   0          2m
my-nginx-7006w   1/1       Running   0          24m

$ kubectl exec curlpod -- curl https://nginxsvc --cacert /etc/nginx/ssl/nginx.crt
...
<title>Welcome to nginx!</title>
...
```
##Exposing the Service
For some parts of your applications you may want to expose a Service onto an external IP address. Kubernetes supports two ways of doing this: NodePorts and LoadBalancers. The Service created in the last section already used **NodePort**, so your nginx https replica is ready to serve traffic on the internet if your node has a public ip.
```
$ kubeclt get svc nginxsvc -o json | grep -i nodeport -C 5
            {
                "name": "http",
                "protocol": "TCP",
                "port": 80,
                "targetPort": 80,
                "nodePort": 32188
            },
            {
                "name": "https",
                "protocol": "TCP",
                "port": 443,
                "targetPort": 443,
                "nodePort": 30645
            }

$ kubectl get nodes -o json | grep ExternalIP
                    {
                        "type": "ExternalIP",
                        "address": "104.197.63.17"
                    }
--
                    },
                    {
                        "type": "ExternalIP",
                        "address": "104.154.89.170"
                    }
$ curl https://104.197.63.17:30645 -k
...
<h1>Welcome to nginx!</h1>
```
Lets now recreate the Service to use a cloud load balancer, just change the **Type** of Service in the nginx-app.yaml from **NodePort** to **LoadBalancer**:
```
$ kubectl delete rc, svc -l app=nginx
$ kubectl create -f ./nginx-app.yaml
$ kubectl get svc -o json | grep -i ingress -A 5
                    "ingress": [
                        {
                            "ip": "104.197.68.43"
                        }
                    ]
                }
$ curl https://104.197.68.43 -k
...
<title>Welcome to nginx!</title>
```
##What's next?
[Learn about more Kubernetes features that will help you run containers reliably in production.](http://kubernetes.io/v1.0/docs/user-guide/production-pods.html)
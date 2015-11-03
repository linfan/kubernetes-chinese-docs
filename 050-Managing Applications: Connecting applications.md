#管理应用：连接应用
##Kubernetes容器连接模型
既然已经有了一个可持续运行的、可复制的应用，现在就可以在网络中暴露出来了。
Now that you have a continuously running, replicated application you can expose it on a network. Before discussing the Kubernetes approach to networking, it is worthwhile to contrast it with the "normal" way networking works with Docker.

By default, Docker uses host-private networking, so containers can talk to other containers only if they are on the same machine. In order for Docker containers to communicate across nodes, they must be allocated ports on the machine's own IP address, which are then forwarded or proxied to the containers. This obviously means that containers must either coordinate which ports they use very carefully or else be allocated ports dynamically.

Coordinating ports across multiple developers is very difficult to do at scale and exposes users to cluster-level issues outside of their control. Kubernetes assumes that pods can communicate with other pods, regardless of which host they land on. We give every pod its own cluster-private-IP address so you do not need to explicitly create links between pods or mapping container ports to host ports. This means that containers within a Pod can all reach each other’s ports on localhost, and all pods in a cluster can see each other without NAT. The rest of this document will elaborate on how you can run reliable services on such a networking model.

This guide uses a simple nginx server to demonstrate proof of concept. The same principles are embodied in a more complete [Jenkins CI application](http://blog.kubernetes.io/2015/07/strong-simple-ssl-for-kubernetes.html).
###Exposing pods to the cluster
We did this in a previous example, but lets do it once again and focus on the networking perspective. Create an nginx pod, and note that it has a container port specification:
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
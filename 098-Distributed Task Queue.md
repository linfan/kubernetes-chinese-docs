# 示例: 分部署任务队列 Celery, RabbitMQ和Flower
## 介绍
Celery是基于分布式消息传递的异步任务队列。它可以用来创建一些执行单元（例如，一个任务），这些任务可以同步，异步的在一个或者多个工作节点执行。

Celery基于Python实现。

因为Celery基于消息传递，需要一些叫作消息代理的中间件（他们用来在发送者和接受者之间处理传递的消息）。RabbitMQ是一种和Celery联合使用的消息中间件。

下面的示例将向你展示，如何使用Kubernetes来建立一个基于Celery作为任务队列，RabbitMQ作为消息代理的分布式任务队列系统。同时，还要展示如何建立一个基于Flower的任务监控前端。

## 目标
### 在例子的最后，我们可以看到：

* 3个pods:
    * 一个Celery任务队列
    * 一个RabbitMQ消息代理
    * Flower前端
* 一个提供访问消息代理的服务
* 可以传递给工作节点的级别Celery任务

## 先决条件

你应该已经拥有一个Kubernetes集群。要完成大部分的例子，确保Kubernetes创建一个以上的节点（例如，通过设置`NUM_MINIONS`环境变量为2或者更多）。

### 第一步：启动RabbitMQ服务

Celery任务队列需要连接到RabbitMQ代理。RabbitMQ队列最终会出现在一个独立的pod上，但是，由于pod是短暂存在的，需要一个服务来透明的路由请求到RabbitMQ。

使用这个文件`examples/celery-rabbitmq/rabbitmq-service.yaml`

```json
apiVersion: v1
kind: Service
metadata:
  labels:
    name: rabbitmq
  name: rabbitmq-service
spec:
  ports:
  - port: 5672
  selector:
    app: taskQueue
    component: rabbitmq
```
这样运行一个服务：
```
$ kubectl create -f examples/celery-rabbitmq/rabbitmq-service.yaml
```
这个服务允许其他pods连接到rabbitmq。对于它们可以使用5672端口，服务也会将流量路由到容器（也通过5672端口）。

### 第二步：启动RabbitMQ







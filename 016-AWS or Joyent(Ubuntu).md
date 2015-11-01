
# 

Juju上部署入门

Juju使通过配置部署Kubernetes，在集群中安装和配置所有系统更加简单。可通过一个命令增加集群尺寸来简单的扩展部署集群

## 

内容列表
* 先决条件
    * Ubuntu上部署
    * Docker相关部署
* 运行Kubernetes集群
* 探索集群
* 运行多个容器！
* 扩展集群
* 运行“k8petsore”示例应用
* 拆除集群
* 更多信息
    * 云兼容

## 

先决条件

注意：如果你运行kube-up，在Ubuntu上——所有的依赖会为您处理。你可以跳转到这个部分：运行Kubernetes集群

### 

Ubuntu上部署

在您的本地Ubuntu系统安装Juju客户端。
```
sudo add-apt-repository ppa:juju/stable
sudo apt-get update
sudo apt-get install juju-core juju-quickstart
```

### 
Docker相关部署

如果您不使用Ubuntu，而是使用DOcker，您可以运行以下命令：
```
mkdir ~/.juju
sudo docker run -v ~/.juju:/home/ubuntu/.juju -ti jujusolutions/jujubox:latest
```
此时你可以在当前路径上获取```juju quickstart```命令。

为您所选择允许的云设置证书：

```
juju quickstart --constraints="mem=3.75G" -i```
容器参数是可选的，他将改变虚拟机尺寸。

## 
运行Kubernetes集群

## 

探索集群

## 

运行多个容器！

## 









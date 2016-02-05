# 故障排查
`译者：林帆` `校对：无`


在使用Kubernetes时，用户常常会遇到一些错误和迷惑，这个小节的内容将针对常见的问题进行解答。我们将这些问题分为以下两个部分：

- 应用程序相关的故障排查：当遇到将代码部署到Kubernetes时发生的各类错误时，请参考这部分内容。
- 集群相关的故障排查：当集群系统的管理者发现Kubernetes的集群工作不正常时，请参考这部分内容。

用户也可以查看[已知问题列表](https://github.com/kubernetes/kubernetes/blob/master/docs/user-guide/known-issues.md)来了解当前所使用的版本中有哪些我们已经发现的系统问题。

# 寻求帮助

如果以上的故障排查文档不能解决你所遇到的问题，还可以通过下面这几种方式寻求社区的帮助。

## 疑问

如果你对Kubernetes还不熟悉，文档的“用户指引”部分也许能帮助到你。

我们总结了一些用户经常遇到的问题，并分成下面几个方面记录下来了。

- 日常使用的常见问题
- 开发和调试的常见问题
- 服务相关的常见问题

此外，在Stack Overflow网站上也有相关的主题通道可供提问和参考。

- [Kubernetes相关的提问](http://stackoverflow.com/questions/tagged/kubernetes)
- [谷歌容器引擎GKE相关的提问](http://stackoverflow.com/questions/tagged/google-container-engine)

## 更多的帮助内容

如果已经提到的这些途径还不足以解答你所遇到的问题，下面提供了其他的一些获取答案的途径。

### Stack Overflow

社区中的其他人也许遇到过和你相同的问题，Kubernetes的开发者小组会订阅并定期的解答Stack Overflow上[包含有kubernetes标签的提问](http://stackoverflow.com/questions/tagged/kubernetes)。如果所有的这些提问都没有包含你所遇到的问题，你可以通过[这个链接](http://stackoverflow.com/questions/ask?tags=kubernetes)把问题告诉我们。

### Slack

Kubernetes开发小组的成员常常会出没在Slack的`#kubernetes-users`频道中，用户可以在这里直接将问题抛给第一线的开发者，并进行交流。Kubernetes的Slack开发者频道入口地址是[这里](https://kubernetes.slack.com/)，第一次进入频道前，用户需要先到[这里](http://slack.kubernetes.io/)进行注册。我们欢迎倾听到各种不同的声音和问题。

### 邮件组

谷歌容器引擎小组有个专用的邮件群组，地址是：[google-containers@googlegroups.com](https://groups.google.com/forum/#!forum/google-containers)

### Bug和需求

如果你发现的问题看起来像是一个潜在的Bug，或者你希望为Kubernetes的开发提出一些需求，请直接提交到Kubernetes在GitHub的[问题追踪页面](https://github.com/kubernetes/kubernetes/issues)里。

需要注意的是，在你到这个页面提出任何问题之前，请先在页面中搜索一下，确认没有其他人已经提过类似的问题。

当提出Bug时，请务必包含以下信息，以便我们能够尽快的重现并定位故障：

- Kubernetes的版本：`kubectl version`命令会告知你所需的信息
- 运行的环境（云服务商名称），操作系统发行版类型和版本，网络配置以及所用容器（例如Docker）的版本
- 重现故障的步骤
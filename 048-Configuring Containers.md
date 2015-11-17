
# 配置kubernetes


除了像kubectl run和kubectl expose这些必要的命令，kubernetes也支持可声明式的配置。配置文件也需要必要的命令，这样就可以在代码审查中检查版本控制和文件改动，而代码审查对复杂的具有鲁棒性的可靠生产系统是非常重要的。
在声明式风格中，所有的配置都保存在YAML或者JSON配置文件中，使用Kubernetes的API资源模式（schema）作为配置的模式（schema）。Kubectl命令可以创建、更新、删除以及获取API资源。kubectl命令用ApiVersion（目前是“v1”），kind资源，name资源去创建合适的API路径来执行特殊的操作。

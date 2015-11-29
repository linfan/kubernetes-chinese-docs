# **Kube调度器**

## **概要**
Kubernetes调度器是一个函数，policy-rich，topology-aware和workload-specific（三个形容词，翻译待考虑），显著影响可用性、性能和容量。该调度器需要考虑个人和集体的资源需求，服务质量需求，硬件、软件、策略约束，亲和力和非亲和力规范，数据局部性，内部工作负载干扰，截止日期等等。特定工作负载需求在必要时会通过API暴露。
```
kube-scheduler
```
## **选项**
```
      --address=127.0.0.1：服务的IP地址 (所有的接口，设置为0.0.0.0)
      --algorithm-provider="DefaultProvider"：提供的调度算法，DfaultProvider是其中一种。
      --bind-pods-burst=100：爆发期间每秒允许绑定的调度数量。
      --bind-pods-qps=50: 可以继续工作的每秒允许绑定的调度数量。
      --google-json-key=""；用户认证的Google Cloud Platform Service Account JSON Key。
      --kubeconfig="": 含有授权和主位置信息的Kube配置文件路径。
      --log-flush-frequency=5s：日志缓冲最大值，单位是秒。
      --master=""：Kubernetes API服务器地址（Kube配置文件里都有说明）
      --policy-config-file="": File with scheduler policy configuration
      --port=10251: The port that the scheduler's http service runs on
      --profiling[=true]: Enable profiling via web interface host:port/debug/pprof/
```
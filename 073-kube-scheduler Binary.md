# **Kube调度器**

## **概要**
Kubernetes调度器是一个函数，policy-rich，topology-aware和workload-specific（三个形容词，翻译待考虑），显著影响可用性、性能和容量。该调度器需要考虑个人和集体的资源需求，服务质量需求，硬件、软件、策略约束，亲和力和非亲和力规范，数据局部性，内部工作负载干扰，截止日期等等。特定工作负载需求在必要时会通过API暴露。
```
kube-scheduler
```
## **选项**
```
      --address=127.0.0.1: The IP address to serve on (set to 0.0.0.0 for all interfaces)
      --algorithm-provider="DefaultProvider": The scheduling algorithm provider to use, one of: DefaultProvider
      --bind-pods-burst=100: Number of bindings per second scheduler is allowed to make during bursts
      --bind-pods-qps=50: Number of bindings per second scheduler is allowed to continuously make
      --google-json-key="": The Google Cloud Platform Service Account JSON Key to use for authentication.
      --kubeconfig="": Path to kubeconfig file with authorization and master location information.
      --log-flush-frequency=5s: Maximum number of seconds between log flushes
      --master="": The address of the Kubernetes API server (overrides any value in kubeconfig)
      --policy-config-file="": File with scheduler policy configuration
      --port=10251: The port that the scheduler's http service runs on
      --profiling[=true]: Enable profiling via web interface host:port/debug/pprof/
```
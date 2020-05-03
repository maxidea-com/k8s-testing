# 通过Grafana展示nginx容器指标

本测试由三部分组成：

1）Nginx指标收集，由nginx-prometheus-exporter转化为prometheus可以处理的指标格式，这部分我们上一节已经完成，本节将复用我们做好的nginx-exporter:v0.1镜像。

2）配置Prometheus对指标进行收集（镜像[https://hub.docker.com/r/prom/prometheus](https://hub.docker.com/r/prom/prometheus)）

3）配置Grafana对指标进行展示

**测试1：配置Prometheus对指标进行收集**

首先，在宿主机上创建`prometheus.yml`文件，内容如下：

```text
global:
  scrape_interval:     15s 
  evaluation_interval: 15s 

scrape_configs:
     - job_name: 'prometheus'
       static_configs:
       - targets: ['localhost:9090']
     - job_name: "nginx"
       static_configs:
       - targets: ['localhost:9113']
```














# 通过Grafana展示nginx容器指标

本测试由三部分组成：

1）Nginx指标收集，由nginx-prometheus-exporter转化为prometheus可以处理的指标格式，这部分我们上一节已经完成。

2）配置Prometheus对指标进行收集

3）配置Grafana对指标进行展示




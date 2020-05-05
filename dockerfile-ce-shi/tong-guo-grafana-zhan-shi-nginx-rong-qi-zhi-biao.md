# 通过Grafana展示nginx容器指标

本测试由三部分组成：

1）Nginx指标收集，由nginx-prometheus-exporter转化为prometheus可以处理的指标格式，这部分我们上一节已经完成，本节将复用我们做好的nginx-exporter:v0.1镜像。

2）配置Prometheus对指标进行收集（镜像[https://hub.docker.com/r/prom/prometheus](https://hub.docker.com/r/prom/prometheus)）

3）配置Grafana对指标进行展示（镜像[https://hub.docker.com/r/grafana/grafana](https://hub.docker.com/r/grafana/grafana)）

## **测试1：配置Prometheus对指标进行收集**

首先，在宿主机上创建`prometheus.yml`文件，内容如下：

```text
global:
  scrape_interval:     15s 
  evaluation_interval: 15s 

scrape_configs:
     - job_name: 'prometheus'
       static_configs:
       - targets: ['localhost:9090']
     - job_name: 'nginx'
       static_configs:
       - targets: ['192.168.2.31:9113']
```

启动Prometheus容器：

```text
docker run --name prometheus1 -d -p 9090:9090 -v /simon-testing/docker/compose/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

然后访问宿主机 http://192.168.2.31:9090/targets，获得如下界面即代表Prometheus和Nginx指标均可获取：

![](../.gitbook/assets/image%20%281%29.png)

## 测试2：配置Grafana对指标进行展示

**1）准备dashboard.json**

首先我们需要定制一个适合展示prometheus收集回来的nginx的指标参数的dashboards仪表板模板，最快的方式是上官网的选择现成的，例如[https://grafana.com/grafana/dashboards/9516](https://grafana.com/grafana/dashboards/9516)

```text
wget -c https://grafana.com/api/dashboards/9516/revisions/3/download -O dashboard.json
```

**2）准备provisioning文件**

参考：[https://grafana.com/docs/grafana/latest/administration/provisioning/](https://grafana.com/docs/grafana/latest/administration/provisioning/)

2-1）定义存放dashboard模板的目录

创建一个dashboard.yaml文件：

```text
apiVersion: 1

providers:
- name: 'nginx'
  orgId: 1
  folder: 'dashboard1'
  folderUid: ''
  type: file
  disableDeletion: false
  editable: true
  updateIntervalSeconds: 10
  allowUiUpdates: false
  options:
    path: /var/lib/grafana/dashboards
```








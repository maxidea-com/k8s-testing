# 通过docker compose部署Grafana和Prometheus

基于我们之前制作的nginx-exporter:v0.2镜像，我们使用docker compose编排文件实现一次性部署Prometheus和Grafana

与前面拆分成三个容器的部署不一样的地方是，我们这里需要修改一下Prometheus和Grafana配置里的数据源地址。

```text
#more prometheus-compose.yml 
global:
  scrape_interval:     15s 
  evaluation_interval: 15s 

scrape_configs:
     - job_name: 'prometheus'
       static_configs:
       - targets: ['localhost:9090']
     - job_name: 'nginx_exporter'
       static_configs:
       - targets: ['nginx_exporter:9113']
         labels:
           group: 'services'
```

```text
#more datasource-compose.yaml 
apiVersion: 1

datasources:
- name: Prometheus
  type: prometheus
  access: proxy
  orgId: 1
  url: http://prometheus:9090
  basicAuth: false
  isDefault: true
  version: 1
  editable: true
```

清除之前测试的容器：

```text
docker stop $(docker ps -q) 
docker rm $(docker ps -aq)
```


# 通过docker compose部署Grafana和Prometheus

**目标:**

基于我们之前制作的nginx-exporter:v0.2镜像，我们使用docker compose编排文件实现一次性部署Prometheus和Grafana

与前面手工部署三个容器不一样的地方是，我们这里需要修改一下Prometheus和Grafana配置里的数据源地址。

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

新建一个docker-compose.yaml文件：

```text
version: '3'
  
services:

  nginx:
    image: nginx-exporter:v0.2
    networks:
      webnet:
        aliases:
        - "nginx_exporter"
    expose:
    - "9113"
    ports:
    - "9113:9113"

  prometheus:
    image: prom/prometheus
    volumes:
    - ./prometheus/prometheus-compose.yml:/etc/prometheus/prometheus.yml
    networks:
      webnet:
        aliases:
        - "prometheus"
    expose:
    - "9090"
    ports:
    - "9090:9090"
    depends_on:
    - nginx

  grafana:
    image: grafana/grafana
    volumes:
    - ./grafana/dashboards/dashboard.json:/var/lib/grafana/dashboards/dashboard.json
    - ./grafana/provisioning/dashboard.yaml:/etc/grafana/provisioning/dashboards/dashboard.yaml
    - ./grafana/provisioning/datasource-compose.yaml:/etc/grafana/provisioning/datasources/datasource.yaml
    networks:
      webnet:
        aliases:
        - "grafana"
    expose:
    - "3000"
    ports:
    - "3000:3000"
    depends_on:
    - prometheus

networks:
  webnet: {}
```

然后使用`docker-compose up -d`命令运行起来，之后访问宿主机3000端口即可获取grafana界面。


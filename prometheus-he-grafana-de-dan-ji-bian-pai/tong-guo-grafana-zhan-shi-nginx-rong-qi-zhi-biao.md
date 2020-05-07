# 通过Grafana展示nginx容器指标

**目标**

本测试由三部分组成：

1）Nginx指标收集，由nginx-prometheus-exporter转化为prometheus可以处理的指标格式，这部分我们上一节已经完成，本节将复用我们做好的`nginx-exporter:v0.2`镜像。

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
     - job_name: 'nginx_exporter'
       static_configs:
       - targets: ['192.168.2.31:9113']
         labels:
           group: 'services'
```

启动Prometheus容器：

```text
docker run --name prometheus1 -d -p 9090:9090 -v /simon-testing/docker/compose/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

然后访问宿主机 http://192.168.2.31:9090/targets，获得如下界面即代表Prometheus和Nginx指标均可获取：

![](../.gitbook/assets/image%20%287%29.png)

![&#x4F7F;&#x7528;Nginx&#x6307;&#x6807;&#x540D;&#x79F0;&#x80FD;&#x5728;Prometheus&#x7ED8;&#x5236;&#x51FA;&#x7B80;&#x5355;&#x56FE;&#x8868;](../.gitbook/assets/image%20%281%29.png)

## 测试2：配置Grafana对指标进行展示

**1）准备dashboard.json**

首先我们需要定制一个适合展示prometheus收集回来的nginx的指标参数的dashboards仪表板模板，最快的方式是上官网的选择现成的，参考[https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/9516)

实际工作中，我们通常使用的`nginx-vts-exporter`能获得更多的指标。

这里我们自己定义一个适合`nginx-prometheus-exporter`指标的json文件，并命名为dashboard.json：

```text
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "gnetId": null,
  "graphTooltip": 0,
  "id": 7,
  "links": [],
  "panels": [
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorPostfix": false,
      "colorPrefix": false,
      "colorValue": true,
      "colors": [
        "#299c46",
        "rgba(237, 129, 40, 0.89)",
        "#d44a3a"
      ],
      "datasource": "Prometheus",
      "format": "none",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": false,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 4,
        "w": 12,
        "x": 0,
        "y": 0
      },
      "id": 4,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "postfix": "",
      "postfixFontSize": "50%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": false,
        "ymax": null,
        "ymin": null
      },
      "tableColumn": "",
      "targets": [
        {
          "expr": "nginx_up",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        }
      ],
      "thresholds": "",
      "timeFrom": null,
      "timeShift": null,
      "title": "Nginx Status",
      "transparent": true,
      "type": "singlestat",
      "valueFontSize": "200%",
      "valueMaps": [
        {
          "op": "=",
          "text": "Down",
          "value": "0"
        },
        {
          "op": "=",
          "text": "Up",
          "value": "1"
        }
      ],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "datasource": "Prometheus",
      "gridPos": {
        "h": 4,
        "w": 12,
        "x": 12,
        "y": 0
      },
      "id": 7,
      "links": [],
      "options": {
        "fieldOptions": {
          "calcs": [
            "mean"
          ],
          "defaults": {
            "mappings": [],
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "red",
                  "value": 80
                }
              ]
            }
          },
          "overrides": [],
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "pluginVersion": "6.7.3",
      "targets": [
        {
          "expr": "nginx_http_requests_total",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        },
        {
          "refId": "B"
        },
        {
          "refId": "C"
        }
      ],
      "timeFrom": null,
      "timeShift": null,
      "title": "Nginx Total Request",
      "transparent": true,
      "type": "gauge"
    },
    {
      "aliasColors": {},
      "bars": false,
      "cacheTimeout": null,
      "dashLength": 10,
      "dashes": false,
      "datasource": "Prometheus",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 9,
        "w": 12,
        "x": 0,
        "y": 4
      },
      "hiddenSeries": false,
      "id": 2,
      "legend": {
        "alignAsTable": false,
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "rightSide": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "6.7.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "rate(nginx_connections_accepted[5m])",
          "interval": "",
          "legendFormat": "accepted_rate",
          "refId": "A"
        },
        {
          "expr": "rate(nginx_connections_handled[5m])",
          "interval": "",
          "legendFormat": "handled_rate",
          "refId": "B"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Connection Rate",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "cacheTimeout": null,
      "dashLength": 10,
      "dashes": false,
      "datasource": "Prometheus",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 9,
        "w": 12,
        "x": 12,
        "y": 4
      },
      "hiddenSeries": false,
      "id": 6,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "6.7.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "nginx_connections_reading",
          "interval": "",
          "legendFormat": "conn_reading",
          "refId": "A"
        },
        {
          "expr": "nginx_connections_waiting",
          "interval": "",
          "legendFormat": "conn_waiting",
          "refId": "B"
        },
        {
          "expr": "nginx_connections_writing",
          "interval": "",
          "legendFormat": "conn_writing",
          "refId": "C"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Connection Status",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    }
  ],
  "schemaVersion": 22,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-30m",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "5s",
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ]
  },
  "timezone": "",
  "title": "nginx-simon",
  "uid": "b2yin83Zz",
  "variables": {
    "list": []
  },
  "version": 3
}
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

2-2）定义数据源

创建一个datasouce.yaml文件：

```text
apiVersion: 1

datasources:
- name: Prometheus
  type: prometheus
  access: proxy
  orgId: 1
  url: http://192.168.2.31:9090
  basicAuth: false
  isDefault: true
  version: 1
  editable: true
```

**3）启动Grafana容器**

```text
docker run -d --name=grafana1 \
-v /simon-testing/docker/compose/grafana/dashboards/dashboard.json:/var/lib/grafana/dashboards/dashboard.json \
-v /simon-testing/docker/compose/grafana/provisioning/dashboard.yaml:/etc/grafana/provisioning/dashboards/dashboard.yaml \
-v /simon-testing/docker/compose/grafana/provisioning/datasource.yaml:/etc/grafana/provisioning/datasources/datasource.yaml \
-p 3000:3000 grafana/grafana
```

现在宿主机上三个容器是：

```text
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                            NAMES
04b5ee1a4128        grafana/grafana       "/run.sh"                3 seconds ago       Up 2 seconds        0.0.0.0:3000->3000/tcp           grafana1
556ca4ec29f8        prom/prometheus       "/bin/prometheus --c…"   12 hours ago        Up 12 hours         0.0.0.0:9090->9090/tcp           prometheus1
71dda6ff4612        nginx-exporter:v0.2   "/bin/sh /run.sh"        12 hours ago        Up 12 hours         80/tcp, 0.0.0.0:9113->9113/tcp   exporter2
```

访问宿主机3000端口，打开Grafana，正常可以看到如下仪表板：

![](../.gitbook/assets/image%20%286%29.png)


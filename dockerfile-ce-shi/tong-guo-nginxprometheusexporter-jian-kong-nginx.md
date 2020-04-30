---
description: 'https://hub.docker.com/r/nginx/nginx-prometheus-exporter'
---

# 通过nginx-prometheus-exporter监控nginx

[NGINX](http://nginx.org/) exposes a handful of metrics via the [stub\_status page](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status). [NGINX Plus](https://www.nginx.com/products/nginx/) provides a richer set of metrics via the [API](https://nginx.org/en/docs/http/ngx_http_api_module.html) and the [monitoring dashboard](https://www.nginx.com/products/nginx/live-activity-monitoring/). NGINX Prometheus exporter fetches the metrics from a single NGINX or NGINX Plus, converts the metrics into appropriate Prometheus metrics types and finally exposes them via an HTTP server to be collected by [Prometheus](https://prometheus.io/).

对于nginx-prometheus-exporter说明，请见这里：[https://github.com/nginxinc/nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter)

测试方式，通过多阶段部署，为nginx:alpine镜像合入nginx/nginx-prometheus-exporter中的程序，并使用脚本同时启动二者，注意让nginx运行于前台，且nginx-prometheus-exporter程序能正常输出nginx的指标。

docker pull nginx:stable-alpine

docker pull nginx/nginx-prometheus-exporter:latest








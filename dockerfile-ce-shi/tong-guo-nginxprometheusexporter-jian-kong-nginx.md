---
description: 'https://hub.docker.com/r/nginx/nginx-prometheus-exporter'
---

# 通过nginx-prometheus-exporter监控nginx

[NGINX](http://nginx.org/) exposes a handful of metrics via the [stub\_status page](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status). [NGINX Plus](https://www.nginx.com/products/nginx/) provides a richer set of metrics via the [API](https://nginx.org/en/docs/http/ngx_http_api_module.html) and the [monitoring dashboard](https://www.nginx.com/products/nginx/live-activity-monitoring/). NGINX Prometheus exporter fetches the metrics from a single NGINX or NGINX Plus, converts the metrics into appropriate Prometheus metrics types and finally exposes them via an HTTP server to be collected by [Prometheus](https://prometheus.io/).

对于nginx-prometheus-exporter说明，请见这里：[https://github.com/nginxinc/nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter)

测试方式，通过多阶段部署，为nginx:alpine镜像合入nginx/nginx-prometheus-exporter中的程序，并使用脚本同时启动二者，注意让nginx运行于前台，且nginx-prometheus-exporter程序能正常输出nginx的指标。

docker pull nginx:alpine

docker pull nginx/nginx-prometheus-exporter:latest

docker run --name nginx1 -d -p 8080 nginx:alpine

docker exec -it nginx1 /bin/sh

/ \# /usr/sbin/nginx -V nginx version: nginx/1.17.10 built by gcc 9.2.0 \(Alpine 9.2.0\) built with OpenSSL 1.1.1d 10 Sep 2019 \(running with OpenSSL 1.1.1g 21 Apr 2020\) TLS SNI support enabled configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client\_temp --http-proxy-temp-path=/var/cache/nginx/proxy\_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi\_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi\_temp --http-scgi-temp-path=/var/cache/nginx/scgi\_temp --with-perl\_modules\_path=/usr/lib/perl5/vendor\_perl --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http\_addition\_module --with-http\_auth\_request\_module --with-http\_dav\_module --with-http\_flv\_module --with-http\_gunzip\_module --with-http\_gzip\_static\_module --with-http\_mp4\_module --with-http\_random\_index\_module --with-http\_realip\_module --with-http\_secure\_link\_module --with-http\_slice\_module --with-http\_ssl\_module --with-http\_stub\_status\_module --with-http\_sub\_module --with-http\_v2\_module --with-mail --with-mail\_ssl\_module --with-stream --with-stream\_realip\_module --with-stream\_ssl\_module --with-stream\_ssl\_preread\_module --with-cc-opt='-Os -fomit-frame-pointer' --with-ld-opt=-Wl,--as-needed






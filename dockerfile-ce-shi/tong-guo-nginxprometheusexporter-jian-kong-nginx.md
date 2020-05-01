---
description: 'https://hub.docker.com/r/nginx/nginx-prometheus-exporter'
---

# 通过nginx-prometheus-exporter监控nginx

[NGINX](http://nginx.org/) exposes a handful of metrics via the [stub\_status page](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status). [NGINX Plus](https://www.nginx.com/products/nginx/) provides a richer set of metrics via the [API](https://nginx.org/en/docs/http/ngx_http_api_module.html) and the [monitoring dashboard](https://www.nginx.com/products/nginx/live-activity-monitoring/). NGINX Prometheus exporter fetches the metrics from a single NGINX or NGINX Plus, converts the metrics into appropriate Prometheus metrics types and finally exposes them via an HTTP server to be collected by [Prometheus](https://prometheus.io/).

对于nginx-prometheus-exporter说明，请见这里：[https://github.com/nginxinc/nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter)

测试方式，通过多阶段部署，为nginx:alpine镜像合入nginx/nginx-prometheus-exporter中的程序，并使用脚本同时启动二者，注意让nginx运行于前台，且nginx-prometheus-exporter程序能正常输出nginx的指标。

下载测试到用到镜像：

`docker pull nginx:alpine`

`docker pull nginx/nginx-prometheus-exporter:latest`

然后运行一下nginx:alpine容器，看看里面是否包含 stub\_status 模块

`docker run --name nginx1 -d nginx:alpine`

`docker exec -it nginx1 /bin/sh`

```text
/ # /usr/sbin/nginx -V 
nginx version: nginx/1.17.10 built by gcc 9.2.0 (Alpine 9.2.0) built with OpenSSL 1.1.1d 10 Sep 2019 (running with OpenSSL 1.1.1g 21 Apr 2020) TLS SNI support enabled configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --with-perl_modules_path=/usr/lib/perl5/vendor_perl --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-Os -fomit-frame-pointer' --with-ld-opt=-Wl,--as-needed
```

看到`--with-http_stub_status_module`，证明这个镜像已经编译好了stub\_status 模块

编辑 `/etc/nginx/conf.d/default.conf`，加入：

```text
    location /nginx-status {                
       stub_status on;                      
       access_log off;                                        
    }     
```

`/usr/sbin/nginx -s reload`重启，然后在新terminal访问容器ip/stub\_status页面：

```text
root@31:/home/ubuntu# curl 172.17.0.2/nginx-status
Active connections: 1 
server accepts handled requests
 3 3 3 
Reading: 0 Writing: 1 Waiting: 0 
```

利用while语句做一个循环输出就可以实现简单的访问量监控：

```text
while true ; do curl 172.17.0.2/nginx-status; sleep 2; done
```




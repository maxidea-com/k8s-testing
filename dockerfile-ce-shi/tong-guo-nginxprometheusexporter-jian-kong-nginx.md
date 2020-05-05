---
description: 'https://hub.docker.com/r/nginx/nginx-prometheus-exporter'
---

# 通过nginx-prometheus-exporter监控nginx指标

[NGINX](http://nginx.org/) 通过 [stub\_status page](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status) 页面公开了一些有用的指标，NGINX Prometheus exporter 则通过获取这些指标，转换成适合Prometheus处理的指标格式，并通过HTTP服务输出，方便我们使用 [Prometheus](https://prometheus.io/) 进行收集和处理。

对于nginx-prometheus-exporter说明，请见这里：[https://github.com/nginxinc/nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter)

**测试方式，通过多阶段部署，把nginx:alpine镜像合入nginx/nginx-prometheus-exporter中的程序，并使用脚本同时启动二者，注意让nginx运行于前台，且nginx-prometheus-exporter程序能正常输出nginx的指标。**

## 步骤1：

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

然后把`/etc/nginx/conf.d/default.conf`复制到宿主机备用：

```text
root@31:/simon-testing/docker/compose/multistage/nginx# docker cp nginx1:/etc/nginx/conf.d/default.conf ./
```

## 步骤2：

通过nginx-prometheus-exporter容器来监控nginx1的指标，命令如下：

```text
root@31:/home/ubuntu# docker run -p 9113:9113 nginx/nginx-prometheus-exporter:latest -nginx.scrape-uri http://172.17.0.2/nginx-status                  
2020/05/01 11:31:38 Starting NGINX Prometheus Exporter Version=0.7.0 GitCommit=a2910f1
2020/05/01 11:31:38 Listening on :9113
2020/05/01 11:31:38 NGINX Prometheus Exporter has successfully started
```

然后在另一个terminal下每秒访问一次nginx1容器的80服务：

```text
while true ; do curl -s -o /dev/null http://172.17.0.2/; sleep 1; done
```

最后访问宿主机的IP:9113/metrics页面，就能看到读取出来的nginx指标数据了。

![](../.gitbook/assets/image%20%287%29.png)



## 步骤3：通过多阶段部署把两个步骤整个到一个镜像里

创建一个Dockerfile

```text
# 编译阶段 命名为 base
FROM nginx/nginx-prometheus-exporter:latest as base
# 运行阶段
FROM nginx:alpine
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=base /usr/bin/exporter /usr/bin/exporter
ADD run.sh /run.sh
RUN chmod 775 /run.sh
EXPOSE 80 9113
CMD [["/bin/sh", "/run.sh"]]
```

编写run.sh

```text
#!/bin/bash
nginx -c /etc/nginx/nginx.conf
nginx -s reload
/usr/bin/exporter -nginx.scrape-uri http://127.0.0.1/nginx-status
tail -f /dev/null #实现本shell永不运行完成，容器不退出。
```

把dockerfile生成镜像文件：

```text
docker build . -t nginx-exporter:v0.1
```

运行容器：

```text
docker run --name exporter1 -d -p 9113:9113 nginx-exporter:v0.1
```

访问`127.0.0.1:9113/metrics`检查exporter运作情况：

```text
curl 127.0.0.1:9113/metrics
# HELP nginx_connections_accepted Accepted client connections
# TYPE nginx_connections_accepted counter
nginx_connections_accepted 2
# HELP nginx_connections_active Active client connections
# TYPE nginx_connections_active gauge
nginx_connections_active 1
# HELP nginx_connections_handled Handled client connections
# TYPE nginx_connections_handled counter
nginx_connections_handled 2
# HELP nginx_connections_reading Connections where NGINX is reading the request header
# TYPE nginx_connections_reading gauge
nginx_connections_reading 0
# HELP nginx_connections_waiting Idle client connections
# TYPE nginx_connections_waiting gauge
nginx_connections_waiting 0
# HELP nginx_connections_writing Connections where NGINX is writing the response back to the client
# TYPE nginx_connections_writing gauge
nginx_connections_writing 1
# HELP nginx_http_requests_total Total http requests
# TYPE nginx_http_requests_total counter
nginx_http_requests_total 2
# HELP nginx_up Status of the last metric scrape
# TYPE nginx_up gauge
nginx_up 1
# HELP nginxexporter_build_info Exporter build information
# TYPE nginxexporter_build_info gauge
nginxexporter_build_info{gitCommit="a2910f1",version="0.7.0"} 1
```

参考资料：[https://github.com/nginxinc/nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter)


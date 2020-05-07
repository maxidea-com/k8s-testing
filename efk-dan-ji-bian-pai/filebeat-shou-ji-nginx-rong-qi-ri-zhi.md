---
description: 'https://www.elastic.co/guide/en/beats/filebeat/'
---

# Filebeat收集nginx容器日志

Filebeat是一个用于转发和集中日志数据的轻量级传送程序。作为代理安装在服务器上，Filebeat监视指定的日志文件或位置，收集日志事件，并将它们转发到Elasticsearch或Logstash进行索引。

![Filebeat&#x7684;&#x5DE5;&#x4F5C;&#x539F;&#x7406;](../.gitbook/assets/image%20%285%29.png)

由于官方docker.elastic.co的国内访问速度很慢，所以我们直接用已经有加速的dockhub仓库地址

拉取Filebeat镜像文件

`docker pull store/elastic/filebeat:7.6.2`

下载官方的配置文件进行修改

```text
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.6/deploy/docker/filebeat.docker.yml
```

文件内容：

```text
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

filebeat.autodiscover:
  providers:
    - type: docker
      hints.enabled: true

processors:
- add_cloud_metadata: ~

output.elasticsearch:
  hosts: '${ELASTICSEARCH_HOSTS:elasticsearch:9200}'
  username: '${ELASTICSEARCH_USERNAME:}'
  password: '${ELASTICSEARCH_PASSWORD:}'
```

把`filebeat.docker.yml`文件配置为基于应用于容器的`docker label`来部署Beats模块，filebeat就能使用自动发现功能，这里我们设置nginx容器的label为`co.elastic.logs/enabled`

```text
docker run \
  --label co.elastic.logs/enable=true \
  --label co.elastic.logs/module=nginx \
  --label co.elastic.logs/fileset.stdout=access \
  --label co.elastic.logs/fileset.stderr=error \
  --name nginx-app \
  -p 80:80 -d \
  nginx:alpine
```




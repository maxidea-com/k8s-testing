---
description: 'https://www.elastic.co/guide/en/beats/filebeat/'
---

# Filebeat收集nginx容器日志并同步到Elastic Cloud

Filebeat是一个用于转发和集中日志数据的轻量级传送程序。作为代理安装在服务器上，Filebeat监视指定的日志文件或位置，收集日志事件，并将它们转发到Elasticsearch或Logstash进行索引。

![Filebeat&#x7684;&#x5DE5;&#x4F5C;&#x539F;&#x7406;](../.gitbook/assets/image%20%2811%29.png)

注意：由于官方docker.elastic.co的国内访问速度很慢，所以我们直接用已经有加速的dockerhub仓库地址。

拉取Filebeat镜像文件（dockerhub官方镜像版本，我们在第一章里用了阿里云镜像对其进行加速）

`docker pull store/elastic/filebeat:7.6.2`

下载官方的配置文件：

```text
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.6/deploy/docker/filebeat.docker.yml
```

filebeat.docker.yml文件内容：

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

`filebeat.docker.yml`文件配置默认为基于应用于容器的`docker label`来部署Beats模块，filebeat就能使用自动发现功能，所以我们启动nginx容器时需要加上label：

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

启动filebeat容器：

```text
docker run -d \
  --name=filebeat \
  --user=root \
  --volume="$(pwd)/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml:ro" \
  --volume="/var/lib/docker/containers:/var/lib/docker/containers:ro" \
  --volume="/var/run/docker.sock:/var/run/docker.sock:ro" \
  store/elastic/filebeat:7.6.2 filebeat -e -strict.perms=false \
  -E cloud.id=<Cloud ID from Elasticsearch Service> \
  -E cloud.auth=elastic:<elastic password>
```

这里由于我们还没有本地配置ElasticSearch，所以把参数`-E output.elasticsearch.hosts=["elasticsearch:9200"]`改掉，暂时使用Elastic Cloud提供的Elasticsearch Service的代替，上面的用户名和密码请以你自己的Elastic Cloud代替。例如：

```text
docker run -d \
   --name=filebeat \
   --user=root \
   --volume="$(pwd)/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml:ro" \
   --volume="/var/lib/docker/containers:/var/lib/docker/containers:ro" \
   --volume="/var/run/docker.sock:/var/run/docker.sock:ro" \
   store/elastic/filebeat:7.6.2 filebeat -e -strict.perms=false \
   -E cloud.id=nginx-simon1:c291dGhlYXN0YXNpYS5henVyZS5lbGFzdGljLWNsb3VkLmNvbTo5MjQzJDgzMjZkYTcyMDRkNDQzODA4YzkyYjMzOTEwNzc0MjIzJGZlYzAyYzQ3NDI0NDQ4YjM5MWRlNGI5MjI5NTY2MThk \
   -E cloud.auth=elastic:HT0QWmB7yvJK4xjLgIs59Ruo
```

确保nginx和filebeat两个容器都启动后，我们刷一下本地80端口访问量，然后打开Elastic Cloud上的Kibana--&gt;Logs，能获取到nginx访问日志即代表配置正确：

![](../.gitbook/assets/image%20%286%29.png)



## 补充：Elastic Cloud设置和获取Cloud ID/password的方法

Elastic Cloud是收费的SAAS服务，[https://www.elastic.co/](https://www.elastic.co/)英文官方网站上提供免费的14天测试。

1）激活账号后登录[https://cloud.elastic.co/home](https://cloud.elastic.co/home)

2）点击“Start your free trail”

3）填写“Create deployment”表单里的内容，例如：Select a cloud platform选择Azure，Select a region选择Southeast Asia \(Singapore\)，其他保持默认。

4）点击Deploy后云端会自动开始准备环境，页面上会提示Save your Elasticsearch and Kibana password。大约三分钟后可以使用。

![](../.gitbook/assets/image%20%2822%29.png)

5）点击左上角Deployment的名字，在页面上获取对应的cloud ID

![](../.gitbook/assets/image%20%283%29.png)




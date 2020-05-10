# 通过docker compose部署EFK

使用到的镜像：

`docker pull store/elastic/filebeat:7.6.2`

`docker pull elasticsearch:7.6.2`

`docker pull kibana:7.6.2`

为了更好理解docker编排文件下filebeat和kibana的配置方法，这里的Elasticsearch使用双节点部署。实际生产环境不会这样配置Elasticsearch集群。（官方的集群样例，请参考这里：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)）

Docker Compose文件里创建两个节点的Elasticsearch集群。节点es01在本地主机上侦听9200端口，es02通过Docker网络与es01对话。

volumes data01和data02是的Docker存储节点数据目录，以便数据在重新启动时保持不变。如果目录不存在，docker compose会在您启动集群时创建它们（如果不指定位置，默认位于/var/lib/docker/volumes/目录下）。

docker-compose.yml文件如下：

```text
version: '3'
services:
  es01:
    image: elasticsearch:7.6.2
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02
      - cluster.initial_master_nodes=es01,es02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: elasticsearch:7.6.2
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01
      - cluster.initial_master_nodes=es01,es02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic
  filebeat:
    image: store/elastic/filebeat:7.6.2
    container_name: filebeat
    user: root
    environment:
      - strict.perms=false
      - output.elasticsearch.hosts=["es1:9200"]
    volumes:
      - ./filebeat.docker-compose.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - elastic
    depends_on:
      - es01  
  kibana:
    image: kibana:7.6.2
    container_name: kibana
    environment:
      ELASTICSEARCH_HOSTS: http://es01:9200
    networks:
      - elastic
    expose:
      - "5601"
    ports:
      - "5601:5601"
    depends_on:
      - es01

volumes:
  data01:
    driver: local
  data02:
    driver: local

networks:
  elastic:
    driver: bridge
```

filebeat.docker-compose.yml文件内容：

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
  hosts: '${ELASTICSEARCH_HOSTS:es01:9200}'
```

常见问题：

1）vm.max\_map\_count设置问题

```text
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

解决办法：

这是由于默认vm.max\_map\_count=65530，因此缺省配置下，单个jvm能开启的最大线程数为其一半，即3w左右，大概32k的量。所以要调大到262144。（确保宿主机内存大于4G的情况下）

`echo vm.max_map_count=262144 >> /etc/sysctl.conf && sysctl -p`

检查配置是否生效：

```text
#cat /proc/sys/vm/max_map_count
262144
```

2）Data volume数据清除

Docker compose文件内配置，当容器启动后，会在/var/lib/docker/volumes/filebeat\_data01/\_data/目录下生成数据并持久化，在每次启动集群时都会复用。如果要删除这些数据，我们需要在停止集群时加上-v参数： `docker-compose down -v`.

```text
# docker-compose down -v
Stopping es02 ... done
Stopping es01 ... done
Removing es02 ... done
Removing es01 ... done
Removing network filebeat_elastic
Removing volume filebeat_data02
Removing volume filebeat_data01
```

3）手动创建的Kibana如何加入集群？

如果不使用docker compose同时创建的kibana，加入Elasticsearch集群时，需要指定集群的网络名，以及连接集群中的Elasticsearch节点名字，例如：

```text
docker run -d --name kibana-cluster --net filebeat_elastic --link es01:elasticsearch -p 5601:5601 kibana:7.6.2
```






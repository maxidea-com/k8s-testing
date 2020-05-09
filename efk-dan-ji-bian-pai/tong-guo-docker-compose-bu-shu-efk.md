# 通过docker compose部署EFK

使用到的镜像：

`docker pull store/elastic/filebeat:7.6.2`

`docker pull elasticsearch:7.6.2`

`docker pull kibana:7.6.2`

这里提高一下难度，ElasticSearch使用双节点部署。

Docker Compose文件里将创建两个节点的Elasticsearch集群。节点es01在本地主机上侦听9200端口，es02通过Docker网络与es01对话。

volumes data01和data02是的Docker存储节点数据目录，以便数据在重新启动时保持不变。如果它们不存在，docker compose会在您启动集群时创建它们（如果不指定位置，默认位于/var/lib/docker/volumes/目录下）。







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






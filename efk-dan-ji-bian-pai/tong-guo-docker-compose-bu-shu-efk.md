# 通过docker compose部署EFK

使用到的镜像：

`docker pull store/elastic/filebeat:7.6.2`

`docker pull elasticsearch:7.6.2`

`docker pull kibana:7.6.2`

这里提高一下难度，ElasticSearch使用双节点部署。

Docker Compose文件里将创建两个节点的Elasticsearch集群。节点es01在本地主机上侦听9200端口，es02通过Docker网络与es01对话。




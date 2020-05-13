# 通过docker部署Elasticsearch并定制Kibana的Dashboard

**目标：**

**使用单节点的Elasticsearch+Kibana，替代上一节使用的Elastic Cloud Service. 并使用Kibana定制一个Dashboard。**

使用到的镜像：

`docker pull elasticsearch:7.6.2`

`docker pull kibana:7.6.2`

**1）运行Elasticsearch单节点容器：**

```text
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2
```

其中，9200端口 是ES节点与外部通讯使用的端口，它是http协议的RESTful接口，各种CRUD操作都是走的该端口。

9300端口是ES节点之间通讯使用的端口，它是tcp协议通讯，集群间和TCPclient都走的它，jar之间就是通过tcp协议通讯。

**2）运行一个新的Filebeat容器，output指定本地9200端口：**

```text
docker run -d \
  --name=filebeat \
  --user=root \
  --volume="$(pwd)/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml:ro" \
  --volume="/var/lib/docker/containers:/var/lib/docker/containers:ro" \
  --volume="/var/run/docker.sock:/var/run/docker.sock:ro" \
  store/elastic/filebeat:7.6.2 filebeat -e -strict.perms=false \
  -E output.elasticsearch.hosts=["192.168.2.31:9200"]
```

其中filebeat.docker.yml文件沿用我们上一节默认配置即可。

**3）检查一下Elasticsearch节点的运行情况：**

```text
#curl -X GET "localhost:9200/_cat/nodes?v&pretty"
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.17.0.3           19          97   0    0.04    0.16     0.08 dilm      *      628769d8630c
```

**4）启动Kibana容器：**

```text
docker run -d --name kibana --link elasticsearch:elasticsearch -p 5601:5601 kibana:7.6.2
```

--link参数后面是Elastisearch节点的容器名或ID。

**5）设置Kibana的仪表板**

5-1）访问宿主机上的kibana（`http://宿主机ip:5601`），首次访问的话，选择INDEX PATTERN为filebeat-7.6.2-{日期}-000001，然后切换到logs页面下，即可获得nginx日志

![](../.gitbook/assets/image%20%2822%29.png)

5-2）然后打开Visualize，创建一个基于Metrics-&gt;Count的计数器，点击Save保存为count。

![](../.gitbook/assets/image%20%2811%29.png)

5-3）再次选择Visualize，创建一个基于data table状态码排行（Buckets-&gt;Split rows,  Aggregation-&gt; Terms, Field-&gt;http.respone.status\_code\) 并保存为status code

![](../.gitbook/assets/image%20%2817%29.png)

5-4）再次选择Visualize，创建一个基于data table的请求来源地址排行（Buckets-&gt;Split rows,  Aggregation-&gt; Terms, Field-&gt;http.request.referrer\) 并保存为referrer。

![](../.gitbook/assets/image%20%283%29.png)

5-5）再次选择Visualize，创建一个基于Gauge的nginx访问来源地址nginx.access.remote\_ip\_list

![](../.gitbook/assets/image%20%281%29.png)

5-6）选择Dashboard，Create new dashboard，然后把前面准备好的几组Visualize添加进去即可。

![](../.gitbook/assets/image%20%286%29.png)

小技巧，如果要换成黑色主题，在Kibana的高级设置里把Dark mode打开即可。

![](../.gitbook/assets/image%20%2825%29.png)


---
description: 'https://hub.docker.com/_/nginx'
---

# 容器网络入门操作2

## 测试2：

拉取一个nginx镜像文件，这里使用的是最小化的版本

`docker pull nginx:alpine`

然后启动nginx容器：

```text
docker run --name nginx1 -d --net bridge nginx:alpine
```

然后通过`docker inspect`查看nginx1这个容器的ip地址：

```text
root@09-1:/data/mysql3# docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nginx1
172.17.0.3
```

在宿主机上使用 curl 172.17.0.3 即可访问nginx容器的主页：

```text
root@09-1:/data/mysql3# curl 172.17.0.3
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

但是这个nginx在宿主机以外是无法访问的，关于docker内置支持的四种网络类型，可以参考这张图：

![docker&#x5185;&#x7F6E;&#x652F;&#x6301;&#x7684;&#x56DB;&#x79CD;&#x7F51;&#x7EDC;&#x7C7B;&#x578B;](../.gitbook/assets/image%20%289%29.png)

所以，要实现跨宿主机的网络访问，首先要把容器的端口暴露在宿主机的网络接口上，原理与路由器上的NAT类似。通过增加参数 `-p [宿主端口]:[容器端口]` 实现，例如：

```text
docker run --name nginx2 -d --net bridge -p 10080:80 nginx:alpine
```

`docker ps`现在可以看到端口转发的情况：

```text
root@09-1:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
925878944cd8        nginx:alpine        "nginx -g 'daemon of…"   4 seconds ago       Up 3 seconds        0.0.0.0:10080->80/tcp   nginx2
9f2bd3b88b8e        nginx:alpine        "nginx -g 'daemon of…"   54 minutes ago      Up 54 minutes       80/tcp                  nginx1
5104ca874b9e        mysql:5.7           "docker-entrypoint.s…"   2 hours ago         Up 2 hours          3306/tcp, 33060/tcp     mysql4
```

也可以使用`docker port`查看：

```text
root@09-1:~# docker port nginx2
80/tcp -> 0.0.0.0:10080
```

使用`iptables -t nat -vnL`可以看到对应的NAT规则：

```text
root@09-1:~# iptables -t nat -vnL
Chain PREROUTING (policy ACCEPT 1 packets, 169 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    7  1201 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 1 packets, 169 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 2 packets, 146 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 2 packets, 146 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.4           172.17.0.4           tcp dpt:80

Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
    0     0 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:10080 to:172.17.0.4:80
```

这样，通过其他主机，访问这台宿主机的`http://ip:10080`端口，就能访问到nginx2容器内的80端口，获取nginx主页信息。


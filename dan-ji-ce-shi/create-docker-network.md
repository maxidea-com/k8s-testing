# 创建docker network

为了使用自定义网络，或者使一个容器有多个网络接口，可以使用`docker network`命令：

```text
sudo docker network create -d bridge --subnet 10.10.0.0/24 net1
```

然后在主机上会生成一个新的网络接口：

```text
ubuntu@09-1:~$ sudo docker inspect net1
[
    {
        "Name": "net1",
        "Id": "ba6e4cae74208858d2bfafbe52da00c8b8177b5948fb027fa990861f8060ae97",
        "Created": "2020-04-24T18:18:14.621826632+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.10.0.0/24"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
ubuntu@09-1:~$ ifconfig 
br-ba6e4cae7420: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.10.0.1  netmask 255.255.255.0  broadcast 10.10.0.255
        ether 02:42:ef:de:5d:93  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

使用docker network ls查看：

```text
ubuntu@09-1:~$ sudo docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
9045dd7942c2        bridge              bridge              local
e3630a6f8b64        host                host                local
ba6e4cae7420        net1                bridge              local
```

### 测试：把容器加入这个网络并增加一个域名解释

```text
docker run --name testbox -it --net net1 --add-host "testbox.maxidea.com:10.10.0.2" busybox /bin/sh

/ # more /etc/hosts 
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
10.10.0.2       testbox.maxidea.com
10.10.0.2       0c8fdfe89b4f

/ # ping testbox.maxidea.com
PING testbox.maxidea.com (10.10.0.2): 56 data bytes
64 bytes from 10.10.0.2: seq=0 ttl=64 time=0.061 ms
64 bytes from 10.10.0.2: seq=1 ttl=64 time=0.032 ms
64 bytes from 10.10.0.2: seq=2 ttl=64 time=0.030 ms
```


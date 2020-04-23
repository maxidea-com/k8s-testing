# 服务器间镜像文件快速复制

网络中有多台主机需要同一镜像文件，而该文件又比较大的时候，逐一拉取是比较麻烦的事情，通过`docker save`打包镜像再通过`scp`传到其他主机是比较方便的，测试如下：

例如mysql镜像文件

```text
mysql                   latest              8e8c6f8dc9df        4 days ago          546MB
```

使用命令打包：

`sudo docker save mysql:latest -o myimg.tar`

打包出来的文件通过`scp`传到另一台主机：

`sudo scp myimg.tar ubuntu@192.168.2.26:/tmp/`

在另一台主机上执行`docker load`还原镜像：

```text
ubuntu@09-2:/tmp$ sudo docker load -i myimg.tar 
b60e5c3bcef2: Loading layer [==================================================>]  72.49MB/72.49MB
746ef614d661: Loading layer [==================================================>]  338.4kB/338.4kB
478bf6a73d06: Loading layer [==================================================>]  9.539MB/9.539MB
246ad53299e0: Loading layer [==================================================>]    4.2MB/4.2MB
f2b1703888ed: Loading layer [==================================================>]  1.536kB/1.536kB
c73d9f519696: Loading layer [==================================================>]  53.75MB/53.75MB
c4a52d4531b7: Loading layer [==================================================>]  6.656kB/6.656kB
53e783b27a6d: Loading layer [==================================================>]  3.584kB/3.584kB
ef2a52de3c1a: Loading layer [==================================================>]  411.2MB/411.2MB
73734f098425: Loading layer [==================================================>]  5.632kB/5.632kB
ace58d0dd227: Loading layer [==================================================>]  16.38kB/16.38kB
fe80e859fd88: Loading layer [==================================================>]  1.536kB/1.536kB
Loaded image: mysql:latest
```

另外，还可以用`docker export/import`对**容器**进行导出打包和重命名：

```text
ubuntu@09-1:~$ sudo docker run --name bbs -d maxidea/bbsimon1
1b75fbc1253fb74ca0222b252a8c752db7856b9f94b56e015eae8fcacb23d0a3
ubuntu@09-1:~$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
1b75fbc1253f        maxidea/bbsimon1    "sh"                3 seconds ago       Exited (0) 3 seconds ago                       bbs
ubuntu@09-1:~$ sudo docker export -o bbs.tar bbs
ubuntu@09-1:~$ sudo scp bbs.tar ubuntu@192.168.2.26:/tmp/
ubuntu@192.168.2.26's password: 
bbs.tar                                                                                                                100% 1410KB 155.9MB/s   00:00  
```

导入：

```text
ubuntu@09-2:/tmp$ sudo docker import bbs.tar maxidea/bbsimon1:imp_fr_09-1
sha256:e2d268936d7ecc158081fe572bb6058134d386baa7775e2c8d3b0aa9cc61a196
ubuntu@09-2:/tmp$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
maxidea/bbsimon1    imp_fr_09-1         e2d268936d7e        19 seconds ago      1.22MB
```

这里可以看出，使用`docker import`导入的镜像文件，可以进行重命名，而使用`docker load`的则不行。




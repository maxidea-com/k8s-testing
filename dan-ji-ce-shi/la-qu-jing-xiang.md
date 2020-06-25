---
description: 了解docker命令的几个基本操作方法
---

# 入门操作测试1

## 测试1：

拉取busybox（嵌入式linux中的瑞士军刀）：

`sudo docker pull busybox`

查看本地的镜像文件：

`sudo docker images`

查看镜像构建的详细信息：

`sudo docker image inspect busybox:latest`

运行这个容器（加-it是产生交换界面和新的terminal）：

`sudo docker run --name bbox -it busybox`

退出容器exit或ctrl+d，注意如果要退出后容器继续运行，则需要用组合键：**Ctrl+P+Q**

删除这个已经停止运行的容器：

`sudo docker rm bbox`

查看运行中的容器：

`sudo docker ps -a`

```text
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
9816af1f0b2c        busybox             "sh"                9 seconds ago       Exited (0) 5 seconds ago                       bbox2
e9da490d3eda        busybox             "sh"                3 minutes ago       Up 3 minutes                                   bbox
```

这里bbox2就是直接exit后的容器，已经变成Exited状态，已经停止了。

如果要重新进入这个容器

`sudo docker attach 9` （这里可以直接用容器ID的首个数字）

terminal会返回错误信息：`You cannot attach to a stopped container, start it first`

要重新把这个容器运行起来，需要命令：

`sudo docker start 9`

**实际工作中，为避免操作错误导致容器停止运作，一般进入容器都使用命令 docker exec，例如**：

`sudo docker exec -it bbox /bin/sh`

这样exit后容器不会停止运行。

查看容器内的进程（遇到容器内没有ps命令时特有用）：

`sudo docker top 容器ID`

查看容器资料占用情况：

`sudo docker stats 容器ID --no-stream`

\[CONTAINER\]：以短格式显示容器的 ID。

\[CPU %\]：CPU 的使用情况。

\[MEM USAGE / LIMIT\]：当前使用的内存和最大可以使用的内存。

\[MEM %\]：以百分比的形式显示内存使用情况。

\[NET I/O\]：网络 I/O 数据。

\[BLOCK I/O\]：磁盘 I/O 数据。

\[PIDS\]：PID 号。


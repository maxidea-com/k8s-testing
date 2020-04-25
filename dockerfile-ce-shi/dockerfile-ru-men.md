---
description: '指令参考：https://docs.docker.com/engine/reference/builder/'
---

# Dockerfile入门

**我们先理解一下什么是Dockerfile：**

> ockerfile 是软件的原材料，Docker 镜像是软件的交付品，而 Docker 容器则可以认为是软件的运行态。从应用软件的角度来看，Dockerfile、Docker 镜像与 Docker 容器分别代表软件的三个不同阶段，Dockerfile 面向开发，Docker 镜像成为交付标准，Docker 容器则涉及部署与运维，三者缺一不可，合力充当 Docker 体系的基石。

Dockerfile 中的每一条命令，都在 Docker 镜像中以一个独立镜像层的形式存在

![](../.gitbook/assets/image%20%284%29.png)

创建一个运行python的简单dockerfile：

```text
FROM alpine:3.11

RUN apk update && apk --no-cache add python3 curl bind-tools iproute2 iptables ipvsadm
RUN pip3 install --no-cache-dir --upgrade pip && \
  pip3 install --no-cache-dir -q Flask==0.12.4 requests==2.21.0

ADD demo.py /usr/local/bin/

ENV DEPLOYENV='Production' RELEASE='Stable' PS1="[\u@\h \w]\\$ "

EXPOSE 80

VOLUME ["/simon-testing/dockerfile/data"]

ENTRYPOINT python3 /usr/local/bin/demo.py
```













参考资料：[https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)






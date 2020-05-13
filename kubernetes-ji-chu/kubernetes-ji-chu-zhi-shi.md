---
description: 'https://kubernetes.io/docs/concepts/overview/components/'
---

# Kubernetes基础知识

Kubernetes（发音 \[kubə'netis\]）简称K8s，它是一个能够跨主机进行容器编排的平台。它使用共享网络把多个主机构建成一个集群，一个或多个主机运行为主节点（Master），余下的所有主机运行为工作节点（Worker Node）。其中Master作为控制中心负责管理整个集群系统，Node接收请求并以Pod（容器集）形式运行工作负载。每个集群里最少需要有一个工作节点。

![](../.gitbook/assets/image%20%282%29.png)




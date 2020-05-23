# K8s基本操作命令

## 一、namespace操作

### 1）列出集群中所有API组名/版本号：

`kubectl api-versions`

输出：

```text
admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
autoscaling/v2beta2
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
coordination.k8s.io/v1
coordination.k8s.io/v1beta1
discovery.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
networking.k8s.io/v1beta1
node.k8s.io/v1beta1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1
scheduling.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
```

### 2）列出集群名称空间：

`kubectl get namespaces`

输出结果：

```text
NAME              STATUS   AGE
default           Active   2d20h
kube-node-lease   Active   2d20h
kube-public       Active   2d20h
kube-system       Active   2d20h
```

### 3）创建新的名称空间：

`kubectl create namespace new-space-name`

以上`namespace`均可以简写成`ns`，如：`kubectl create ns new-space-name`

## 二、Pod操作

### 1）根据配置清单创建pod：

`kubectl apply -f new-pod.yaml`

### 2）删除pod：

`kubectl delete pods pod-name` 

### 3）查看pod：

`kubectl get pods`

### 4）通过镜像文件直接创建pod和无状态应用：

```text
kubectl create deployment test1 --image=maxidea/flask-demo-app:v1.0
```

### 5）查看一下对应的pod和无状态应用：

```text
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
test1-86d54d9655-q9lv5   1/1     Running   0          5m15s

$ kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
test1   1/1     1            1           5m30s
```

访问这个服务，首先查看一下它的ip地址，可以看到它被部署到哪个节点上去（这里被部署到36工作节点上），使用命令格式：

`kubectl get pods -o wide`

```text
$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
test1-86d54d9655-q9lv5   1/1     Running   0          7m40s   10.244.2.2   36     <none>           <none>

$ curl 10.244.2.2
flask-demo-app v1.0 / ClientIP: 10.244.0.0, ServerName: test1-86d54d9655-q9lv5, ServerIP: 10.244.2.2!
```

### 6）查看pod的配置清单

```text
$ kubectl create deployment test2 --image=maxidea/flask-demo-app:v1.0 --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test2
  name: test2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test2
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test2
    spec:
      containers:
      - image: maxidea/flask-demo-app:v1.0
        name: flask-demo-app
        resources: {}
status: {}
```

### 7）查看pod的label

`kubectl get pods --show-labels`

```text
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
new-pod                  1/1     Running   0          15h     <none>
test1-86d54d9655-q9lv5   1/1     Running   0          18h     app=test1,pod-template-hash=86d54d9655
test2-69444f54b-4phj2    1/1     Running   0          7m40s   app=test2,pod-template-hash=69444f54b
```

## 三、Service操作

### 1）创建service

`kubectl create service clusterip test1 --tcp=80`

这里要注意，如果需要pod自动关联这个service，service的名字需要和pod一致。

查看这个创建的service：

```text
$ kubectl get service                            
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   3d4h
test1        ClusterIP   10.102.176.227   <none>        80/TCP    18s
```

查看service相关详细信息：

```text
$ kubectl describe service test1
Name:              test1
Namespace:         default
Labels:            app=test1
Annotations:       <none>
Selector:          app=test1
Type:              ClusterIP
IP:                10.102.176.227
Port:              80  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.2.2:80
Session Affinity:  None
Events:            <none>
```

这里看到pod test1已经作为这个service的Endpoints了。

从同一集群的其他pod（用`kubectl exec -it pod_name -- /bin/sh`进入）访问这个service，注意地址结构：`[service name].[namespace].svc.[cluster domain]`也可以使用这个service的ip地址访问：`10.102.176.227`

```text
# curl test1.default.svc.cluster.local
flask-demo-app v1.0 / ClientIP: 10.244.1.3, ServerName: test1-86d54d9655-q9lv5, ServerIP: 10.244.2.2!
```

### 2）扩展service下的pod个数

例如，把test1的副本数量扩展到三个：

`kubectl scale deployment/test1 --replicas=3`

```text
$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP           NODE   NOMINATED NODE   READINESS GATES
new-pod                  1/1     Running   0          18h    10.244.1.3   35     <none>           <none>
test1-86d54d9655-c9h5m   1/1     Running   0          84s    10.244.1.4   35     <none>           <none>
test1-86d54d9655-htbkd   1/1     Running   0          84s    10.244.3.4   37     <none>           <none>
test1-86d54d9655-q9lv5   1/1     Running   0          21h    10.244.2.2   36     <none>           <none>
```

可以看到在三个工作节点上会自动生成另外两个一样的pod。

```text
flask-demo-app v1.0 / ClientIP: 10.244.1.3, ServerName: test1-86d54d9655-htbkd, ServerIP: 10.244.3.4!
flask-demo-app v1.0 / ClientIP: 10.244.1.3, ServerName: test1-86d54d9655-q9lv5, ServerIP: 10.244.2.2!
flask-demo-app v1.0 / ClientIP: 10.244.1.3, ServerName: test1-86d54d9655-c9h5m, ServerIP: 10.244.1.4!
```

### 3）service的iptable规则

集群每创建一个新的pod，都会在节点宿主机的`iptables`上创建对应的新规则，例如：

```text
# iptables -t nat -S KUBE-SERVICES | grep test1
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.102.176.227/32 -p tcp -m comment --comment "default/test1:80 cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.102.176.227/32 -p tcp -m comment --comment "default/test1:80 cluster IP" -m tcp --dport 80 -j KUBE-SVC-VRCN2UH6RARGOJZA

# iptables -t nat -S KUBE-SVC-VRCN2UH6RARGOJZA
-N KUBE-SVC-VRCN2UH6RARGOJZA
-A KUBE-SVC-VRCN2UH6RARGOJZA -m comment --comment "default/test1:80" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-M4LMZLRGTFEOZERX
-A KUBE-SVC-VRCN2UH6RARGOJZA -m comment --comment "default/test1:80" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-LL5YVFAV2NKPJ2W3
-A KUBE-SVC-VRCN2UH6RARGOJZA -m comment --comment "default/test1:80" -j KUBE-SEP-OSFWLWIVOCAZBH6N

# iptables -t nat -S KUBE-SEP-M4LMZLRGTFEOZERX
-N KUBE-SEP-M4LMZLRGTFEOZERX
-A KUBE-SEP-M4LMZLRGTFEOZERX -s 10.244.1.4/32 -m comment --comment "default/test1:80" -j KUBE-MARK-MASQ
-A KUBE-SEP-M4LMZLRGTFEOZERX -p tcp -m comment --comment "default/test1:80" -m tcp -j DNAT [unsupported revision]
```

完整的iptables规则如下图：

![&#x6765;&#x6E90;&#xFF1A;https://github.com/cilium/k8s-iptables-diagram](../.gitbook/assets/iptables.svg)
















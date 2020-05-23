---
description: 'https://kubernetes.io/docs/concepts/workloads/controllers/deployment/'
---

# k8s滚动发布

首先把pod增加到10个：

`kubectl scale deployment/test1 --replicas=10`

```text
kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
new-pod                  1/1     Running   0          20h     10.244.1.3   35     <none>           <none>
test1-86d54d9655-9rnhg   1/1     Running   0          29s     10.244.3.6   37     <none>           <none>
test1-86d54d9655-c9h5m   1/1     Running   0          144m    10.244.1.4   35     <none>           <none>
test1-86d54d9655-ddqh4   1/1     Running   0          29s     10.244.1.6   35     <none>           <none>
test1-86d54d9655-fvc8r   1/1     Running   0          29s     10.244.2.4   36     <none>           <none>
test1-86d54d9655-gwv82   1/1     Running   0          29s     10.244.2.3   36     <none>           <none>
test1-86d54d9655-htbkd   1/1     Running   0          144m    10.244.3.4   37     <none>           <none>
test1-86d54d9655-n79mr   1/1     Running   0          29s     10.244.2.5   36     <none>           <none>
test1-86d54d9655-q9lv5   1/1     Running   0          23h     10.244.2.2   36     <none>           <none>
test1-86d54d9655-xgz9k   1/1     Running   0          29s     10.244.3.5   37     <none>           <none>
test1-86d54d9655-xmz2r   1/1     Running   0          29s     10.244.1.5   35     <none>           <none>
test2-69444f54b-4phj2    1/1     Running   0          5h19m   10.244.3.3   37     <none>           <none>
```

目前各pod内跑的都是`flask-demo-app v1.0`版本。

然后通过命令进行滚动发布v1.1版本：

```text
kubectl set image deployments/test1 flask-demo-app="maxidea/flask-demo-app:v1.1"
```

如果使用部分按比例发布，只更部分pod，那么命令改成：

```text
kubectl set image deployments/test1 flask-demo-app="maxidea/flask-demo-app:v1.1" && kubectl rollout pause deployment/test1
```

检查过发布没有问题，可以继续发布，使用命令：

`kubectl rollout resume deployment/test1`

如果需要回滚，使用命令：

`kubectl rollout undo deployment/test1`






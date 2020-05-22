# K8s基本操作命令

列出集群中所有API组名/版本号：

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

列出集群名称空间：

`kubectl get namespaces`

输出结果：

```text
NAME              STATUS   AGE
default           Active   2d20h
kube-node-lease   Active   2d20h
kube-public       Active   2d20h
kube-system       Active   2d20h
```

创建新的名称空间：

`kubectl create namespace new-space-name`

以上`namespace`均可以简写成ns，如：`kubectl create ns new-space-name`

\`\`

\`\`

\`\`

\`\`


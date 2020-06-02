---
description: 'https://kubernetes.github.io/ingress-nginx/'
---

# Ingress安装

## 一、Ingress controller 安装

下载yaml配置清单

```text
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/cloud/deploy.yaml
```

创建：

```text
kubectl apply -f ./deploy.yaml 
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
```



```text
kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.96.158.117   <pending>     80:30773/TCP,443:32233/TCP   8m48s
ingress-nginx-controller-admission   ClusterIP      10.97.62.60     <none>        443/TCP                      8m48s
```

创建过程中：

```text
kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS              RESTARTS   AGE
ingress-nginx-admission-create-8mhz8        0/1     Completed           0          2m38s
ingress-nginx-admission-patch-t568k         0/1     Completed           0          2m38s
ingress-nginx-controller-866488c6d4-85lph   0/1     ContainerCreating   0          2m48s
```

## 二、Ingress 安装








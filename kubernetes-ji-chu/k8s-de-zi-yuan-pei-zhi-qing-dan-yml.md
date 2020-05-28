# 通过K8s部署wordpress:5-php7.2-fpm

## 目标

在默认的名称空间下创建`wordpress:5-php7.2-fpm、nginx、mysql`的pod。

由于nginx和wordpress（php-fpm）需要共享一个volume，所以这两个pod需要指定在同一node上部署（使用nodeSelector）。

## 一、给一个node打标签

目前我们测试环境有三个工作节点，分别是35、36、37：

```text
kubectl get nodes
NAME   STATUS   ROLES    AGE     VERSION
31     Ready    master   8d      v1.18.2
32     Ready    master   6d19h   v1.18.2
33     Ready    master   6d19h   v1.18.2
35     Ready    <none>   6d23h   v1.18.2
36     Ready    <none>   6d23h   v1.18.2
37     Ready    <none>   6d23h   v1.18.2
```

我选择为36节点打上label：

```text
kubectl label nodes 36 nodetype=wordpress
node/36 labeled
```

用`kubectl get nodes --show-labels` 可以看到36节点最后面加入了`nodetype=wordpress`标签。

## 二、设置nginx

### 1）nginx的default.conf配置

```text
server {
    listen       80;
    server_name  localhost;
	  access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /var/www/html;
        index  index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$args;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    location ~ \.php$ {
        fastcgi_pass   wordpress.default.svc.cluster.local:9000;
        fastcgi_index  index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

### 2）设置nginx的yaml文件

```text
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
  namespace: default
data:
  default.conf: |
    server {
        listen       80;
    server_name  localhost;
    access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /var/www/html;
        index  index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$args;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    location ~ \.php$ {
        fastcgi_pass   wordpress.default.svc.cluster.local:9000;
        fastcgi_index  index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
        include        fastcgi_params;
        }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        nodetype: wordpress #指定使用的node
      containers:
      - image: nginx:alpine
        name: nginx
        volumeMounts:
        - name: ngxconfs
          mountPath: /etc/nginx/conf.d/
          readOnly: true
        - name: wwwdata
          mountPath: /var/www/html
      volumes:
      - name: ngxconfs
        configMap:
          name: nginx-conf
          optional: false
      - name: wwwdata
        hostPath:
          path: /root/wwwdata
        
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
  namespace: default
spec:
  ports:
  - name: "80"
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort #NodePort实现Pod外部通信
---

```

## 二、设置mysql

创建mysql.yaml，内容如下：

```text
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wpdb
  name: wpdb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wpdb
  template:
    metadata:
      labels:
        app: wpdb
    spec:
      containers:
      - image: mysql:5.7
        name: wpdb
        env:
        - name: MYSQL_DATABASE
          value: wpdb
        - name: MYSQL_USER
          value: wpuser
        - name: MYSQL_PASSWORD
          value: wppass
        - name: MYSQL_RANDOM_ROOT_PASSWORD
          value: '1'
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: wpdb
  name: wpdb
  namespace: default
spec:
  ports:
  - name: "3306"
    port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    app: wpdb
  type: ClusterIP
```

## 三、设置wordpress（php-fpm镜像版本）

创建wordpress.yaml

```text
apiVersion: v1
kind: ConfigMap
metadata:
  name: fpm-conf
  namespace: default
data:
  www.conf: |
    [www]
    user = www-data
    group = www-data
    listen = 127.0.0.1:9000
    pm = dynamic
    pm.max_children = 5
    pm.start_servers = 2
    pm.min_spare_servers = 1
    pm.max_spare_servers = 3
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wordpress
  name: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      nodeSelector:
        nodetype: wordpress
      containers:
      - image: wordpress:5-php7.2-fpm
        name: wordpress
        volumeMounts:
        - name: fpmconfs
          mountPath: /usr/local/etc/php-fpm.d/
          readOnly: true
        - name: wwwdata
          mountPath: /var/www/html
        env:
        - name: WORDPRESS_DB_NAME
          value: wpdb
        - name: WORDPRESS_DB_USER
          value: wpuser
        - name: WORDPRESS_DB_PASSWORD
          value: wppass
        - name: WORDPRESS_DB_HOST
          value: "wpdb.default.svc.cluster.local."
      volumes:
      - name: fpmconfs
        configMap:
          name: fpm-conf
          optional: fals
      - name: wwwdata
        hostPath:
          path: /root/wwwdata
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: wordpress
  name: wordpress
  namespace: default
spec:
  ports:
  - name: "80"
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: wordpress
  type: ClusterIP

```

## 四、创建应用

```text
$ kubectl apply -f mysql.yaml 
deployment.apps/wpdb created
service/wpdb created

$ kubectl apply -f wordpress.yaml 
configmap/fpm-conf unchanged
deployment.apps/wordpress created
service/wordpress created

$ kubectl apply -f nginx.yaml 
configmap/nginx-conf created
deployment.apps/nginx created
service/nginx created
```

查看pod的状态

```text
$ kubectl get pod -o wide
NAME                         READY   STATUS             RESTARTS   AGE     IP            NODE   NOMINATED NODE   READINESS GATES
nginx-7fc4b6d479-q5q6q       1/1     Running            0          2m6s    10.244.2.22   36     <none>           <none>
wordpress-666b699b97-5lggt   0/1     CrashLoopBackOff   7          15m     10.244.2.20   36     <none>           <none>
wpdb-698dcf9f99-pz58j        1/1     Running            0          2m53s   10.244.2.21   36     <none>           <none>

```

P.S. 在部署wordpress的容器时我曾经遇到`CrashLoopBackOff`错误，用`kubectl logs 容器Name -p`查看日志进行排障，显示为FPM的设置问题。

```text
[27-May-2020 16:03:34] ERROR: unable to bind listening socket for address 'wordpress.default.svc.cluster.local.:9000': Cannot assign requested address (99)
[27-May-2020 16:03:34] ERROR: FPM initialization failed
```

当时误把socket的监听地址写成容器地址了。

查看pod的端口：

```text
$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        8d
nginx        NodePort    10.100.128.76    <none>        80:31456/TCP   72m
wordpress    NodePort    10.100.230.206   <none>        80:31009/TCP   94s
wpdb         ClusterIP   10.99.43.10      <none>        3306/TCP       73m
```








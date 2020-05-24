# 通过K8s部署wordpress:5-php7.2-fpm

## 目标

在默认的名称空间下创建`wordpress:5-php7.2-fpm、nginx、mysql`的pod。

## 一、设置nginx

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
      containers:
      - image: nginx:alpine
        name: nginx
        volumeMounts:
        - name: ngxconfs
          mountPath: /etc/nginx/conf.d/
          readOnly: true
        - name: wwwdata
          mountPath: /var/www/html/
      volumes:
      - name: ngxconfs
        configMap:
          name: nginx-conf
          optional: false
      - hostPath:
        name: wwwdata
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
















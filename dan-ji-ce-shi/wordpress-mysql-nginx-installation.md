---
description: 'https://hub.docker.com/_/wordpress'
---

# Wordpress+Mysql+nginx反向代理安装

## 步骤1：配置mysql容器

首先创建`db`目录，下面添加一个mysql容器使用的`env.list`文件

```text
MYSQL_DATABASE=wpdb
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppass
MYSQL_RANDOM_ROOT_PASSWORD=1
```

然后启动mysql容器：

```text
docker run --name db -d --net net1 -v /simon-testing/wordpress/db/data:/var/lib/mysql --env-file ./db/env.list mysql:5.7
```

mysql跑起来了：

```text
root@09-1:/simon-testing/wordpress# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
d512e5e68e6e        mysql:5.7           "docker-entrypoint.s…"   4 seconds ago       Up 3 seconds        3306/tcp, 33060/tcp   db
```

## 步骤2：配置wordpress容器

拉取wordpress镜像，这个镜像里已经自带apache和php7.2：

`docker pull wordpress:5-php7.2`

创建wp目录，下面添加一个wordpress容器使用的`env.list`文件，注意`WORDPRESS_DB_HOST`要与mysql的容器名一致：

```text
WORDPRESS_DB_HOST=db
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=wppass
WORDPRESS_DB_NAME=wpdb
```

启动wordpress容器，把80端口绑定到宿主机的8080端口上：

```text
docker run --name wp -p 8080:80 --net net1 -d --env-file ./wp/env.list wordpress:5-php7.2
```

之后，在其他主机上用浏览器访问宿主机的8080端口，就能看到wordpress的安装引导界面了：

![](../.gitbook/assets/image%20%288%29.png)

## 步骤3：配置nginx反向代理容器

创建`nginx`目录，下面添加一个nginx容器使用的`default.conf`文件，里面设置反向代理`wp`容器：

```text
server {
    listen       80;
    server_name  localhost;
    access_log  /var/log/nginx/host.access.log  main;
    location / {
        #root   /usr/share/nginx/html;
        index  index.php index.html index.htm;
        proxy_pass http://wp/;
        #Proxy Settings
        proxy_redirect     off;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

然后运行nginx容器，link到wp容器，并把宿主机80端口与nginx容器80端口绑定：

```text
docker run --name nginx -d -v /simon-testing/wordpress/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro --link wp --net net1 -p 80:80 nginx:alpine
```

三个容器一起运行中：

```text
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                  NAMES
e160335b6070        nginx:alpine         "nginx -g 'daemon of…"   56 seconds ago      Up 54 seconds       0.0.0.0:80->80/tcp     nginx
1e0276d9a234        mysql:5.7            "docker-entrypoint.s…"   19 minutes ago      Up 19 minutes       3306/tcp, 33060/tcp    db
80f68202f960        wordpress:5-php7.2   "docker-entrypoint.s…"   25 minutes ago      Up 25 minutes       0.0.0.0:8080->80/tcp   wp
```

现在访问宿主机，不用添加8080端口也能访问到wordpress了：

![](../.gitbook/assets/image%20%283%29.png)






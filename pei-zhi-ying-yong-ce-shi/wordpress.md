---
description: 'https://hub.docker.com/_/wordpress'
---

# Wordpress+Mysql安装

## 步骤1：

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

## 步骤2：

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

最后，在其他主机上用浏览器访问宿主机的8080端口，就能看到wordpress的安装引导界面了：

![](../.gitbook/assets/image%20%284%29.png)










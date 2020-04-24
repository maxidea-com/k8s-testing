# Wordpress

首先创建`db`目录，下面添加一个mysql容器使用的`env.list`文件

```text
WORDPRESS_DB_HOST=db
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=wppass
WORDPRESS_DB_NAME=wpdb
```

然后启动mysql容器：

```text
docker run --name db -d --net net1 -v /simon-testing/wordpress/db/data:/var/lib/mysql --env-file /simon-testing/wordpress/db/env.list mysql:5.7
```




# 通过docker compose安装wordpress:5-php7.2-fpm

我们之前测试用的镜像`wordpress:5-php7.2`，它自带apache，我们当时用了nginx做反向代理。由于nginx比apache更加轻量级，所以本次测试改用`wordpress:5-php7.2-fpm`那样做好处是，直接使用nginx容器接收80端口的请求并传递给wordpress容器的fastcgi处理，fastcgi把匹配到的php文件给9000端口的fpm进行执行。

借用[Fuck PHP-FPM with FastCGI](https://blog.1pwnch.com/websecurity/2019/06/12/Fuck-PHP-FPM-with-Fastcgi/)一文的图片，就很好理解这个处理过程：

![](../.gitbook/assets/image%20%286%29.png)

首先拉取镜像：

`docker pull wordpress:5-php7.2-fpm`

清除之前测试的容器：

```text
docker stop $(docker ps -q) 
docker rm $(docker ps -aq)
```

创建一个www.conf文件，内容如下，用于等会替换容器内默认的`/usr/local/etc/php-fpm.d/www.conf`：

```text
[www]
user = www-data
group = www-data
listen = 0.0.0.0:9000
pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
```

修改nginx用的`default.conf`为如下：

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

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    
    location ~ \.php$ {
        fastcgi_pass   wp.maxidea.com:9000;
        fastcgi_index  index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

这里要注意一点，nginx容器需要访问得到wordpress-fpm容器内的`/var/www/html`目录的内容，所以在修改docker-compose文件时要注意把存储卷共享给两个容器一同访问。

修改`docker-compose.yaml`文件成如下：

```text
version: '3'

services:
  
  db:
    image: mysql:5.7
    container_name: db.maxidea.com
    volumes:
    - ./db/data/:/var/lib/mysql/
    env_file: 
    - ./db/env.list
    networks:
      net2:
        ipv4_address: 10.10.1.101
    expose:
    - "3306"

  wp:
    image: wordpress:5-php7.2-fpm
    container_name: wp.maxidea.com
    volumes:
    - ./fpm/www.conf:/usr/local/etc/php-fpm.d/www.conf:ro
    - ./wp/wwwdata:/var/www/html
    env_file: 
    - ./wp/env.list
    networks:
      net2:
        ipv4_address: 10.10.1.102
    expose:
    - "9000"
    extra_hosts:
    - "db.maxidea.com:10.10.1.101"
    - "nx.maxidea.com:10.10.1.103"

  nginx:
    image: nginx:alpine
    container_name: nx.maxidea.com
    volumes:
    - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    - ./wp/wwwdata:/var/www/html
    networks:
      net2:
        ipv4_address: 10.10.1.103
    expose:
    - "80"
    ports:
    - "80:80"
    extra_hosts:
    - "db.maxidea.com:10.10.1.101"
    - "wp.maxidea.com:10.10.1.102"

networks:
  net2: 
    ipam:
      driver: default
      config:
        - subnet: "10.10.1.0/24"
```








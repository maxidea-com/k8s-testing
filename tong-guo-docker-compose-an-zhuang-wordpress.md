---
description: 根据上一个测试的环境，使用docker compose实现一个命令安装完成wordpress+mysql+nginx反向代理
---

# 通过docker compose安装Wordpress

首先把前一个实验的三个容器全部删除（如果服务器上还有其他容器要保留，不要使用以下命令）：

```text
docker stop $(docker ps -q) && docker rm $(docker ps -aq)
```

然后创建一个`docker-compose.yaml`文件：

```text
version: '3'
  
services:

  db:
    image: mysql:5.7
    volumes:
    - ./db/data/:/var/lib/mysql/
    env_file:
    - ./db/env.list
    networks:
      webnet:
        aliases:
        - "mysql"
    expose:
    - "3306"

  wp:
    image: wordpress:5-php7.2
    env_file:
    - ./wp/env.list
    networks:
      webnet:
        aliases:
        - "wordpress"
    expose:
    - "80"
    ports:
    - "8080:80"
    depends_on:
    - db

  nginx:
    image: nginx:alpine
    volumes:
    - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      webnet:
        aliases:
        - "www"
    expose:
    - "80"
    ports:
    - "80:80"
    depends_on:
    - db
    - wp

networks:
  webnet: {}
```

安装命令工具：`apt install docker-compose`

然后使用`docker-compose up`命令运行起来

```text
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                  NAMES
ea683c8ec5dc        nginx:alpine         "nginx -g 'daemon of…"   47 seconds ago      Up 46 seconds       0.0.0.0:80->80/tcp     wordpress_nginx_1
9267439c0a0e        wordpress:5-php7.2   "docker-entrypoint.s…"   48 seconds ago      Up 47 seconds       0.0.0.0:8080->80/tcp   wordpress_wp_1
2aba0bad886d        mysql:5.7            "docker-entrypoint.s…"   48 seconds ago      Up 48 seconds       3306/tcp, 33060/tcp    wordpress_db_1
```




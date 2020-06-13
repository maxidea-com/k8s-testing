---
description: 'https://hub.docker.com/_/mysql'
---

# MySQL镜像安装和测试

拉取mysql 5.7版本的镜像：

`sudo docker pull mysql:5.7`

在宿主机上创建一个目录存放数据（作为存储卷）：

`sudo mkdir -p /data/mysql`

创建mysql容器时，加入`-v [宿主机存储卷]:[容器存储路径]` 指定绑定的存储卷位置，例如：

```text
sudo docker run -v /data/mysql:/var/lib/mysql --name mysql1 -it mysql:5.7
2020-04-23 06:13:59+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.29-1debian10 started.
2020-04-23 06:13:59+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2020-04-23 06:13:59+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.29-1debian10 started.
2020-04-23 06:13:59+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
        You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
```

这里创建出来的容器是停止状态的，而且没有启动成功，因为没有指定数据库的ROOT PASSWORD

所以命令改为加`--env` 或 `-e` 或 `--env-file`指定密码等参数运行（加-d 参数，使容器以后台方式运行，不占用终端）：

```text
sudo docker run --env MYSQL_ROOT_PASSWORD='test1234' -d -v /data/mysql:/var/lib/mysql --name mysql2 -it mysql:5.7
```

ubuntu中，如果不想一直用`sudo`操作，可以切换为root，再查看一下状态（ubuntu里首次使用root用户，记得通过`sudo passwd`设置它的密码），容器mysql2已经正常运作起来了：

```text
root@09-1:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                 NAMES
3ae0f16430b5        mysql:5.7           "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes                3306/tcp, 33060/tcp   mysql2
96e04560482d        mysql:5.7           "docker-entrypoint.s…"   12 minutes ago      Exited (1) 12 minutes ago                         mysql1
```

同时可以用`-exec`参数进入交互界面：

`docker exec -it mysql2 /bin/sh`

```text
root@09-1:~# docker exec -it mysql2 /bin/sh
# 
# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.29 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

在宿主机上，mysql的文件会保留在存储卷上，删除容器也不会丢失了：（停止mysql的容器时，使用命令`docker stop` 确保数据保存完整）

```text
# exit
root@09-1:/data/mysql# docker stop mysql2
mysql2
root@09-1:/data/mysql# docker rm mysql2
mysql2
root@09-1:/data/mysql# ls
auto.cnf    ca.pem           client-key.pem  ibdata1      ib_logfile1  mysql               private_key.pem  server-cert.pem  sys
ca-key.pem  client-cert.pem  ib_buffer_pool  ib_logfile0  ibtmp1       performance_schema  public_key.pem   server-key.pem
```



额外补充：mysql 5.7的dockerfile可以在这里查看：

[https://github.com/docker-library/mysql/blob/fcc68c84b7d30cf1ac2b4aed835425ad8311b440/5.7/Dockerfile](https://github.com/docker-library/mysql/blob/fcc68c84b7d30cf1ac2b4aed835425ad8311b440/5.7/Dockerfile)


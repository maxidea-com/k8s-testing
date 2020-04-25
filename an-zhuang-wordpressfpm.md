# 通过docker compose安装wordpress:5-php7.2-fpm

我们之前测试用的镜像`wordpress:5-php7.2`，它自带apache，我们当时用了nginx做反向代理。由于nginx比apache更加轻量级，所以本次测试改用`wordpress:5-php7.2-fpm`那样做好处是，直接使用nginx容器接收80端口的请求并传递给wordpress容器的fastcgi处理，fastcgi把匹配到的php文件给9000端口的fpm进行执行。

借用[Fuck PHP-FPM with FastCGI](https://blog.1pwnch.com/websecurity/2019/06/12/Fuck-PHP-FPM-with-Fastcgi/)一文的图片，就很好理解这个处理过程：

![](.gitbook/assets/image%20%287%29.png)

首先拉取镜像：

`docker pull wordpress:5-php7.2-fpm`

清除之前测试的容器：

```text
docker stop $(docker ps -q) 
docker rm $(docker ps -aq)
```

创建一个www.conf文件，内容如下，用于等会替换容器内默认的/usr/local/etc/php-fpm.d/www.conf：

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












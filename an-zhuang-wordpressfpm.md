# 安装wordpress-fpm

我们之前测试用的镜像是wordpress:5-php7.2，它自带apache，由于nginx比apache更加轻量级，所以这里改用`wordpress:5-php7.2-fpm` ，那样直接使用nginx容器接收80端口的请求并传递给wordpress容器的fastcgi处理，fastcgi把匹配到的php文件给9000端口的fpm进行执行。

借用[Fuck PHP-FPM with FastCGI](https://blog.1pwnch.com/websecurity/2019/06/12/Fuck-PHP-FPM-with-Fastcgi/)一文的图片，就很好理解这个处理过程：

![](.gitbook/assets/image%20%287%29.png)

`docker pull wordpress:5-php7.2-fpm`

\`\`

\`\`


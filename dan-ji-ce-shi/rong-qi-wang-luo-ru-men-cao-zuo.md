---
description: 访问容器内的mysql
---

# 容器网络入门操作1

## 测试1：从宿主机访问容器内的mysql

启动一个mysql镜像：

```text
sudo docker run --env MYSQL_ROOT_PASSWORD='test1234' -d -v /data/mysql4:/var/lib/mysql --name mysql4 -it mysql:5.7
```

然后在宿主机上使用`docker inspect mysql4 | grep IPAddress` 获取到容器的IP地址，例如`"IPAddress": "172.17.0.2"`

当然也可以使用docker inspect -f 配合模板指令获取，例如：

```text
root@09-1:/data/mysql3# docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mysql4
172.17.0.2
```

最后通过在宿主机上使用`mysql -uroot -p -h 172.17.0.2`去访问这个数据库即可。

关于docker inspect的模板指令，建议参考此文：[https://www.jianshu.com/p/65377285662e](https://www.jianshu.com/p/65377285662e)


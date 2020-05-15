---
description: 'https://hub.docker.com/'
---

# 登录docker hub并上传镜像

从terminal登录docker hub，可以拉取私有镜像或者上传自己修改过的镜像。

`sudo docker login`

然后使用`docker push`上传镜像到自己的仓库里，例如：

```text
#提交修改到镜像并命名版本为maxidea/bbsimon1，其中81f4af5e47e7为容器ID
sudo docker commit 81f4af5e47e7 maxidea/bbsimon1

#查看镜像是否生成
ubuntu@09-1:~$ sudo docker images
REPOSITORY              TAG                 IMAGE ID            CREATED              SIZE
maxidea/bbsimon1        latest              a7ca50aebb96        About a minute ago   1.22MB
busybox                 latest              88dd06b9f99d        3 hours ago          1.22MB
```

从上面可以看出，maxidea/bbsimon1这个镜像是根据busybox修改而来的。

其中maxidea是我个人的dockerhub仓库id

然后通过docker push上传到docker hub：

```text
ubuntu@09-1:~$ sudo docker push maxidea/bbsimon1
The push refers to repository [docker.io/maxidea/bbsimon1]
57670c415e28: Pushed 
5b0d2d635df8: Mounted from library/busybox 
latest: digest: sha256:1186c737e9c302ca82489a12ab4ea810396e75b49939f047fca76d2cff4f5256 size: 734
```

刷新docker hub个人页面，就能看到这个镜像了：

![](../.gitbook/assets/image%20%2830%29.png)


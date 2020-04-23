# Docker配置文件daemon.json

在/etc/docker下创建daemon.json

加入镜像加速地址：

```text
{ 
        "exec-opts": ["native.cgroupdriver=systemd"],
        "registry-mirrors": ["https://r9xxm8z8.mirror.aliyuncs.com", "https://registry.docker-cn.com", "https://docker.mirrors.ustc.edu.cn" ]
} 
```

配置完重启docker

`sudo systemctl restart docker`


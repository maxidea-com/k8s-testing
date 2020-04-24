# 安装Docker

安装步骤：

参考：阿里云镜像 [https://developer.aliyun.com/mirror/docker-ce](https://developer.aliyun.com/mirror/docker-ce)

```text
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get -y update 
sudo apt-get -y install docker-ce
```

在`/etc/docker`下创建`daemon.json`

加入镜像加速地址：

```text
{ 
        "exec-opts": ["native.cgroupdriver=systemd"],
        "registry-mirrors": ["https://r9xxm8z8.mirror.aliyuncs.com", "https://registry.docker-cn.com", "https://docker.mirrors.ustc.edu.cn" ]
} 
```

启动docker：

`sudo systemctl start docker`

查看docker状态：

`sudo docker info`

查看docker指令：

`docker run --help`


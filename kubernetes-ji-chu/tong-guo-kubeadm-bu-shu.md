# 通过Kubeadm部署K8s集群

##  目标

通过Kubeadm堆叠式部署K8s集群

## 一、准备工作

### 1）节点准备工作

#### 1-1）各节点命名方式

| 主机名 | 内网IP地址 | 域名 |
| :--- | :--- | :--- |
| 31 | 192.168.2.31 | master31.maxidea.com |
| 32 | 192.168.2.32 | master32.maxidea.com |
| 33 | 192.168.2.33 | master33.maxidea.com |
| 34 | 192.168.2.34 | node34.maxidea.com |
| 35 | 192.168.2.35 | node35.maxidea.com |
| 36 | 192.168.2.36 | node36.maxidea.com |

#### 1-2）各节点时间同步

`chronyd`使用网络上的时间服务器时间来测量本机时间的偏移量从而调整系统时钟。我们可以在里面自定义本地ntp服务器，或者默认使用它的时间同步源。

安装指令：`sudo apt install -y chrony`

跳板机（跳板机的公钥已经写入到各节点`authorized_keys`）上批量安装指令`chrony`可以用如下脚本：

```text
#! /bin/bash
server_array=(31 32 33 35 36 37)
ipadd=192.168.2
for server in ${server_array[*]}
do
echo "ssh to $server >>>"
ssh -Tq ubuntu@$ipadd.$server >/dev/null <<rmssh
cd /tmp
echo your_sudo_passwd | sudo -S bash -c "apt install -y chrony"
rmssh
done
```

安装完成后检查一下服务器时间是否一致，脚本如下：

```text
#! /bin/bash
server_array=(31 32 33 35 36 37)
ipadd=192.168.2
remote_cmd="cd /tmp;"
for server in ${server_array[*]}
do
{
    ssh -Tq ubuntu@$ipadd.$server $remote_cmd $1
}&
done
wait
```

这个脚本我一般用于批量执行远程主机命令，使用方式是`bash batch.sh [远程指令]`，例如：

```text
$bash batch.sh date
2020年 05月 17日 星期日 13:43:02 CST
2020年 05月 17日 星期日 13:43:02 CST
2020年 05月 17日 星期日 13:43:02 CST
2020年 05月 17日 星期日 13:43:02 CST
2020年 05月 17日 星期日 13:43:02 CST
```

#### 1-3）禁用swap设备

首先，在所有节点上运行`sudo swapoff -a`

批量操作我们可以用以下这个脚本实现：

```text
#! /bin/bash
sudoerpw=your_sudo_passwd
rmcmd="echo $sudoerpw | sudo -S bash -c '$1'"
server_array=(32 33 35 36 37)
ipadd=192.168.2
for server in ${server_array[*]}
do
        echo "ssh to $server >>>"
        ssh -Tq ubuntu@$ipadd.$server $rmcmd
done
```

使用方式是`bash batch_sudo.sh [远程指令]`，例如：`bash batch_sudo.sh 'swapoff -a'`

其次，在`/etc/fstab`文件里，把`swapfile`一行用\#号进行屏蔽：`sudo sed -i 's/\/swapfile/#&/' /etc/fstab`

使用批量修改脚本为`bash batch_sudo.sh 'sed -i "s/\/swapfile/#&/" /etc/fstab'`

#### 1-4）关闭防火墙

使用批量修改脚本为`bash batch_sudo.sh 'ufw disable'`

确认防火墙是否已经停掉：`bash batch_sudo.sh 'ufw status'`

如果系统是centos，则需要关闭SEinux：

1、临时关闭（不用重启机器，设置SELinux 成为permissive模式）：

`setenforce 0` 

2、修改配置文件（需要重启机器）：

修改`/etc/selinux/config` 文件

将`SELINUX=enforcing`改为`SELINUX=disabled`

#### 1-5）各节点域名解释

各节点上`/etc/hosts`要配置如1-1中给出的域名解释，使节点之间能互相访问。我这里使用批量脚本配置全部节点：

```text
#! /bin/bash
server_array=(31 32 33 35 36 37)
ipadd=192.168.2
sudoerpw=yedpay
for server in ${server_array[*]}
do
echo "ssh to $server >>>"
ssh -Tq ubuntu@$ipadd.$server >/dev/null <<rmssh
cd /etc
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.31    master31.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.32    master32.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.33    master33.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.34    node34.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.35    node35.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.36    node36.maxidea.com'>> hosts"
rmssh
done
```






















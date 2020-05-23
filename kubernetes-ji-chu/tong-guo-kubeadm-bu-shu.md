# 通过Kubeadm部署K8s集群

##  目标

通过Kubeadm部署一个简单的K8s集群。

## 一、节点准备工作

### 1）各节点命名方式

| 主机名 | 内网IP地址 | 域名 |
| :--- | :--- | :--- |
| 31 | 192.168.2.31 | master31.maxidea.com |
| 32 | 192.168.2.32 | master32.maxidea.com |
| 33 | 192.168.2.33 | master33.maxidea.com |
| 34 | 跳板机 | 跳板机 |
| 35 | 192.168.2.35 | node35.maxidea.com |
| 36 | 192.168.2.36 | node36.maxidea.com |
| 37 | 192.168.2.37 | node37.maxidea.com |

### 2）各节点时间同步

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
2020年 05月 17日 星期日 13:43:02 CST
```

### 3）禁用swap设备

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

### 4）关闭防火墙

使用批量修改脚本为`bash batch_sudo.sh 'ufw disable'`

确认防火墙是否已经停掉：`bash batch_sudo.sh 'ufw status'`

如果系统是centos，则需要关闭SEinux：

1、临时关闭（不用重启机器，设置SELinux 成为permissive模式）：

`setenforce 0` 

2、修改配置文件（需要重启机器）：

修改`/etc/selinux/config` 文件

将`SELINUX=enforcing`改为`SELINUX=disabled`

### 5）各节点域名解释

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
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.31    master31.maxidea.com    k8s-api.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.32    master32.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.33    master33.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.35    node35.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.36    node36.maxidea.com'>> hosts"
echo $sudoerpw | sudo -S bash -c "echo '192.168.2.37    node37.maxidea.com'>> hosts"
rmssh
done
```

### 6）各节点上安装docker

使用我预先准备好的一键部署脚本：

```text
sudo curl -s https://gitee.com/maxidea/shell/raw/master/docker-install-k8s.sh | bash
```

### 7）各节点上删除回环设备

```text
sudo apt autoremove --purge snapd
```

## 二、主节点安装

### 1）镜像设定

主节点和工作节点上，都需要安装`kubelet、kubeadm、kubectl`三个组件，参考阿里云镜像站：[https://developer.aliyun.com/mirror/kubernetes](https://developer.aliyun.com/mirror/kubernetes)

```text
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

或者使用我预先准备好的一键部署脚本：

```text
sudo curl -s https://gitee.com/maxidea/shell/raw/master/k8s-install-init.sh | bash
```

### 2）主节点初始化

由于网络原因，国内无法直接访问gcr.io，所以在kubeadm init时要指定参数，使用阿里云镜像：

```text
kubeadm init  --image-repository=registry.aliyuncs.com/google_containers
```

由于31节点同时用来做高可用集群的负载均衡器，所以要加上参数：

```text
--control-plane-endpoint k8s-api.maxidea.com 
--apiserver-advertise-address 192.168.2.31 
```

pod网络默认用flannel的10.244.0.0./16：

```text
--pod-network-cidr 10.244.0.0/16
```

整条命令运行下来：

```text
$sudo kubeadm init \
    --image-repository=registry.aliyuncs.com/google_containers \
    --control-plane-endpoint k8s-api.maxidea.com \
    --apiserver-advertise-address 192.168.2.31 \
    --pod-network-cidr 10.244.0.0/16

#省略执行过程#

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join k8s-api.maxidea.com:6443 --token q9de5i.toeqluiwv9ij99o2 \
    --discovery-token-ca-cert-hash sha256:2556bfaa0cb5a99771867b310eb00f38736736da33289d040a4e88c36d7d81af \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-api.maxidea.com:6443 --token q9de5i.toeqluiwv9ij99o2 \
    --discovery-token-ca-cert-hash sha256:2556bfaa0cb5a99771867b310eb00f38736736da33289d040a4e88c36d7d81af         
```

### 3）主节点其他配置

接着按照上面提示执行三行命令（必须使用非root用户，我们默认使用系统ubuntu用户）：

```text
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

现在可以用kubectl查看节点状态：

```text
$ kubectl get nodes 
NAME   STATUS     ROLES    AGE    VERSION
31     NotReady   master   9m9s   v1.18.2
```

这里显示节点状态为NotReady，为探究竟，可以使用`kubectl describe node 31` 查看一下原因：

```text
Events:
  Type    Reason                   Age   From            Message
  ----    ------                   ----  ----            -------
  Normal  Starting                 37m   kubelet, 31     Starting kubelet.
  Normal  NodeHasSufficientMemory  37m   kubelet, 31     Node 31 status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    37m   kubelet, 31     Node 31 status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     37m   kubelet, 31     Node 31 status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  37m   kubelet, 31     Updated Node Allocatable limit across pods
  Normal  Starting                 37m   kube-proxy, 31  Starting kube-proxy.
```

显示“`NodeHasSufficientMemory`”，节点的内存不足，我们需要分配更多内存给这台主机。（分配完更多内存后还出现同样问题，则需要分析其他原因）

如果出错信息里带有：

```text
runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
```

则表示CNI插件还没有运行起来，我们可以先把`flannel`安装完成，再回过头来处理其他错误信息。

### 4）主节点安装CNI（Flannel）

根据[https://kubernetes.io/docs/concepts/cluster-administration/addons/](https://kubernetes.io/docs/concepts/cluster-administration/addons/)页面的提示，我们可以找到flannel的项目github地址[https://github.com/coreos/flannel](https://github.com/coreos/flannel) ，并且在README.md下可以看到这个提示：

For Kubernetes v1.7+ `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

由于墙的原因，访问上述github的raw地址会经常出错，所以我们可以通过fork flannel项目的github然后再同步到国内自己的码云repo（具体方式请自行baidu），然后再通过码云的raw地址获取上面的配置清单文件。例如：[https://gitee.com/maxidea/flannel/raw/master/Documentation/kube-flannel.yml](https://gitee.com/maxidea/flannel/raw/master/Documentation/kube-flannel.yml)

于是，安装命令改为：

```text
$ kubectl apply -f https://gitee.com/maxidea/flannel/raw/master/Documentation/kube-flannel.yml
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created
```

验证一下是否跑起来了：

```text
$ kubectl get pod -n kube-system 
NAME                          READY   STATUS    RESTARTS   AGE
coredns-7ff77c879f-7t8nw      1/1     Running   0          8h
coredns-7ff77c879f-bpkwh      1/1     Running   0          8h
etcd-31                       1/1     Running   2          8h
kube-apiserver-31             1/1     Running   2          8h
kube-controller-manager-31    1/1     Running   2          8h
kube-flannel-ds-amd64-76cdz   1/1     Running   0          2m29s
kube-proxy-hwfss              1/1     Running   2          8h
kube-scheduler-31             1/1     Running   2          8h
```

再次用`kubectl get nodes`查看节点状态，这个主节点已经从`NotReady`变成`Ready`了：

```text
$ kubectl get nodes              
NAME   STATUS   ROLES    AGE   VERSION
31     Ready    master   8h    v1.18.2
```

## 三、工作节点安装

### 1）工作节点安装组件

如2-1的操作，各工作节点上运行一下一键部署脚本

```text
sudo curl -s https://gitee.com/maxidea/shell/raw/master/k8s-install-init.sh | bash
```

### 2）工作节点加入到集群

如2-2初始化第一个控制平面时，获得的提示：

```text
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-api.maxidea.com:6443 --token q9de5i.toeqluiwv9ij99o2 \
    --discovery-token-ca-cert-hash sha256:2556bfaa0cb5a99771867b310eb00f38736736da33289d040a4e88c36d7d81af 
```

本例子里，我的三个工作节点（35、36、37）上均以root用户执行以上语句加入到集群里。

### 3）token过期的处理方式

一般主节点初始化时产生的token有效期只有24小时（使用命令`kubeadm token list`可查看token状态），过期后其他节点就没办法加入。

token过期之后，我们需要在**控制平面（第一个主节点）**上重新生成新的token，才能继续操作集群。命令如下：

`kubeadm token create`

在控制平面上查看新的token id：

```text
kubeadm token list  
TOKEN                     TTL         EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
qizy9v.32fin79ao8ekgrip   23h         2020-05-21T20:55:26+08:00   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
```

生成新的token后，**工作节点**上运行`kubeadm join`替换后面的`--token q9de5i.toeqluiwv9ij99o2` 为 `--token qizy9v.32fin79ao8ekgrip`即可，但`--discovery-token-ca-cert-hash sha256:2556bfaa0cb5a99771867b310eb00f38736736da33289d040a4e88c36d7d81af`保持不变，例如：

```text
# kubeadm join k8s-api.maxidea.com:6443 --token qizy9v.32fin79ao8ekgrip --discovery-token-ca-cert-hash sha256:2556bfaa0cb5a99771867b310eb00f38736736da33289d040a4e88c36d7d81af 
W0520 20:59:08.784524    4146 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

大约10～30秒后，节点上的pod启动完毕，在控制平面上再次运行`kubectl get nodes`，就可以看到各工作节点已经加入了：

```text
$ kubectl get nodes
NAME   STATUS   ROLES    AGE     VERSION
31     Ready    master   29h     v1.18.2
35     Ready    <none>   4m56s   v1.18.2
36     Ready    <none>   53s     v1.18.2
37     Ready    <none>   43s     v1.18.2
```

### 4）工作节点移除

 对节点执行维护操作之前（例如：内核升级，硬件维护等），可以使用 `kubectl drain` 安全驱逐节点上面所有的 pod。

`kubectl drain <node name>` \#该节点上的pod会被删除并调度到其他节点上面。

`kubectl cordon/uncordon <node name>` \#维护/结束维护节点，pod不移除

## 四、主节点高可用安装

为了实现集群的高可用，我们在32和33主机上增加两个控制平面，把它们以主节点的方式加入到集群里。

### 1）主节点证书CA的分类和共享

我们创建集群的第一个主节点的`/etc/kubernetes/pki/`目录下，主要有三种类型的证书：

| 文件 | CN | 说明 |
| :--- | :--- | :--- |
| ca.pem,key | kubernetes-ca | k8s集群相关证书 |
| etcd-ca.pem,key | etcd-ca | etcd集群相关证书 |
| front-proxy-ca.pem,key | kubernetes-front-proxy-ca | [front-end proxy](https://kubernetes.io/docs/tasks/access-kubernetes-api/configure-aggregation-layer/) CA |

其他加入集群的主节点，需要和这个控制平面共享这一些证书，所以可以复制到对应的主节点，或者使用`kubeadm`上传到集群的secret名称空间上去。

```text
# kubeadm init phase upload-certs --upload-certs
W0521 00:32:26.439917     753 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
76ac4d6f4ab459cd57bbb1ba874d7b6eae188116fcb0089eca621fcdad0ed490
```

完成后使用普通用户运行`kubectl get secret -n kube-system`命令查看。

### 2）其他主节点加入到集群

完成证书复制后，可以开始把其他主节点加入成为控制平面，如2-2初始化第一个控制平面时，获得的提示：

```text
You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join k8s-api.maxidea.com:6443 --token q9de5i.toeqluiwv9ij99o2 \
    --discovery-token-ca-cert-hash sha256:2556bfaa0cb5a99771867b310eb00f38736736da33289d040a4e88c36d7d81af \
    --control-plane 
```

同样，由于token进行了变更，**其他主节点**上运行`kubeadm join`替换后面的`--token q9de5i.toeqluiwv9ij99o2` 为 `--token qizy9v.32fin79ao8ekgrip`即可，记得最后面加上 `--control-plane`，不然会变成工作节点。后面再加上验证名称空间用的`certificate key`（如4-1生成的），完整的命令如下：

```text
  kubeadm join k8s-api.maxidea.com:6443 --token qizy9v.32fin79ao8ekgrip \
    --discovery-token-ca-cert-hash sha256:2556bfaa0cb5a99771867b310eb00f38736736da33289d040a4e88c36d7d81af \
    --control-plane --certificate-key 76ac4d6f4ab459cd57bbb1ba874d7b6eae188116fcb0089eca621fcdad0ed490
```

本例子里，我在32，33上执行这条命令，使它们都变成控制平面加入到集群里了。

```text
$ kubectl get nodes
NAME   STATUS   ROLES    AGE     VERSION
31     Ready    master   33h     v1.18.2
32     Ready    master   5m10s   v1.18.2
33     Ready    master   2m29s   v1.18.2
35     Ready    <none>   3h49m   v1.18.2
36     Ready    <none>   3h44m   v1.18.2
37     Ready    <none>   3h44m   v1.18.2
```

到此为止，k8s一个简单的集群安装完成。






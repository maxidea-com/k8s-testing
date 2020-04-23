# 前期准备

根据本地测试环境，搭建docker及k8s相关测试环境。

物理主机使用64g内存的服务器一台（DELL R720），上面用ESXi划分6个虚拟机做宿主机（安装Ubuntu18.04LTS），再根据实际测试需要进行docker及k8s测试。

主机及ip如下：

| 主机名 | 内网IP地址 | CPU核心 | 内存GB |
| :--- | :--- | :--- | :--- |
| 06 | 192.168.2.21 | 虚拟4核 | 12 |
| 07 | 192.168.2.22 | 虚拟4核 | 12 |
| 08 | 192.168.2.23 | 虚拟4核 | 12 |
| 09-1 | 192.168.2.24 | 虚拟4核 | 6 |
| 09-2 | 192.168.2.26 | 虚拟4核 | 6 |
| 10 | 192.168.2.25 | 虚拟4核 | 12 |




# 使用kubeadm安装k8s 1.14.1

## 1.硬件环境
> 硬件环境

|System|Hostname|IP|内存|
|-|-|-|-|
|CentOS 7|k8s-master|192.168.0.70|2G|
|CentOS 7|k8s-node1|192.168.0.71|2G|
|CentOS 7|k8s-node2|192.168.0.72|2G|
网络插件：calico

## 2.所有主机环境配置


- 关闭firewalld

```` systemctl stop firewalld && systemctl disable firewalld  ````
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

- 关闭SElinux

```` setenforce 0 && sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config ````

- 关闭Swap

```` swapoff -a && sed -i "s/\/dev\/mapper\/centos-swap/\#\/dev\/mapper\/centos-swap/g" /etc/fstab ````

- 使用阿里云yum源

    yum install -y wget
    wget -O /etc/yum.repos.d/CentOS7-Aliyun.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
    
- 配置主机名

    vi /etc/hosts
    
    192.168.0.70   k8s-master
    192.168.0.71   k8s-node1
    192.168.0.72   k8s-node2
    
    echo "k8s-master" > /etc/hostname
    hostname k8s-master
    
    echo "k8s-node1" > /etc/hostname
    hostname k8s-node1
    
    echo "k8s-node2" > /etc/hostname
    hostname k8s-node2
    
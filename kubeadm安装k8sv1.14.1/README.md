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
关闭firewalld

` systemctl stop firewalld && systemctl disable firewalld  `

关闭SElinux

` setenforce 0 && sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config `

关闭Swap

` swapoff -a && sed -i "s/\/dev\/mapper\/centos-swap/\#\/dev\/mapper\/centos-swap/g" /etc/fstab `

使用阿里云yum源

    yum install -y wget
    wget -O /etc/yum.repos.d/CentOS7-Aliyun.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
    

配置主机名

    vi /etc/hosts
     
    192.168.0.70  k8s-master
    192.168.0.71  k8s-node1
    192.168.0.72  k8s-node2
     
    echo "k8s-master" /etc/hostname && hostname k8s-master
    
    echo "k8s-node1" /etc/hostname && hostname k8s-node1
    
    echo "k8s-node2" /etc/hostname && hostname k8s-node2
        
## 3.所有主机安装docker
安装阿里云docker源

`wget -O /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`

安装docker 18.06.3

`yum install -y docker-ce-18.06.3.ce`

启动docker

`systemctl enable docker && systemctl start docker`

调整docker部分参数

    cat <<EOF > /etc/docker/daemon.json
    {
      "registry-mirrors":["https://registry.docker-cn.com"],
      "exec-opts": ["native.cgroupdriver=systemd"]
    }
    EOF
    
重新启动docker

`systemctl daemon-reload && systemctl restart docker`

## 4.所有主机安装k8s初始化工具
使用阿里云的kubernetes源

    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF

安装k8s 1.14.1版本工具

`yum install -y kubelet-1.14.1 kubeadm-1.14.1 kubectl-1.14.1`

启动kubelet

`systemctl enable kubelet && systemctl start kubelet`

## 5.master初始化设置
执行下载镜像脚本
        
    #!/bin/bash
    
    set -e
    
    KUBE_VERSION=v1.14.1
    KUBE_PAUSE_VERSION=3.1
    ETCD_VERSION=3.3.10
    CORE_DNS_VERSION=1.3.1
    
    GCR_URL=k8s.gcr.io
    ALIYUN_URL=registry.cn-hangzhou.aliyuncs.com/google_containers
    
    images=(kube-proxy:${KUBE_VERSION}
    kube-scheduler:${KUBE_VERSION}
    kube-controller-manager:${KUBE_VERSION}
    kube-apiserver:${KUBE_VERSION}
    pause:${KUBE_PAUSE_VERSION}
    etcd:${ETCD_VERSION}
    coredns:${CORE_DNS_VERSION})
    
    
    for imageName in ${images[@]} ; do
      docker pull $ALIYUN_URL/$imageName
      docker tag  $ALIYUN_URL/$imageName $GCR_URL/$imageName
      docker rmi $ALIYUN_URL/$imageName
    done
    
初始化集群

`kubeadm init --kubernetes-version=v1.14.1 --pod-network-cidr=10.17.0.0/16`   

初始化成功后按照提示执行

    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    chown $(id -u):$(id -g) $HOME/.kube/config 

安装calico网络插件

`kubectl apply -f calico.yaml`


## 6.node加入集群
执行下载镜像脚本

    #!/bin/bash
    
    set -e
    
    KUBE_VERSION=v1.14.1
    KUBE_PAUSE_VERSION=3.1
    
    
    GCR_URL=k8s.gcr.io
    ALIYUN_URL=registry.cn-hangzhou.aliyuncs.com/google_containers
    
    images=(kube-proxy:${KUBE_VERSION}
    pause:${KUBE_PAUSE_VERSION})
    
    
    for imageName in ${images[@]} ; do
      docker pull $ALIYUN_URL/$imageName
      docker tag  $ALIYUN_URL/$imageName $GCR_URL/$imageName
      docker rmi $ALIYUN_URL/$imageName
    done
    
执行初始化master完成后控制台打印的加入集群命令，加入集群

    kubeadm join 192.168.0.70:6443 --token v3o2m8.uqvveyu70140wsg8 \
      --discovery-token-ca-cert-hash sha256:03264c87a65c76e292bc904b72d6fa1e057c0a0fdd8aba648ae04c2159ac2266

## 附：常用命令集

    #查看已经运行的k8s系统pod
    kubectl get pod -n kube-system -owide
    
    #查看节点状态
    kubectl get node -owide
    
    #master删除节点
    kubectl delete node [node-name]

    #node节点重置
    kubeadm reset



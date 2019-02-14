1，设置关闭防火墙及SELINUX
---
    systemctl stop firewalld && systemctl disable firewalld
    setenforce 0
    vi /etc/selinux/config
    SELINUX=disabled

2，关闭Swap,打开/etc/fstab，注释掉 swap这一行
---
    swapoff -a && sysctl -w vm.swappiness=0
    vi /etc/fstab
    #UUID=7bff6243-324c-4587-b550-55dc34018ebf swap  swap    defaults        0 0

3，开启linux内核网桥转发
---
    cat << EOF | tee /etc/sysctl.d/k8s.conf
    net.ipv4.ip_forward = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sysctl -p /etc/sysctl.d/k8s.conf

4，设置docker 开机启动
---
    systemctl enable docker

5，国内源 安装 kubelet kubeadm kubectl
---
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=0
    repo_gpgcheck=0
    gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF
    
    yum makecache fast
    yum install -y kubelet kubeadm kubectl
6，开启 systemctl enable kubelet.service
---
7，准备所需镜像（提前找好所需镜像，可以借助阿里云的镜像管理构建）
---
    #所需镜像列表如下：
    k8s.gcr.io/kube-proxy:v1.13.0
    k8s.gcr.io/kube-apiserver:v1.13.0
    k8s.gcr.io/kube-scheduler:v1.13.0
    k8s.gcr.io/kube-controller-manager:v1.13.0
    k8s.gcr.io/coredns:1.2.6
    k8s.gcr.io/etcd:3.2.24
    k8s.gcr.io/pause:3.1
8，kubeadm init 初始话master1
    kubeadm init --kubernetes-version=v1.13.0 --pod-network-cidr=10.244.0.0/16
9，配置 kubectl 的权限
---
    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
10，验证
---
    kubectl get all --all-namespaces=true
    #这时除coreDNS pod启动失败，其余pod启动正常
11,验证完成后，配置一些alias方便使用kubectl
---
    vi /etc/profile
    #在文件末尾加上
    alias kga='kubectl get all --all-namespaces=true'
    alias kcf='kubectl create -f '
    alias kdf='kubectl delete -f '
    alias klf='kubectl logs -f '
    alias kdp='kubectl delete pod '
    alias kaf='kubectl apply -f '
    alias kgp='kubectl get pod --all-namespaces=true'
    
    source /etc/profile
12,master1 打tag
---
   kubectl label nodes master1 node=kube-system
   #这使得之后创建的关于kube-system namespace下的pod，会被调度到master1上


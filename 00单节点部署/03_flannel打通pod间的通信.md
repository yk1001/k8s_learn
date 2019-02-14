1,将yamlfiles文件夹负责到本地
---
2,创建secret对象 system-sec 两种方式创建
---
   1，kubectl create secret docker-registry system-sec --docker-server=registry.cn-hangzhou.aliyuncs.com --docker-username=yukang0001 --docker-password=1qaz@WSX --namespace=kube-system
   1,kaf yamlfiles/kube-system/00dockerhub-secret.yaml
   #00dockerhub-secret.yaml 中dockerconfigjson 的生成
   #echo {"auths":{"registry.cn-hangzhou.aliyuncs.com":{"username":"yukang0001","password":"1qaz@WSX","auth":"eXVrYW5nMDAwMToxcWF6QFdTWA=="}}} | base64
3,部署flannel
---
    kaf yamlfiles/kube-system/base00flannel.yaml
4,验证
---
    ifconfig
    #这时会有意图tun设备flannel.1 和一个网桥设备 cni0
    #查看coreDNS是否正确启动，这时coreDNS应该已正确启动
5，flannel说明
---
    flannel 对于打通容器间的通信，提供三种方案，UDP,vxlan,host-gw
    vxlan是默认方案，原理是对"三层的数据包"包一层"二层协议头"，最后再包一层"三层网络协议头",使用了linux支持的vxlan
        #结合veth、网桥、tun、iptables共同完成容器间通信
        #flannel 使用cni0网桥代替了docker0网桥来实现，同一宿主机下容器间通信，
        #因为网桥的实现是通过iptables，若docker0网桥与cni0网桥子网有冲突，容器间无法通信完全可以关闭docker0网桥 ifconfig docker0 down
        #pod中的容器共享ip，pod中会有一个虚拟网卡，并分配一个ip
        #pod若理解为实体机，多个pod间通信就需要，借助网线与交换机，veth设备就相当与网线，网桥就相当于交换机
        #pod中的网卡，是veth设备的一端，另一端连接网桥cni0
        #pod跨宿主机的通信，就需要借助tun设备flannel.1，flanneld可以收到 flannel.1 设备中的数据包，flanneld对数据包进行再包装，用于跨宿主机的通信
    UDP是早期方案，需要多次的linux内核用户态和内核态 的切换，效率较低
    host-gw 原理是iptables的转发规则，需要宿主机二层网络联通，适合于大型集群（大型集群有更好的解决方案）
    
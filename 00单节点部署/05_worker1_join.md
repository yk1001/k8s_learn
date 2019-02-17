1，环境准备
---
    执行00_环境准备.md、01_更新docker版本.md 中的步骤
    注意 ip定义为192.168.100.131 hostname 改为 worker1
2,join worker1
---
    执行02_init_master1.md 的前6步骤，注意kubectl 可以不用安装
    kubeadm join --token 19f284.da47998c9abb01d3 192.168.100.130:6443 --discovery-token-ca-cert-hash sha256:0fd95a9bc67a7bf0ef42da968a0d55d92e52898ec37c971bd77ee501d845b538
    注意：token失效时间为24小时，生成新的token和获取ca证书sha256编码hash值如下
    #切换到master1上执行 下面的命令 生成token
    kubeadm token create
    #切换到master1上执行下面的命令  获取ca证书sha256编码hash值
    openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
3，验证worker1是否加入集群
---
    在master1上执行kubectl get node，查看worker1是否已经可以查到
    在master1上执行kgp ,查看flannelpod是否已经调度并启动成功,若已经启动成功，则kubectl get node返回woker1已经为ready
4，worker1打标签，方便后期app应用调度到worker1
---
    在master1上执行 kubectl label nodes woker1 node=app
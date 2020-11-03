---
title: 利用kubeadm安装k8s
date: 2019-08-26 20:41:43
tags: 
  - k8s
  - docker
---

## 环境

### 主机配置和角色分配
ip|host|角色|配置
---|---|---|---
192.168.136.130|master.huany.com|master|CentOS 7.6
192.168.136.131|node1.huany.com|node1|CentOS 7.6
192.168.136.132|node2.huany.com|node2|CentOS 7.6

## 预先设置

### 关闭swap
- 临时关闭，运行：`swapoff -a`，下次启动还有
- 到/etc/fstab中永久删除或关闭swap分区，使用 # 注释掉即可。

### 关闭和清理ufw
下面的命令将清除现有的所有防火墙规则：
```
$ iptables -F
```

### 设置cgroups
确保kubelet使用的cgroup driver与 Docker的一致。要么使用下面的方法更新 Docker：
```
cat << EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```
要么，设置kubernetes的cgroup driver，如：kubelet 的 --cgroup-driver 标志设置为与 Docker 一样(e.g. cgroupfs)。


## 安装master节点

### 修改内核配置

- 编辑 /etc/sysctl.conf
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
vm.swappiness = 0
net.ipv4.neigh.default.gc_stale_time = 120
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_announce = 2
net.ipv4.conf.all.arp_announce = 2
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 1024
net.ipv4.tcp_synack_retries = 2
kernel.sysrq = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 1024 65023
net.ipv4.tcp_max_syn_backlog = 10240
net.ipv4.tcp_max_tw_buckets = 400000
net.ipv4.tcp_max_orphans = 60000
net.ipv4.tcp_synack_retries = 3
net.core.somaxconn = 10000
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness = 0
```

- 然后执行`sysctl -p`使修改生效

```
$ sysctl -p
```

### 安装docker环境

```
1. SET UP THE REPOSITORY

$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

2. INSTALL DOCKER ENGINE - COMMUNITY

$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable

安装18.06.0.ce-3版本
sudo yum install docker-ce-18.06.0.ce-3.e17 docker-ce-cli-18.06.0.ce-3.el7 containerd.io

3. RUN DOCKER

$ sudo systemctl start docker
```

### 安装配置kubeadm

- 配置阿里云的kubernetes镜像仓库

```
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
EOF

$ yum makecache fast
```

- 安装kubeadm cni 等工具

```
yum install -y kubernetes-cni kubelet kubeadm kubectl --skip-broken
systemctl start kubelet.service
systemctl enable kubelet.service
```

**默认情况下这里的kubeadm kubectl kubelet默认安装的都是最新版本**

- 配置CNI网络配置

```
mkdir /etc/cni/net.d/ -p
$ cat >/etc/cni/net.d/10-mynet.conf <<-EOF
{
    "cniVersion": "0.3.0",
    "name": "mynet",
    "type": "bridge",
    "bridge": "cni0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "subnet": "10.244.0.0/16",
        "routes": [
            {"dst": "0.0.0.0/0"}
        ]
    }
}
EOF
$ cat >/etc/cni/net.d/99-loopback.conf <<-EOF
{
    "cniVersion": "0.3.0",
    "type": "loopback"
}
EOF
```

- 配置kubeadm的配置文件

```
kind: MasterConfiguration
apiVersion: kubeadm.k8s.io/v1alpha2
#kubernetesVersion: "stable"
kubernetesVersion: "v1.15.3"
apiServerCertSANs: []
#imageRepository: crproxy.trafficmanager.net:6000/google_containers
#imageRepository: mirrorgooglecontainers
imageRepository: registry.aliyuncs.com/google_containers
#imageRepository: ""
controllerManagerExtraArgs:
  horizontal-pod-autoscaler-use-rest-clients: "true"
  horizontal-pod-autoscaler-sync-period: "10s"
  node-monitor-grace-period: "10s"
  feature-gates: "AllAlpha=true"
  enable-dynamic-provisioning: "true"
apiServerExtraArgs:
  runtime-config: "api/all=true"
  feature-gates: "AllAlpha=true"
  #feature-gates: "CoreDNS=true"
networking:
  podSubnet: "10.244.0.0/16"
```

- 拉取镜像

脚本如下，如果需要其它的容器镜像可以照此增加即可，可以将版本号修改为自己需要的。
- 注意：kubernetes每个版本依赖的版本不同，下面适用1.15.3。

```
echo "=================================================="
echo "Pulling Docker Images from registry.aliyuncs.com..."

echo "==>kube-apiserver:"
docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.15.3
docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.15.3 k8s.gcr.io/kube-apiserver:v1.15.3

echo "==>kube-controller-manager:"
docker pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.15.3
docker tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.15.3 k8s.gcr.io/kube-controller-manager:v1.15.3

echo "==>kube-scheduler:"
docker pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.15.3 
docker tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.15.3 k8s.gcr.io/kube-scheduler:v1.15.3

echo "==>kube-proxy:"
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.15.3
docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.15.3 k8s.gcr.io/kube-proxy:v1.15.3

echo "==>k8s-dns-kube-dns:"
docker pull registry.aliyuncs.com/google_containers/coredns:1.3.1
docker tag registry.aliyuncs.com/google_containers/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1

echo "==>etcd:"
docker pull registry.aliyuncs.com/google_containers/etcd:3.3.10
docker tag registry.aliyuncs.com/google_containers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10

echo "==>pause:"
docker pull registry.aliyuncs.com/google_containers/pause:3.1
docker tag registry.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1

echo finished.
echo "you are so lucy"
```

- 初始化

```
$ kubeadm init --config kubeadm.yml --pod-network-cidr 10.244.0.0/16 -ignore-preflight-errors all
```

执行成功后可以看到如下内容：

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.136.130:6443 --token 32ozhx.8xneb9rzjukjc3au \
    --discovery-token-ca-cert-hash sha256:00c3d4e272d3736c0edcfcdda38f255c9fb75118a7d640a3e98542829f4bb3ec 
```

- 切换到普通用户执行

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- 查看节点

```
$ kubectl get nodes
NAME               STATUS   ROLES    AGE    VERSION
master.huany.com   NotReady    master   2d3h   v1.15.3
```

如果重启或者kubeadm reset 将清理掉cni配置，需要重新配置

```
mkdir /etc/cni/net.d/ -p
cat >/etc/cni/net.d/10-mynet.conf <<-EOF
{
    "cniVersion": "0.3.0",
    "name": "mynet",
    "type": "bridge",
    "bridge": "cni0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "subnet": "10.244.0.0/16",
        "routes": [
            {"dst": "0.0.0.0/0"}
        ]
    }
}
EOF
cat >/etc/cni/net.d/99-loopback.conf <<-EOF
{
    "cniVersion": "0.3.0",
    "type": "loopback"
}
EOF
```

查看节点情况

```
$ kubectl get nodes
NAME               STATUS   ROLES    AGE    VERSION
master.huany.com   Ready    master   2d3h   v1.15.3
```

## 在node节点上安装

同上，修改Linux内核，安装docker，安装kubeadm 

- 加入到集群中去
```
kubeadm join 192.168.136.130:6443 --token 32ozhx.8xneb9rzjukjc3au \
    --discovery-token-ca-cert-hash sha256:00c3d4e272d3736c0edcfcdda38f255c9fb75118a7d640a3e98542829f4bb3ec 
```

- 配置cni网络

```
mkdir /etc/cni/net.d/ -p
cat >/etc/cni/net.d/10-mynet.conf <<-EOF
{
    "cniVersion": "0.3.0",
    "name": "mynet",
    "type": "bridge",
    "bridge": "cni0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "subnet": "10.244.0.0/16",
        "routes": [
            {"dst": "0.0.0.0/0"}
        ]
    }
}
EOF
cat >/etc/cni/net.d/99-loopback.conf <<-EOF
{
    "cniVersion": "0.3.0",
    "type": "loopback"
}
EOF
```

### 在master上查看节点情况：

```
[meichaofan@master keights]$ kubectl get nodes
NAME               STATUS   ROLES    AGE    VERSION
master.huany.com   Ready    master   2d3h   v1.15.3
node1.huany.com    Ready    <none>   2d3h   v1.15.3
node2.huany.com    Ready    <none>   2d3h   v1.15.3
```

- 在master附属flannel网络

``` 
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml
```

- 查看运行的服务

```
[meichaofan@master keights]$ kubectl get pods -n kube-system
NAME                                       READY   STATUS             RESTARTS   AGE
coredns-5c98db65d4-mc25l                   1/1     Running            17         7h18m
coredns-5c98db65d4-p7mtp                   1/1     Running            16         7h19m
etcd-master.huany.com                      1/1     Running            2          2d3h
kube-apiserver-master.huany.com            1/1     Running            2          2d3h
kube-controller-manager-master.huany.com   0/1     CrashLoopBackOff   22         2d3h
kube-flannel-ds-8l7jc                      1/1     Running            0          2d3h
kube-flannel-ds-l952q                      1/1     Running            2          2d3h
kube-flannel-ds-qmfgk                      1/1     Running            0          2d3h
kube-proxy-b2r8g                           1/1     Running            3          2d3h
kube-proxy-ccdr4                           1/1     Running            0          2d3h
kube-proxy-klp8f                           1/1     Running            0          2d3h
kube-scheduler-master.huany.com            1/1     Running            17         2d3
```

### 在node上拉取pause、kubeproxy容器，并转换tag

- node1
```
# pause
docker pull registry.aliyuncs.com/google_containers/pause:3.1
docker tag registry.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1

# kube-proxy
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.15.3
docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.15.3 k8s.gcr.io/kube-proxy:v1.15.3
```

# 安装k8s集群-准备工作

注意事项:

> 1.安装k8s前请先安装好docker，具体安装方法参考文档<<docker生产安装.md>>。
> 2.文档中IP地址为示意地址，安装时请替换为实际生产地址。
> 3.本文档不要一次性执行一个命令框（灰色框）内的全部命令，应按照步骤说明分步执行。


## 准备工作

规划机器。操作系统：CentOS Linux release 7.9.2009 (Core)

~~~
192.168.1.11 master
192.168.1.12 node1
192.168.1.13 node2
~~~

提前准备docker镜像

~~~
#拉取镜像
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.19.7
docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.19.7
docker pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.19.7
docker pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.19.7 
docker pull registry.aliyuncs.com/google_containers/etcd:3.4.13-0 
docker pull registry.aliyuncs.com/google_containers/coredns:1.7.0
docker pull registry.aliyuncs.com/google_containers/pause:3.2
docker pull calico/node:v3.17.2
docker pull calico/pod2daemon-flexvol:v3.17.2
docker pull calico/cni:v3.17.2           
docker pull calico/kube-controllers:v3.17.2
docker pull kubernetesui/dashboard:v2.0.0-rc7
docker pull kubernetesui/metrics-scraper:v1.0.4

#保存镜像
docker save -o  ./tars/1.tar registry.aliyuncs.com/google_containers/kube-proxy:v1.19.7
docker save -o  ./tars/2.tar registry.aliyuncs.com/google_containers/kube-apiserver:v1.19.7
docker save -o  ./tars/3.tar registry.aliyuncs.com/google_containers/kube-scheduler:v1.19.7
docker save -o  ./tars/4.tar registry.aliyuncs.com/google_containers/kube-controller-manager:v1.19.7 
docker save -o  ./tars/5.tar registry.aliyuncs.com/google_containers/etcd:3.4.13-0 
docker save -o  ./tars/6.tar registry.aliyuncs.com/google_containers/coredns:1.7.0
docker save -o  ./tars/7.tar registry.aliyuncs.com/google_containers/pause:3.2
docker save -o  ./tars/9.tar calico/node:v3.17.2
docker save -o  ./tars/10.tar calico/pod2daemon-flexvol:v3.17.2
docker save -o  ./tars/11.tar calico/cni:v3.17.2           
docker save -o  ./tars/12.tar calico/kube-controllers:v3.17.2
docker save -o  ./tars/13.tar kubernetesui/dashboard:v2.0.0-rc7
docker save -o  ./tars/14.tar kubernetesui/metrics-scraper:v1.0.4

~~~

拷贝文件

~~~
#拷贝文件都3台机器
scp -r tars root@192.168.1.11:/root/
scp -r tars root@192.168.1.12:/root/
scp -r tars root@192.168.1.13:/root/
~~~

设置hostname

~~~bash
#机器192.168.2.11执行
hostnamectl --static set-hostname  master

#机器192.168.2.12执行
hostnamectl --static set-hostname  node1

#机器192.168.2.13执行
hostnamectl --static set-hostname  node2

~~~
```
#所有机器上执行，hosts文件追加本地解析记录
cat <<EOF >>  /etc/hosts
192.168.1.11    master
192.168.1.12    node1
192.168.1.13    node2
EOF

cat /etc/hosts
```
## 安装步骤

*以下命令在所有机器执行*

```
#### 关闭防火墙
systemctl stop firewalld.service
systemctl status firewalld.service
systemctl disable firewalld

#### 关闭Swap
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab
echo "vm.swappiness = 0" >> /etc/sysctl.conf 
sysctl -p

#### 关闭selinux
sed -i s/SELINUX=enforcing/SELINUX=disabled/g /etc/selinux/config

#### 设置启动参数
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```
#### 安装docker

```
#离线安装参考《docker生产安装》

#在线安装
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum list docker-ce --showduplicates | sort -r
yum install -y docker-ce-18.06.3.ce-3.el7
systemctl start docker
systemctl enable docker
```

设置docker的cgroupdriver为systemd

~~~
# Set up the Docker daemon
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

#重启
systemctl restart docker
~~~

导入提前准备好的镜像

~~~
#导入镜像脚本
docker load < ./tars/1.tar
docker load < ./tars/2.tar
docker load < ./tars/3.tar
docker load < ./tars/4.tar
docker load < ./tars/5.tar
docker load < ./tars/6.tar
docker load < ./tars/7.tar
docker load < ./tars/9.tar
docker load < ./tars/10.tar
docker load < ./tars/11.tar
docker load < ./tars/12.tar
docker load < ./tars/13.tar
docker load < ./tars/14.tar
~~~

#### 安装kubeadm,kubelet,kubectl

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet-1.19.7 kubeadm-1.19.7 kubectl-1.19.7
systemctl enable kubelet && systemctl start kubelet
```


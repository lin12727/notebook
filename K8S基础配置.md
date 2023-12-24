#   K8S基础配置

## 1.基础环境准备

1.1）启动三台虚拟机，配置节点

~~~
添加主机名与 IP 对应关系
vi /etc/hosts
10.0.2.15 k8s-node1
10.0.2.24 k8s-node2
10.0.2.25 k8s-node3
hostnamectl set-hostname <newhostname>：指定新的 hostname su 切换过来
~~~

1.2）设置iptables(初始化防火墙)（可以写成脚本）

~~~
关闭防火墙：
systemctl stop firewalld NetworkManager
systemctl disable firewalld NetworkManager
 
关闭selinux：
sed -i 's/enforcing/disabled/' /etc/selinux/config
sed -ri  's#(SELINUX=).*#\disabled#' /etc/selinux/config
setenforce 0

再次关闭防火墙：
systemctl disable firewalld  && systemctl stop firewalld
getenforce  0

设置[iptables]
iptables -F
iptables -X
iptables -Z
iptables -P FORWARD ACCEPT
~~~

1.3）关闭swap

~~~
关闭 swap：
swapoff -a 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab 永久 

free -g 验证，swap必须为 0； 
防止开机自动挂载 swap 分区
sed -i  '/swap/ s/^\(.*\)$/#\1/g'  /etc/fstab
~~~

1.4）配置yun源（阿里云源）

~~~
cd /etc/yum.repos.d/

新建一个repo目录，可以保留原来的repo文件

wget http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all   #清除系统所有的yum缓存
yum makecache   # 生成yum缓存

yum list | grep epel-release
epel-release.noarch                         7-11                       extras   
yum install -y epel-release
# epel源安装成功，比原来多了一个epel.repo和epel-testing.repo文件

wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum clean all
yum makecache
~~~

1.5）ntp配置（同步网络）

​	





~~~
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.121.135:6443 --token 6qrbnc.stofuu3mlmjd3sjg \
    --discovery-token-ca-cert-hash sha256:2bfcb3f451195401db8e6e3eb5220d4fdd1f653323066f6402d35b884c8440d2 
~~~


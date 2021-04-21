# K8s初识

## 8s系统架构

从系统架构来看，k8s分为2个节点
Master 控制节点 指挥官
Node 工作节点 干活的

### 1.Master节点组成

API Server ：提供k8s API接口
主要处理Rest操作以及更新Etcd中的对象
是所有资源增删改查的唯一入口。

Scheduler：资源调度器
根据etcd里的节点资源状态决定将Pod绑定到哪个Node上

Controller Manager
负责保障pod的健康存在
资源对象的自动化控制中心，Kubernetes集群有很多控制器。

Etcd
这个是Kubernetes集群的数据库
所有持久化的状态信息存储在Etcd中

master节点组成

![img](http://www.weixuecn.cn/uploads/cj/1690286-20200308180248162-1324894735.png)

API相当于命令,沟通API server

### 2.Node节点的组成

Docker Engine
负责节点容器的管理工作，最终创建出来的是一个Docker容器。

kubelet
安装在Node上的代理服务，用来管理Pods以及容器/镜像/Volume等，实现对集群对节点的管理。

kube-proxy
安装在Node上的网络代理服务，提供网络代理以及负载均衡，实现与Service通讯。

node节点组成

![img](http://www.weixuecn.cn/uploads/cj/1690286-20200308180322469-1537381622.png)

## k8s逻辑架构

```
从逻辑架构上看，k8s分为
Pod 
Controller 
Service   

1.POD 
  POD是k8s的最小单位
  POD的IP地址是随机的，删除POD会改变IP
  POD都有一个根容器
  一个POD内可以由一个或多个容器组成
  一个POD内的容器共享根容器的网络命名空间
  一个POD的内的网络地址由根容器提供

2.Controller
  用来管理POD
  控制器的种类有很多

- RC Replication Controller  控制POD有多个副本
- RS ReplicaSet              RC控制的升级版
- Deployment                 推荐使用，功能更强大，包含了RS控制器
- DaemonSet                  保证所有的Node上有且只有一个Pod在运行
- StatefulSet		       有状态的应用，为Pod提供唯一的标识，它可以保证部署和scale的顺序

3.Service
  NodeIP  
  CluterIP
  POD IP  
```

DaemonSet

![img](http://www.weixuecn.cn/uploads/cj/1690286-20200308180347296-772589184.png)

三个不通的ip

![img](http://www.weixuecn.cn/uploads/cj/1690286-20200308180404723-1241380056.png)

## k8s安装

手动安装

https://www.kubernetes.org.cn/3096.html

### 涉及的命令

```
ipvsadm -Ln			#查看ipvs规则
kubectl get pod			#查看pod信息
kubectl get pod -o wide		#查看pod的详细信息 ip labels
kubectl get pod -n kube-system -o wide	#指定查看某个命名空间的pod的详细信息 
kubectl get nodes		#查看节点信息
kubectl get nodes -o wide	#查看节点详细信息

kubectl -n kube-system edit cm kube-proxy 	#编辑某个资源的配置文件
kubectl -n kube-system logs -f kube-proxy-7cdbn #查看指定命名空间里的指定pod的日志


kubectl create -f kube-flannel.yml 	#根据资源配置清单创建相应的资源
kubectl delete -f kube-flannel.yml   	#删除资源配置清单相应的资源

kubeadm reset 			#重置kubeadm节点
kubeadm token create --print-join-command	#打印出node节点加入master节点的命令
kubeadm join 10.0.0.11:6443 --token uqf018.mia8v3i1zcai19sj     --discovery-token-ca-cert-hash sha256:e7d36e1fb53e59b12f0193f4733edb465d924321bcfc055f801cf1ea59d90aae  #node节点加入master的命令
```

### 实验环境准备：

```
配置信息：
主机名	  IP地址	       推荐配置     勉强配置
node1     10.0.0.11    1C4G40G     1C2G		
node2     10.0.0.12    1C2G40G     1C1G
node3     10.0.0.13    1C2G40G     1C1G

初始化操作：
干净环境
关闭防火墙
关闭SELinux
配置时间同步
配置主机名
配置host解析
更新好阿里源 
确保网络通畅

软件准备：
harbor离线包 

1.防火墙关闭
iptables -nL
systemctl disable firewalld && systemctl stop firewalld
iptables -F
iptables -X
iptables -Z
iptables -nL

2.selinux关闭
getenforce
grep "disabled" /etc/selinux/config 
setenforce 0

3.同不定时任务
* * * * * /sbin/ntpdate time1.aliyun.com > /dev/null 2>&1

4.临时关闭swap分区
swapoff -a
永久关闭
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

### 安装部署docker

```
1.设置国内YUM源
cd /etc/yum.repos.d/
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

2.安装指定的docker版本
yum -y install docker-ce-18.09.7-3.el7 docker-ce-cli-18.09.7

3.设置docker使用阿里云加速
mkdir /etc/docker
cat > /etc/docker/daemon.json <<EOF
    {
      "registry-mirrors": ["https://ig2l319y.mirror.aliyuncs.com"],
      "exec-opts": ["native.cgroupdriver=systemd"]
    }
EOF

4.启动后台进程
systemctl enable docker && systemctl start docker

5.查看docker版本
docker -v
```

### 部署kubeadm和kubelet

```
 部署kubeadm和kubelet

注意！所有机器都需要操作！！！
注意！所有机器都需要操作！！！
注意！所有机器都需要操作！！！

1.设置k8s国内yum仓库
cat >/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF


2.安装kubeadm
yum install -y kubelet-1.16.2 kubeadm-1.16.2 kubectl-1.16.2 ipvsadm

3.设置k8s禁止使用swap
cat > /etc/sysconfig/kubelet<<EOF
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBELET_EXTRA_ARGS="--fail-swap-on=false"
EOF

4.设置内核参数
cat >  /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --system

5.设置kubelet开机启动
systemctl enable kubelet && systemctl start kubelet

6.加载IPVS模块
cat >/etc/sysconfig/modules/ipvs.modules<<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
chmod +x /etc/sysconfig/modules/ipvs.modules
source /etc/sysconfig/modules/ipvs.modules
lsmod | grep -e ip_vs -e nf_conntrack_ipv
```

### node1节点初始化

```
1.初始化命令
https://v1-16.docs.kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/

注意！只在node1节点运行!!!
注意！只在node1节点运行!!!
注意！只在node1节点运行!!!

kubeadm init \
--apiserver-advertise-address=10.0.0.11 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.16.2 \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.2.0.0/16 \
--service-dns-domain=cluster.local \
--ignore-preflight-errors=Swap \
--ignore-preflight-errors=NumCPU

执行完成后会有输出，这是node节点加入k8s集群的命令
===============================================

kubeadm join 10.0.0.11:6443 --token 2an0sn.kykpta54fw6uftgq \
    --discovery-token-ca-cert-hash sha256:e7d36e1fb53e59b12f0193f4733edb465d924321bcfc055f801cf1ea59d90aae
 
===============================================

2.为kubectl准备kubeconfig
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

3.获取node节点信息
[root@node1 ~]# kubectl get nodes
NAME    STATUS     ROLES    AGE   VERSION
node1   NotReady   master   15m   v1.16.

4.支持命令补全
yum install bash-completion -y
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
kubectl completion bash >/etc/bash_completion.d/kubectl

5.设置kube-proxy使用ipvs模式
#执行命令，然后将mode: ""修改为mode: "ipvs"然后保存退出
kubectl edit cm kube-proxy -n kube-system

#重启kube-proxy
kubectl get pod -n kube-system |grep kube-proxy |awk '{system("kubectl delete pod "$1" -n kube-system")}'

#查看pod信息
kubectl get -n kube-system pod|grep "kube-proxy" 

#检查日志，如果出现IPVS rr就表示成功
[root@node1 ~]# kubectl -n kube-system logs -f kube-proxy-7cdbn 
I0225 08:03:57.736191       1 node.go:135] Successfully retrieved node IP: 10.0.0.11
I0225 08:03:57.736249       1 server_others.go:176] Using ipvs Proxier.
W0225 08:03:57.736841       1 proxier.go:420] IPVS scheduler not specified, use rr by default

#检查IPVS规则
[root@node1 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.1.0.1:443 rr
  -> 10.0.0.11:6443               Masq    1      0          0         
TCP  10.1.0.10:53 rr
TCP  10.1.0.10:9153 rr
UDP  10.1.0.10:53 rr
```

显示结果如下,安装成功

![img](http://www.weixuecn.cn/uploads/cj/1690286-20200308180448396-1719132416.png)

```
kubeadm join 10.0.0.11:6443 --token ahpc8m.eo6crouhvqw5p8e3 \
    --discovery-token-ca-cert-hash sha256:b02959c21fddd87430985c7ee3a421c5c81c8d57434e258891a07f1b7faec543 
```

为什么删除kube-proxy

![img](http://www.weixuecn.cn/uploads/cj/1690286-20200308180612077-1642130649.png)

### node1部署网络插件

```
部署网络插件
1.部署Flannel网络插件
git clone --depth 1 https://github.com/coreos/flannel.git

2.修改资源配置清单
cd /root/flannel-master/Documentation
vim kube-flannel.yml
egrep -n "10.2.0.0|mirror|eth0" kube-flannel.yml
128:      "Network": "10.2.0.0/16",
172:        image: quay-mirror.qiniu.com/coreos/flannel:v0.11.0-amd64
186:        image: quay-mirror.qiniu.com/coreos/flannel:v0.11.0-amd64
192:        - --iface=eth0   #新增加

3.应用资源配置清单
kubectl create -f kube-flannel.yml

4.检查pod运行状态，等一会应该全是running
[root@node1 ~]# kubectl -n kube-system get pod
NAME                            READY   STATUS    RESTARTS   AGE
coredns-58cc8c89f4-bzlkw        1/1     Running   0          77m
coredns-58cc8c89f4-sgs44        1/1     Running   0          77m
etcd-node1                      1/1     Running   0          76m
kube-apiserver-node1            1/1     Running   0          76m
kube-controller-manager-node1   1/1     Running   0          76m
kube-flannel-ds-amd64-cc5g6     1/1     Running   0          3m10s
kube-proxy-7cdbn                1/1     Running   0          23m
kube-scheduler-node1            1/1     Running   0          76m
```

修改错配置文件

解决办法:

```
查看pod信息
kubectl get -n kube-system pod|grep "kube-proxy" 

kubectl -n kube-system logs -f +pod对应的信息
503报错
防火墙没关

在目录里操作：
第一步 kubectl delete -f kube-flannel.yml
第二步  cp kube-flannel.yml /opt
第三部 rm -rf kube-flannel.yml
然后把老师拖进来任意目录下
最后执行kubectl create -f kube-flannel.yml
查看kubectl -n kube-system get pod
```

### node2-3节点执行

```
1.在master节点上输出增加节点的命令
kubeadm token create --print-join-command

2.在node2和node3节点执行加入集群的命令 每个人的token不一样
kubeadm join 10.0.0.11:6443 --token uqf018.mia8v3i1zcai19sj     --discovery-token-ca-cert-hash sha256:e7d36e1fb53e59b12f0193f4733edb465d924321bcfc055f801cf1ea59d90aae 

3.在node1节点上查看状态
kubectl get nodes

4.给其他节点打标签
kubectl label nodes node2 node-role.kubernetes.io/node=
kubectl label nodes node3 node-role.kubernetes.io/node=

5.再次查看节点状态
[root@node1 ~]# kubectl get nodes
NAME    STATUS   ROLES    AGE    VERSION
node1   Ready    master   171m   v1.16.2
node2   Ready    node     27m    v1.16.2
node3   Ready    node     27m    v1.16.2
```

## 常用资源类型

```
1.工作负载类型
  RC  ReplicaController
  RS  ReplicaSet 
  DP  Deployment
  DS  DaemonSet  

2.服务发现及负载均衡
  Service 
  Ingress 

3.配置与存储资源
  ConfigMap 存储配置文件
  Secret    存储用户字典

4.集群级别资源
  Namespace
  Node
  Role
  ClusterRole
  RoleBinding
  ClusterRoleBinding
```

### 资源配置清单

```
1.创建资源的方法
  apiserver仅能接受json格式的资源定义 
  yaml格式提供的清单，apiserver可以自动将其转换为json格式再提交

2.资源清单介绍
  查看资源清单所需字段
  kubectl explain pod

  资源清单字段介绍
  apiVersion: v1  #属于k8s哪一个API版本或组
  kind: Pod	  #资源类型
  metadata:	  #元数据，嵌套字段
  spec:		  #定义容器的规范，创建的容器应该有哪些特性
  
  status: 	  #只读的，由系统控制，显示当前容器的状态


  json嵌套
{ 1级：
     { 2级：
          { 3级：Value

          }
      }
}


3.查看资源清单嵌套的命令
  kubectl explain pod
  kubectl explain pod.spec
  kubectl explain pod.spec.volumes

4.使用命令行创建一个pod
  kubectl create deployment nginx --image=nginx:alpine
  kubectl get pod -o wide


5.将刚才创建的pod配置到处成yaml格式
  kubectl get pod -o yaml > nginx-pod.yaml
  
  精简资源清单，删掉不需要的配置
cat nginx-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80

  json格式写法：
{
 apiVersion: "v1",
 kind: "Pod",
 metadata: 
   {
      name: "nginx", 
      labels: 
        {
           app: "nginx"
        }        
    }
 spec: 
   {
     containers:
       {
         name: "nginx",
         image: "nginx:alpine",
         imagePullPolicy: "IfNotPresent"
       }
   }    
}       

  删除命令行创建的资源
  kubectl delete deployments.apps nginx

  应用资源配置清单
  kubectl create -f nginx-pod.yaml

  查看pod信息
  kubectl get pod -o wide

  查看pod详细信息
  kubectl describe pod nginx
  
  
6.POD资源清单总结
  声明式管理 我想运行一个Nginx k8s帮你干活 
  
apiVersion: v1	#api版本
kind: Pod	#资源类型
metadata:	#元数据
  name: nginx	#元数据名称
  labels:	#pod标签
    app: nginx   
spec:		#容器定义
  containers:   #容器的特性
  - name: nginx #容器名称
    image: nginx:alpine #容器的镜像名称
    imagePullPolicy: IfNotPresent  #容器的拉取策略
    ports:      #容器端口
    - name: http 
      containerPort: 80	 #容器暴露的端口
```

### 升级版本

```
改变资源配置清单版本
image: "nginx:alpine",
执行
kubectl apply -f nginx-pod.yaml 
```

### 容器打标签

```
1.标签说明
  一个标签可以给多个POD使用
  一个POD也可以拥有多个标签
    
2.查看POD标签
  kubectl get pod --show-labels

3.添加标签方法
方法1:直接编辑资源配置清单：
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    release: beta
----------------------
方法2:命令行打标签
kubectl label pods nginx release=beta
kubectl label pods nginx job=linux
kubectl get pod --show-labels  


4.删除标签
kubectl label pod nginx job- 
kubectl get pod --show-labels


5.实验: 生成2个POD，打上不同的标签，然后根据标签选择
kubectl create deployment nginx --image=nginx:1.14.0
kubectl get pod --show-labels
kubectl label pods nginx-xxxxxxxx release=stable
kubectl get pod --show-labels

根据条件查看
kubectl get pods -l release=beta --show-labels 
kubectl get pods -l release=stable --show-labels 


根据条件删除
kubectl delete pod -l app=nginx
```

### Node打标签

```
Node打标签

1.查看node的标签
kubectl get node --show-labels

2.给node打标签
kubectl label nodes node2 CPU=Xeon
kubectl label nodes node3 disktype=ssd

3.编辑POD资源配置清单，使用node标签选择器
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.0
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
  nodeSelector:
    #CPU: Xeon
    disktype: SSD

4.删除容器重新创建
kubectl delete pod nginx
kubectl create -f nginx-pod.yaml

5.查看结果
kubectl get pod -o wide

6.删除节点标签
```

## 部署harbor作为k8s镜像仓库

```
部署harbor作为k8s镜像仓库

1.实验目标
  部署k8s私有镜像仓库harbor
  把demo小项目需要的镜像上传到harbor上
  修改demo项目的资源配置清单，镜像地址修改为harbor的地址

2.在node1上安装harbor
cd /opt/
tar zxf harbor-offline-installer-v1.9.0-rc1.tgz
cd harbor/

3.编辑harbor配置文件
vim harbor.yml
hostname: 10.0.0.11
port: 8888
harbor_admin_password: 123456
data_volume: /data/harbor

4.执行安装
yum install docker-compose -y
./install.sh

5.浏览器访问
http://10.0.0.11:8888
admin
123456

6.建立镜像仓库
这里有2种访问级别：
公开：任何人都可以直接访问并下载镜像
私有：登陆授权后才允许下载镜像

如果创建私有仓库，k8s是不能直接下载的，需要配置安全文件


7.所有节点都配置docker信任harbor仓库并重启docker
cat >/etc/docker/daemon.json <<EOF
    {
      "registry-mirrors": ["https://ig2l319y.mirror.aliyuncs.com"],
      "exec-opts": ["native.cgroupdriver=systemd"],
      "insecure-registries" : ["http://10.0.0.11:8888"]
    }
EOF
systemctl restart docker

###############注意###############
在node1上重启docker后，如果harbor不正常了，重启harbor即可
cd /opt/harbor
docker-compose restart 

8.docker登陆harbor
docker login 10.0.0.11:8888
admin
123456

9.在相对的节点上下载镜像修改tag并push到harbor上
查看节点分布
kubectl get pods -o wide
在node2上执行
docker login 10.0.0.11:8888
docker tag kubeguide/tomcat-app:v1 10.0.0.11:8888/k8s/tomcat-app:v1
docker tag mysql:5.7 10.0.0.11:8888/k8s/mysql:5.7

docker push 10.0.0.11:8888/k8s/tomcat-app:v1
docker push 10.0.0.11:8888/k8s/mysql:5.7 

10.节点上删除镜像
docker rmi mysql:5.7 
docker rmi kubeguide/tomcat-app:v1
docker rmi 10.0.0.11:8888/k8s/mysql:5.7 
docker rmi 10.0.0.11:8888/k8s/tomcat-app:v1

11.node1节点删除以前的demo项目
kubectl delete -f tomcat-demo.yaml

12.修改demo项目的资源配置清单里的镜像地址 直接第15开始
image: 10.0.0.11:8888/k8s/mysql:5.7
image: 10.0.0.11:8888/k8s/tomcat-app:v1

13.应用资源配置清单
kubectl create -f tomcat-demo.yaml 

14.报错
此时查看pod状态会发现镜像拉取失败了
[root@node1 ~/demo]# kubectl get pod
NAME                     READY   STATUS             RESTARTS   AGE
mysql-7d746b5577-jcs7q   0/1     ImagePullBackOff   0          8s
myweb-764df5ffdd-fptn9   0/1     ImagePullBackOff   0          8s
myweb-764df5ffdd-pmkz7   0/1     ErrImagePull       0          8s

查看pod创建的详细信息
kubectl describe pod mysql-7d746b5577-jcs7q 

关键报错信息：
Failed to pull image "10.0.0.11:8888/k8s/mysql:5.7": rpc error: code = Unknown desc = Error response from daemon: pull access denied for 10.0.0.11:8888/k8s/mysql, repository does not exist or may require 'docker login'


15.查看docker登陆的密码文件
docker login 10.0.0.11:8888
cat /root/.docker/config.json

16.将docker密码文件解码成base64编码
[root@node1 ~/demo]# cat /root/.docker/config.json|base64
ewoJImF1dGhzIjogewoJCSIxMC4wLjAuMTE6ODg4OCI6IHsKCQkJImF1dGgiOiAiWVdSdGFXNDZN
VEl6TkRVMiIKCQl9Cgl9LAoJIkh0dHBIZWFkZXJzIjogewoJCSJVc2VyLUFnZW50IjogIkRvY2tl
ci1DbGllbnQvMTguMDkuOSAobGludXgpIgoJfQp9

17.创建并应用docker登陆的Secret资源(master节点执行)
注意！！！
1.dockerconfigjson: xxx直接写base64的编码，不需要换行
2.base64编码是一整行，不是好几行
3.最后的type字段不能少

cat >harbor-secret.yaml<<EOF 
apiVersion: v1
kind: Secret
metadata:
  name: harbor-secret
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxMC4wLjAuMTE6ODg4OCI6IHsKCQkJImF1dGgiOiAiWVdSdGFXNDZNVEl6TkRVMiIKCQl9Cgl9LAoJIkh0dHBIZWFkZXJzIjogewoJCSJVc2VyLUFnZW50IjogIkRvY2tlci1DbGllbnQvMTguMDkuOSAobGludXgpIgoJfQp9
type: kubernetes.io/dockerconfigjson
EOF

kubectl create -f harbor-secret.yaml
kubectl get secrets

18.修改demo资源配置清单，添加拉取镜像的参数
查看命令帮助
kubectl explain deployment.spec.template.spec.imagePullSecrets

修改文件
image: 10.0.0.11:8888/k8s/mysql:5.7
image: 10.0.0.11:8888/k8s/tomcat-app:v1
----------------------------
      imagePullSecrets: 
      - name: harbor-secret
  添加两处 
----------------------------

19.应用资源配置清单并查看
kubectl create -f tomcat-demo.yaml 
kubectl get pod -o wide

20.浏览器查看
http://10.0.0.11:30001/demo
```
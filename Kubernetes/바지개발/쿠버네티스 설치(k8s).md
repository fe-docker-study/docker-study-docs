#  쿠버네티스 설치(k8s)

- 참고 : 컨테이너 인프라 환경 구축을 위한 쿠버네티스/도커
  https://github.com/sysnet4admin/_Book_k8sInfra
- 사전 설치 : VitualBox, Vagrant



## 컨테이너 인프라 환경

- 리눅스 운영 체제의 커널 하나에서 여러 개의 컨테이너가 격리된 상태로 실행되는 인프라 환경

  ※ 컨테이너는 하나 이상의 목적을 위해 독립적으로 작동하는 프로세스

- 가상화 환경에서는 각각의 가상 머신이 모두 독립적인 운영체제 커널을 가지고 있어야 하기 떄문에 자원소모가 많음. 그러나 컨테이너 인프라 환경은 운영 체제 커널 하나에 컨테이너 여러개가 격리된 형태로 실행되기 때문에 자원을 효율적으로 사용할 수 있고 거치는 단계가 적어서 속도가 빠름

  

## 쿠버네티스

- 다양한 형태의 쿠버네티스가 지속적으로 계속 발전되고 있어서 컨테이너 오케스트레이션을 넘어 IT 인프라 자체를 컨테이너화하고, 컨테이너화된 인프라 제품군을 쿠버네티스 위에서 동작할 수 있게 만들어줌. 즉 거의 모든 벤더와 오픈 소스 진영 모두에서 쿠버네티스를 지원하고 그에 맞게 통합개발하고 있음

  

## 쿠버네티스 구성방법

### **관리형 쿠버네티스**

- 퍼블릭 클라우드 업체에서 제공하는 관리형 쿠버네티스인 EKS(Amazon Elastic Kubernetes Service), AKS(Azure Kubernetes Service), GKE(Google Kubernetes Service) 등을 사용
- 구성이 이미 다 갖춰져 있고 마스터 노드를 클라우드 업체에서 관리

### **설치형 쿠버네티스**

- 수세의 Rancher, 레드햇의 OpenShift와 같은 플랫폼에서 제공하는 설치형 쿠버네티스를 사용
- 유료

### **구성형 쿠버네티스**

- 사용하는 시스템에 쿠버네티스 클러스터를 자동으로 구성해주는 솔루션을 사용

- 주요 4가지 솔루션-  kubeadm, kops(Kubernetes operations), KRIB(Kubernetes Rebar Integrated Bootstrap), Kuberspray

- kubeadm은 사용자가 변경하기 수월하고 온프레미스(On-Premises)와 클라우드를 모두 지원하며, 배우기 쉬움. 가장 널리 알려져 있음

  

## VirtualBox 설치

- virtualBox: 가상화 소프트웨어

- https://www.virtualbox.org/wiki/Downloads

  

## 베이그런트 설치하기

- 베이그런트 : 사용자의 요구에 맞게 시스템 자원을 할당, 배치, 배포해 두었다가 필요할 때 시스템을 사용할 수 있는 상태로 만들어 줌(가상머신 동작 제어 도구)

- https://www.vagrantup.com/downloads

- cmd에서 베이그런트 초기화 명령을 실행해 프로비저닝에 필요한 기본코드 생성

  ```java
  > cd c:\\HashiCorp
  c:\\HashiCorp>vagrant init
  A `Vagrantfile` has been placed in this directory. You are now
  ready to `vagrant up` your first virtual environment! Please read
  the comments in the Vagrantfile as well as documentation on
  `vagrantup.com` for more information on using Vagrant.
  ```

  

## kubeadm로 쿠버네티스 구성

### vagrantfile

- 베이그런트 프로비저닝을 위한 정보를 담고 있는 메인 파일
- 명령 프롬프트에서 Vagrantfile이 있는 경로에서 vagrant up을 하면 현재 호스트 내부에 Vagrantfile에 정의된 가상 머신들을 생성하고, 생성한 가상 머신에 쿠버네티스 클러스트를 구성하기 위한 파일들을 호출해 쿠버네티스 클러스터를 자동으로 구성

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  N = 3 # max number of worker nodes
  Ver = '1.18.4' # Kubernetes Version to install

  #=============#
  # Master Node #
  #=============#

    config.vm.define "m-k8s" do |cfg|
      cfg.vm.box = "sysnet4admin/CentOS-k8s"
      cfg.vm.provider "virtualbox" do |vb|
        vb.name = "m-k8s(github_SysNet4Admin)"
        vb.cpus = 2
        vb.memory = 3072
        vb.customize ["modifyvm", :id, "--groups", "/k8s-SgMST-1.13.1(github_SysNet4Admin)"]
      end
      cfg.vm.host_name = "m-k8s"
      cfg.vm.network "private_network", ip: "192.168.1.10"
      cfg.vm.network "forwarded_port", guest: 22, host: 60010, auto_correct: true, id: "ssh"
      cfg.vm.synced_folder "../data", "/vagrant", disabled: true 
      cfg.vm.provision "shell", path: "config.sh", args: N
      # args: [ Ver, "Main" ] 은 버전 정보(Ver)와 Main이라는 문자를 install_pkg.sh로 넘김. 
      # Ver 변수는 해당 버전의 쿠버네티스를 설치하게 함
      # Main은 install_pkg.sh에서 조건문으로 처리해 마스터 노드에만 전체 실행코드 다운
      cfg.vm.provision "shell", path: "install_pkg.sh", args: [ Ver, "Main" ]
      cfg.vm.provision "shell", path: "master_node.sh"
    end

  #==============#
  # Worker Nodes #
  #==============#

  (1..N).each do |i|
    config.vm.define "w#{i}-k8s" do |cfg|
      cfg.vm.box = "sysnet4admin/CentOS-k8s"
      cfg.vm.provider "virtualbox" do |vb|
        vb.name = "w#{i}-k8s(github_SysNet4Admin)"
        vb.cpus = 1
        vb.memory = 2560
        vb.customize ["modifyvm", :id, "--groups", "/k8s-SgMST-1.13.1(github_SysNet4Admin)"]
      end
      cfg.vm.host_name = "w#{i}-k8s"
      cfg.vm.network "private_network", ip: "192.168.1.10#{i}"
      cfg.vm.network "forwarded_port", guest: 22, host: "6010#{i}", auto_correct: true, id: "ssh"
      cfg.vm.synced_folder "../data", "/vagrant", disabled: true
      cfg.vm.provision "shell", path: "config.sh", args: N
      cfg.vm.provision "shell", path: "install_pkg.sh", args: Ver
      cfg.vm.provision "shell", path: "work_nodes.sh"
    end
  end

end
```



### config.sh

- kubeadm으로 쿠버네티스를 설치하기 위한 사전 조건을 설정하는 스크립트 파일

```bash
#!/usr/bin/env bash

# vim configuration 
echo 'alias vi=vim' >> /etc/profile

# swapoff -a to disable swapping
# 쿠버네티스의 설치 요구 조건을 맞추기 위해 스왑되지 않도록 설정
swapoff -a
# sed to comment the swap partition in /etc/fstab
# 시스템이 다시 시작되더라도 스왑되지 않도록 설정
sed -i.bak -r 's/(.+ swap .+)/#\\1/' /etc/fstab

# kubernetes repo
# 쿠버네티스의 리포지터리를 설정하기 위한 경로가 너무 길어지지 않게 변수로 처리
gg_pkg="packages.cloud.google.com/yum/doc" # Due to shorten addr for key
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://${gg_pkg}/yum-key.gpg <https://$>{gg_pkg}/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
# selinux가 제한적으로 사용되지 않도록 permissive 모드로 변경
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# RHEL/CentOS 7 have reported traffic issues being routed incorrectly due to iptables bypassed
# 브리지 네트워크를 통과하는 IPv4와 IPv6의 패킷을 iptables가 관리하게 설정. Pod의 통신을 iptables로 제어
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# br_netfilter 커널 모듈을 사용해 브리지로 네트워크를 구성 
# 이때 IP 마스커레이드(Masquerade)를 사용해 내부 네트워크와 외부 네트워크를 분리
# IP 마스커레이드는 쉽게 설명하면 커널에서 제공하는 NAT(네트워크 주소 변환) 기능
# 실제로는 br_netfilter를 적용함으로써 앞서 적용한 iptables가 활성화
modprobe br_netfilter

# local small dns & vagrant cannot parse and delivery shell code.
# 쿠버네티스 안에서 노드 간 통신을 이름으로 할 수 있도록 각 노드의 호스트 이름과 IP를 /etc/hosts에 설정  
echo "192.168.1.10 m-k8s" >> /etc/hosts
for (( i=1; i<=$1; i++  )); do echo "192.168.1.10$i w$i-k8s" >> /etc/hosts; done

# config DNS
# 외부와 통신할 수 있게 DNS 서버를 지정
cat <<EOF > /etc/resolv.conf
nameserver 1.1.1.1 #cloudflare DNS
nameserver 8.8.8.8 #Google DNS
EOF
```



### install_pkg.sh

- 클러스터를 구성하기 위해서 가상머신에 설치돼야하는 의존성 패키지를 명시
- 환경설정에 필요한 소스코드를 특정 가상머신(m-k8s) 내부에 내려받도록 설정

```bash
#!/usr/bin/env bash

# install packages 
yum install epel-release -y
yum install vim-enhanced -y
# 깃허브 이용을 위해 git을 설치
yum install git -y

# install docker 
# 쿠버네티스를 관리하는 컨테이너를 설치하기 위해 도커를 설치/구동
yum install docker -y && systemctl enable --now docker

# install kubernetes cluster 
yum install kubectl-$1 kubelet-$1 kubeadm-$1 -y
systemctl enable --now kubelet

# git clone _Book_k8sInfra.git 
# 깃에서 코드를 clone하여 실습을 진행할 루트 홈디렉토리(/root)로 옮김
# 배시 스크립트(.sh)를 찾아서 바로 실행 가능한 상태가 되도록 chmod 700으로 설정
if [ $2 = 'Main' ]; then
  git clone <https://github.com/sysnet4admin/_Book_k8sInfra.git>
  mv /home/vagrant/_Book_k8sInfra $HOME
  find $HOME/_Book_k8sInfra/ -regex ".*\\.\\(sh\\)" -exec chmod 700 {} \\;
fi
```



### master_node.sh

- 1개의 가상머신(m-k8s)을 쿠버네티스 마스터 노드로 구성하는 스크립트
- 쿠버네티스 클러스터를 구성할 때 곡 선택해야하는 컨테이너 네트워크 인터페이스(CNI, Container Network Interface)도 함께 구성

```bash
#!/usr/bin/env bash

# init kubernetes 
# kubeadm을 통해 쿠버네티스의 워커 노드를 받아들일 준비
# 먼저 토큰을 임의로 지정하고 ttl(time to live, 유지되는 시간)을 0으로 설정해서 기본값인 24시간 후에 토큰이 계속 유지되게 함
# 쿠버네티스가 자동으로 컨테이너에 부여하는 네트워크를 172.16.0.0/16(172.16.0.1~172.16.255.254)로 제공
# 워커 노드가 접속하는 API 서버의 IP를 192.168.1.10으로 지정해 워커 노드들이 자동으로 API서버에 연결되게 함
kubeadm init --token 123456.1234567890123456 --token-ttl 0 \\
--pod-network-cidr=172.16.0.0/16 --apiserver-advertise-address=192.168.1.10 

# config for master node only 
# 마스터 노드에서 현재 사용자가 쿠버네티스를 정상적으로 구동할 수 있게 설정 파일을 루트의 홈디텍토리(/root)에 복사
# 쿠버네티스를 이용할 사용자에게 권한을 줌
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# config for kubernetes's network 
# 컨테이너 네트워크 인터페이스인 캘리코(Calico)의 설정을 적용해 쿠버네티스의 네트워크를 구성
kubectl apply -f \\
<https://raw.githubusercontent.com/sysnet4admin/IaC/master/manifests/172.16_net_calico.yaml>
```



### work_node.sh

- 워커노드를 구성하는 스크립트
- 마스터 노드에 구성된 클러스터에 조인이 필요한 정보가 모든 코드화돼 스크립트를 실행하기만 하면 편하게 워커 노드로서 쿠버네티스 클러스터에 조인

```bash
#!/usr/bin/env bash

# config for work_nodes only 
# kubeadm을 이용해 쿠버네티스 마스터 노드에 접속
# 연결에 필요한 토큰은 기존에 마스터 노드에서 생성한 임의의 토큰을 사용
# 간단하게 구성하기 위해 `--discovery-token-unsafe-skip-ca-verification`으로 인증을 무시하고
# API 서버 주소인 192.168.1.10으로 기보 포트 번호인 6443번 포트에 접속하도록 설정
kubeadm join --token 123456.1234567890123456 \\
             --discovery-token-unsafe-skip-ca-verification 192.168.1.10:6443
```





## 쿠버네티스 클러스터 자동 구성



### Vagrantfile을 읽어 들여 프로비저닝을 진행

- vagrant up

  ```bash
  C:\\HashiCorp>vagrant up
  Bringing machine 'm-k8s' up with 'virtualbox' provider...
  Bringing machine 'w1-k8s' up with 'virtualbox' provider...
  Bringing machine 'w2-k8s' up with 'virtualbox' provider...
  Bringing machine 'w3-k8s' up with 'virtualbox' provider...
  ==> m-k8s: Importing base box 'sysnet4admin/CentOS-k8s'...
  ==> m-k8s: Matching MAC address for NAT networking...
  ==> m-k8s: Checking if box 'sysnet4admin/CentOS-k8s' version '0.7.4' is up to date...
  (생략)
  ```



### 터미널로 가상머신 접속

- 서버 이름 :  m-k8s, w1-k8s, w2-k8s, w3-k8s
- hostname : 127.0.0.1
- username : root
- password : vagrant
- port : 60010, 60101, 60102, 60103



### m-k8s에 접속해 마스터 노드와 워커 노드 정상 생성과 연결 확인

- kubectl get nodes

```bash
[root@m-k8s ~]# kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
m-k8s    Ready    master   25m   v1.18.4
w1-k8s   Ready    <none>   23m   v1.18.4
w2-k8s   Ready    <none>   20m   v1.18.4
w3-k8s   Ready    <none>   18m   v1.18.4
```



### 설치된 쿠버네티스 구성요소 확인

- kubectl get pods --all-namespaces
- 쿠버네티스 클러스터를 이루는 구성 요소들은 파드 형태로 이뤄짐

```bash
[root@m-k8s ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-99c9b6f64-9sqgm   1/1     Running   0          29m
kube-system   calico-node-54cld                         1/1     Running   0          22m
kube-system   calico-node-5j2xg                         1/1     Running   0          27m
kube-system   calico-node-8vbsr                         1/1     Running   0          29m
kube-system   calico-node-jfffd                         1/1     Running   0          24m
kube-system   coredns-66bff467f8-fnv7q                  1/1     Running   0          29m
kube-system   coredns-66bff467f8-jj9xt                  1/1     Running   0          29m
kube-system   etcd-m-k8s                                1/1     Running   0          29m
kube-system   kube-apiserver-m-k8s                      1/1     Running   0          29m
kube-system   kube-controller-manager-m-k8s             1/1     Running   0          29m
kube-system   kube-proxy-6frpb                          1/1     Running   0          27m
kube-system   kube-proxy-pr7fj                          1/1     Running   0          24m
kube-system   kube-proxy-rbpw5                          1/1     Running   0          22m
kube-system   kube-proxy-rsd5j                          1/1     Running   0          29m
kube-system   kube-scheduler-m-k8s                      1/1     Running   0          29m
```



### 워커 노드에서 kubectl  사용하기

- kubectl이 어디에 있더라도  API 서버를 통해 쿠버네티스에 명령을 내릴 수 있음. 따라서 API 서버의 접속정보만 있다면 쿠버네티스 클러스터에 명령을 내릴 수 있음
- scp [root@192.168.1.10](mailto:root@192.168.1.10):/etc/kubernetes/admin.conf .
- 쿠버네티스 클러스터 정보를 마스터 노드에서 secure copy 명령을 현재 디렉터리로 받아옴

```bash
[root@w3-k8s ~]# scp root@192.168.1.10:/etc/kubernetes/admin.conf .
The authenticity of host '192.168.1.10 (192.168.1.10)' can't be established.
ECDSA key fingerprint is SHA256:l6XikZFgOibzSygqZ6+UYHUnEmjFEFhx7PpZw0I3WaM.
ECDSA key fingerprint is MD5:09:74:43:ef:38:3e:36:a1:7e:51:76:1a:ac:2d:7e:0c.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.10' (ECDSA) to the list of known hosts.
root@192.168.1.10's password: 
admin.conf                                                               100% 5452     7.0MB/s   00:00    
[root@w3-k8s ~]# kubectl get nodes --kubeconfig admin.conf
NAME     STATUS   ROLES    AGE   VERSION
m-k8s    Ready    master   67m   v1.18.4
w1-k8s   Ready    <none>   64m   v1.18.4
w2-k8s   Ready    <none>   62m   v1.18.4
w3-k8s   Ready    <none>   60m   v1.18.4
```
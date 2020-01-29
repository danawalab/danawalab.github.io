---
layout: post
title:  "쿠버네티스 고가용성 클러스터 구성"
description: "쿠버네티스는 마스터 노드에 내부적으로 API-SERVER를 가지고 있어 마스터 노드를 통해 워커 노드로 통신이 전달됩니다. 대량에 통신량이 발생하게 되면 마스터 노드는 부하를 많이 받아 장애 발생 가능성이 커지게 됩니다. 마스터 노드를 고가용성 클러스터로 연결하게 되면 통신량을 분산시킬 수 있고, 일부가 마스터 노드에서 장애가 발생시 워커 노드의 운영 시스템에는 영향을 줄일 수 있습니다."
date:   2020.01.28.
writer: "김준우"
categories: Kubernetes
---



> ## 소개

쿠버네티스는 마스터 노드에 내부적으로 API-SERVER를 가지고 있어 마스터 노드를 통해 워커 노드로 통신이 전달됩니다. 대량에 통신량이 발생하게 되면 마스터 노드는 부하를 많이 받아 장애 발생 가능성이 커지게 됩니다. 마스터 노드를 고가용성 클러스터로 연결하게 되면 통신량을 분산시킬 수 있고, 일부가 마스터 노드에서 장애가 발생시 워커 노드의 운영 시스템에는 영향을 줄일 수 있습니다.


> ## 구성 환경

아래 사양으로 총 3대 서버를 준비합니다.   

OS: CentOS 7   
CPU: 2   
RAM: 4   
Storage: 100G   


> ## 구성목표

1. 마스터 노드 3개를 클러스터구성
2. 가상 네트워크(weave) 적용
3. ceph storage 적용   


> ## 도커&쿠버네티스 설치
CentOS 패키지업데이트와 방화벽을 정지및 스왑을 종료합니다.

```
$ sudo yum update -y
$ sudo systemctl disable firewalld && sudo systemctl stop firewalld
$ sudo selinux disabled
$ sudo setenforce 0
$ sudo iptables piv4 forward
$ sudosudo bash -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
$ sudo swapoff -a
$ sudo sed -i '/swap/d' /etc/fstab
```


**운영 환경에서는 아래 포트를 오픈하여 진행합니다.*

| 노드 | 포트 | TCP/UDP |
| --- |--- |---|
| 마스터 | 6443, 2379-2380, 10250, 10251, 10252 | TCP |
| 마스터, 워커 | 6783  | TCP |
| 마스터, 워커 | 6783, 6784  | UDP |
| 워커 | 10250, 30000-32767  | TCP |
| 로드 밸런서 | 26443  | TCP |


### 호스트네임 설정

**호스트네임은 서로 다르게 지정해야합니다.*
```
$ sudo hostnamectl set-hostname node1

$ sudo hostnamectl set-hostname node2

$ sudo hostnamectl set-hostname node3
```


### 도커 설치
도커 레파지토리를 등록하여 yum 으로 설치 진햅니다.   
```
$ sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

$ sudo yum install docker-ce docker-ce-cli containerd.io

$ sudo systemctl start docker

```

**설치후 daemon.json을 추가하여 cgroupdriver를 systemd 옵션을 사용할 수 있도록 합니다.*
```
$ sudo cat <<EOF > /etc/docker/daemon.json
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

$ sudo systemctl daemon-reload
$ sudo systemctl enable docker
$ sudo systemctl restart docker
```


### 쿠버네티스 설치
쿠버네티스의 레파지토리 저장소를 등록하여 yum 으로 kubelet, kubeadm, kubectl을 설치합니다.

- kubeadm: 클러스터 관련 작업시 사용됩니다.
- kubectl: 클러스터의 서비스및 pod 관리시 사용됩니다.
- kubelet: 클러스터 에이전트역할을 합니다.

```
$ sudo cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg [https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg](https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg)
EOF

$ sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

$ sudo systemctl enable kubelet
$ sudo systemctl restart kubelet
```



node1, node2, node3에 도커와 쿠버네티스가 설치를 하였다면 아래 명령어를 통해 확인해볼 수 있습니다.
```
$ sudo docker verison
$ kubectl version
```


> ## 로드밸런서 설치

쿠버네티스에서 공식문서에서 사용되고 있는 HAProxy를 사용하도록 하겠습니다.   
테스트환경이기 때문에 keepalive및 failover 기능 사용없이 진행하도록 하겠습니다.  


### 노드1에만 haproxy 를 설치하도록 하겠습니다.
```
$ sudo yum install haproxy -y
```

### haproxy 로드밸런싱 설정
node1 IP의 26443포트로 전달받은 데이터를 node1 ~ node3의 6443 포트로 포워드 시켜줍니다.   
balance 방식은 라운드로빈으로 순차적으로 접근하도록 하겠습니다.

```
$ sudo cat <<EOF >> /etc/haproxy/haproxy.cfg
frontend kubernetes-master-lb
bind 0.0.0.0:26443
option tcplog
mode tcp
default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
mode tcp
balance roundrobin
option tcp-check
option tcplog
server node1 <node1 IP>:6443 check
server node2 <node2 IP>:6443 check
server node3 <node3 IP>:6443 check
EOF

$ sudo systemctl restart haproxy
```

### haproxy 확인
해당 포트가 오픈되어 있는지 확인합니다.   
포트가 오픈되어 있어도 접근이 안될때는 방화벽을 확인해보아야 합니다.
```
netstat -an|grep 26443
```


> ## 쿠버네티스 클러스터

클러스터 생성할때 --upload-certs, --control-plane-endpoint 플래그를 추가해야 인증서가 자동 배포가 되고, 마스터 노드 조인 명령어가 출력됩니다.

```
$ sudo kubeadm init --control-plane-endpoint "<노드1 DNS/IP or LoadBalancer DNS/IP>:26443" \
                --upload-certs \
                --pod-network-cidr "10.244.0.0/16"
```


### config 생성
kubectl 사용하기 위해서는 사용자 디렉토리 하위에 .kube/config 가 필요합니다. kubeadm init 명령어를 진행하고 마지막에 로그를 잘보면 config를 생성하는 명령어가 적혀있습니다.

```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> ## 쿠버네티스 클러스터
노드1에서 클러스터 생성시 kubeadm join 명령어가 출력되는데 --control-plane --certificate-key 플래그 포함하여 명령어를 실행하면 마스터 노드로 연결이되고, 플래그는 추가하지 않고 사용하게 되면 워커노드로 연결됩니다. 

### 노드2 클러스터 연결
```
$ sudo kubeadm join 192.168.248.251:26443 --token 06glbc.s0yaqonyajs95ez3 \
    --discovery-token-ca-cert-hash sha256:379ff0daa2cffa3f6581ae25c96aa3d6e4a9af43df92b8d0ac5a4dbb4c7e5547 \
    --control-plane --certificate-key 8eb7fa9a38ca1579c3fb61e0a1106935c4e9dfd81cbad259121f223f0adf555f
```

### config 생성
노드2의 사용자에서도 kubectl 명령어를 사용하기 위해선 config를 생성해야합니다.
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### 노드3 클러스터 연결
노드3에서도 노드2와 동일하게 연결을 진행합니다.
```
$ sudo kubeadm join 192.168.248.251:26443 --token 06glbc.s0yaqonyajs95ez3 \
    --discovery-token-ca-cert-hash sha256:379ff0daa2cffa3f6581ae25c96aa3d6e4a9af43df92b8d0ac5a4dbb4c7e5547 \
    --control-plane --certificate-key 8eb7fa9a38ca1579c3fb61e0a1106935c4e9dfd81cbad259121f223f0adf555f
```
### config 생성
노드3에서도 동일하게 config를 생성해야합니다.
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


> ## 클러스터 구성 확인

노드가 연결이 정상적으로 되었다면 아래 명령어를 실행하였을때 노드가 전부 보여야합니다.
```
$ kubectl get node
```

마스터 노드에는 기본설정이 pod를 배포가 안됩니다. 
taint명령어를 통해 pod 배포가 가능하도록 해지 시키도록 하겠습니다.
```
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```


> ## 가상 네트워크(weave) 적용

Calico, Canal, Clilum, Flannel, weave 등 다양한 가상네트워크가 있지만 공식문서에서 제공하는 weave를 적용하도록 하겠습니다.
```
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```


### 가상네트워크 적용 확인
최초 클러스터를 진행하게 되면 노드의 상태는 NotReady 입니다. 가상네트워크를 적용하게 되면 약 1 ~ 3분 소요되고 Ready 상태로 변경되게 됩니다. 이것으로 가상네트워크 적용이 완료 되었습니다.
```
$ kubectl get node
```



> ### nginx 배포 해보기

nginx pod 3개를 배포하는 명령어입니다.
```
kubectl run --image=nginx --port 80 --replicas=3 nginx-app
```

아래 명령어를 통해 nginx-app 배포된 노드를 보면 node1 ~ 3 골고루 배포되어 있는걸 확인할 수 있습니다.
```
kubectl get pod -o wide
```

> ## 정리

이번에 쿠버네티스 고가용성 클러스터를 서버에 구축해보았습니다. 이전 쿠버네티스 버전 1.6 이후 부터는 인증서배포및 클러스터 연결하는 방법이 간편하게 변경되었습니다. 쿠버네티스를 설치후에는 호스트에 tomcat, apache 등 서버를 구축하지 않고 컨테이너 기반으로 서버를 사용할 수 있습니다.


> ## 참고문서
- [kubernetes.io](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)

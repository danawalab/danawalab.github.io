---
layout: post
title:  "쿠버네티스 고가용성 클러스터 구성"
description: "쿠버네티스 고가용성 클러스터 구성해보도록 하겠습니다."
date:   2020.01.28.
writer: "김준우"
categories: Kubernetes
---



## 소개

쿠버네티스는 마스터 노드에 내부적으로 API-SERVER를 가지고 있어 통신이 마스터 노드를 통해 워커 노드로 전달됩니다. 대량에 통신량이 발생하게 되면 마스터 노드는 부하를 많이 받아 장애 발생 가능성이 커지게 됩니다. 마스터 노드를 고가용성 클러스터로 연결하게 되면 통신량을 분산시킬 수 있고, 일부가 마스터 노드에서 장애가 발생해도 워커 노드에서 운영 중인 시스템에는 영향을 줄일 수 있습니다.


## 테스트환경


노드3개
OS: CentOS 7
CPU: 2
RAM: 4
Storage: 100G
각 노드 자원은 동일

## 구성목표

1. 마스터 노드 3개를 클러스터구성
2. 마스터 노드 3개의 pod 배포까지 진행을 위해 taint 해제
3. CNI(weave) 구성 (노드끼리 pod 공유가 됨)
4. 쿠버네티스 웹UI 구성
5. ceph, ceph-dashboard 구성
6. ceph block storage 구성및 사용

### 선행 작업 (전체 노드 동일 작업)

    # 업데이트
    sudo yum update -y
    # 방화벽 끄기 * 쿠버에서 필요함.
    sudo systemctl disable firewalld && sudo systemctl stop firewalld
    # selinux disabled * 쿠버에서 필요함.
    sudo setenforce 0
    # iptables piv4 forward * 쿠버에서 필요함.
    sudo bash -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
    # 스왑종료 * 쿠버에서 필요함.
    sudo swapoff -a
    sudo sed -i '/swap/d' /etc/fstab
    
    # 노드1 
    sudo hostnamectl set-hostname node1
    # 노드2 
    sudo hostnamectl set-hostname node2
    # 노드3
    sudo hostnamectl set-hostname node3
    
### [옵션] 방화벽 키고 진행할려면?
```
쿠버에서 사용하는 포트 목록

마스터 노드
INBOUND 6443 TCP
INBOUND 2379-2380 TCP
INBOUND 10250 TCP
INBOUND 10251 TCP
INBOUND 10252 TCP

  가상네트워크 
    weave
       INBOUND 6783 TCP
       INBOUND 6783 UDP
       INBOUND 6784 UDP

-----------------------------------------------------
워커 노드
INBOUND 10250 TCP
INBOUND 30000-32767 TCP

-----------------------------------------------------
HA 클러스터에 사용하는 로드밸런싱 포트
블로그에서는 26443

```
- 자세한 내용은 아래 링크 참고
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports


### 도커 설치 (전체 노드 동일 작업)

    sudo yum install -y yum-utils \
      device-mapper-persistent-data \
      lvm2
    
    sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    
    sudo yum install docker-ce docker-ce-cli containerd.io
    
    sudo systemctl start docker
    
    sudo cat <<EOF > /etc/docker/daemon.json
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
    
    systemctl daemon-reload
    sudo systemctl enable docker
    sudo systemctl restart docker
    

### 쿠버네티스 설치 (전체 노드 동일 작업)

    sudo cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=0
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg [https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg](https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg)
    EOF
    
    sudo dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
    
    sudo systemctl enable kubelet
    sudo systemctl restart kubelet
    
    # kube init/join 시 자동 이미지 받긴합니다.
    kubeadm config images pull

### 로드밸런서 설치 (노드1에서 진행)

    sudo dnf install haproxy -y
    
    # haproxy 설정하기.
    # * keepalive 및 failover 없이 기본 라운드로빈방식 전송
    
    sudo cat <<EOF >> /etc/haproxy/haproxy.cfg
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
    
    sudo systemctl restart haproxy
    
    netstat -an|grep 26443

### 클러스터 초기화 (노드1에서 진행)

    kubeadm init --control-plane-endpoint "<노드1 DNS/IP or LoadBalancer DNS/IP>:26443" \
                 --upload-certs \
                 --pod-network-cidr "10.244.0.0/16"
    
    결과내용을 잘봐야합니다.
    
    # 샘플)
    #Your Kubernetes control-plane has initialized successfully!
    #
    #To start using your cluster, you need to run the following as a regular user:
    #
    #  mkdir -p $HOME/.kube
    #  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    #  sudo chown $(id -u):$(id -g) $HOME/.kube/config
    #
    #You should now deploy a pod network to the cluster.
    #Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    #  https://kubernetes.io/docs/concepts/cluster-administration/addons/
    #
    #You can now join any number of the control-plane node running the following command on each as root:
    #
    #  kubeadm join 192.168.248.251:6443 --token 06glbc.s0yaqonyajs95ez3 \
    #    --discovery-token-ca-cert-hash sha256:379ff0daa2cffa3f6581ae25c96aa3d6e4a9af43df92b8d0ac5a4dbb4c7e5547 \
    #    --control-plane --certificate-key 8eb7fa9a38ca1579c3fb61e0a1106935c4e9dfd81cbad259121f223f0adf555f
    #
    #Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
    #As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
    #"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.
    #
    #Then you can join any number of worker nodes by running the following on each as root:
    #
    #kubeadm join 192.168.248.251:6443 --token 06glbc.s0yaqonyajs95ez3 \
    #    --discovery-token-ca-cert-hash sha256:379ff0daa2cffa3f6581ae25c96aa3d6e4a9af43df92b8d0ac5a4dbb4c7e5547
    
    
    
    #kubectl 사용할수있는 admin 설정값이 기록되어 있습니다.
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    
    # 마스터 노드 조인 명령어
    # * control-plane --certificate-key 가추가되어 있습니다.
    sudo kubeadm join 192.168.248.251:6443 --token 06glbc.s0yaqonyajs95ez3 \
                 --discovery-token-ca-cert-hash sha256:379ff0daa2cffa3f6581ae25c96aa3d6e4a9af43df92b8d0ac5a4dbb4c7e5547 \
                 --control-plane --certificate-key 8eb7fa9a38ca1579c3fb61e0a1106935c4e9dfd81cbad259121f223f0adf555f
    
    
    # 워커 노드 조인 명령어 입니다.
    sudo kubeadm join 192.168.248.251:6443 --token 06glbc.s0yaqonyajs95ez3 \
                 --discovery-token-ca-cert-hash sha256:379ff0daa2cffa3f6581ae25c96aa3d6e4a9af43df92b8d0ac5a4dbb4c7e5547

### 클러스터 조인 (노드2, 노드3에서 진행)

    # * 저희는 마스터 노드 3개로만 구성하기 때문에 워커 노드 조인은 사용하지 않겠습니다.
    
    #노드2
    sudo kubeadm join 192.168.248.251:6443 --token 06glbc.s0yaqonyajs95ez3 \
                 --discovery-token-ca-cert-hash sha256:379ff0daa2cffa3f6581ae25c96aa3d6e4a9af43df92b8d0ac5a4dbb4c7e5547 \
                 --control-plane --certificate-key 8eb7fa9a38ca1579c3fb61e0a1106935c4e9dfd81cbad259121f223f0adf555f
    
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    #-----------------------------------------------------------------------------------------
    #노드3
    sudo kubeadm join 192.168.248.251:6443 --token 06glbc.s0yaqonyajs95ez3 \
                 --discovery-token-ca-cert-hash sha256:379ff0daa2cffa3f6581ae25c96aa3d6e4a9af43df92b8d0ac5a4dbb4c7e5547 \
                 --control-plane --certificate-key 8eb7fa9a38ca1579c3fb61e0a1106935c4e9dfd81cbad259121f223f0adf555f
    
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

### 노드 구성 확인연결 확인및 가상 네트워크 구성

    kubectl get node
    #노드3개가 보이면 노드 연결은 정상입니다.

### 마스터 노드 taint 해제

    #마스터 노드는 기본적으로 pod 생성을 못하지만 명령어를 통해 해제 하도록하겠습니다.
    kubectl taint nodes --all node-role.kubernetes.io/master-

### 가상 네트워크 구성

    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    #kubectl get node시 status가 Ready가 되면 정상입니다. (약 1 ~ 3분 소요)

가상 네트워크는 대표적으로 Calico, Canal, Clilum, Flannel, WeaveNet 등이 있지만 HA Cluster 시 weave가 노드끼리 POD 공유를 지원을 해서 선택하였습니다.
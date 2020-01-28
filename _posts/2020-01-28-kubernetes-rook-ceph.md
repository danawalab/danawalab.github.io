---
layout: post
title:  "Kubernetes ROOK 사용해보기"
description: "Rook 은 오픈소스 클라우드 네이티브 스토리지 오케스트레이터로 , 클라우드 네이티브 환경과 기본적으로 통합 할 수있는 다양한 스토리지 솔루션 세트에 대한 플랫폼, 프레임 워크 및 지원합니다."
date:   2020-01-28 14:00:00 +0900
writer: "김준우"
categories: Kubernetes
---

# Kubernetes ROOK 사용해보기

# Rook?

---

Rook 은 오픈소스 클라우드 네이티브 스토리지 오케스트레이터로 , 클라우드 네이티브 환경과 기본적으로 통합 할 수있는 다양한 스토리지 솔루션 세트에 대한 플랫폼, 프레임 워크 및 지원합니다.

High-level Storage Provider는 Ceph와 EdgeFS를 제공하며 그외 Cassandra, cockroachDB, Minio, NFS, YugabyteDB 등을 제공합니다.

![/images/2020-01-28-kubernetes-rook-ceph/Untitled.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled.png)

## Ceph vs EdgeFS

---

- Ceph is a highly scalable distributed storage solution for block storage, object storage, and shared filesystems with years of production deployments.
- EdgeFS is high-performance and fault-tolerant decentralized data fabric with access to object, file, NoSQL and block.

Ceph 와 EdgeFS를 비교했을때 EdgeFS는 대용량 스토리지가 필요할때 적합한거같고, Ceph는 공유/확장에 적합한거같다.  우리는 3개의 서버를 HA Cluster로 묶을 예정이고 각 POD는 공유되는 파일 시스템이 필요합니다.

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%201.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%201.png)

### Ceph는 Block, Object, Shared의 유형의 스토리지를 제공

---

- Block Storage

    단일 POD에 스토리지 마운트 

- Object Storage

    애플리케이션이 쿠버네티스 클러스터 내부 또는 외부에서 액세스 할수있느 데이터를 넣고 가져올 수있도록 S3 API를 스토리지 클러스터에 노출

- Shared Stroage

    여러 POD에서 공유할 수있는 파일 시스템 마운트

우리가 필요한 Ceph의 Shared Stroage를 선택한 이유입니다.

### Ceph - Shared Stroage 설치

---

**테스트 환경**

    OS: ubuntu:18.04
    Rook: v1.2
      * Rook:v1.2는 쿠버네티스 1.10 이상을 지원합니다.
    Kubernetes: 1.16.4

**ceph 설치 명령어**

    git clone --single-branch --branch release-1.2 https://github.com/rook/rook.git
    
    cd rook/cluster/examples/kubernetes/ceph
    
    kubectl apply -f common.yaml
    kubectl apply -f operator.ymal
    kubectl apply -f cluster-test.yaml

설치가 많이 심플하다.. git에 example로 클러스터 구성하는 yaml이 어느 정도 제공을하고 합니다.

common.yaml, operator.yaml 은 공통적으로 실행하는 매니패스트다. cluster-test.yaml 은 클러스터를 구성하는 거에 따라 다른 yaml을 실행

나는 테스트노드1에 먼저 설치를 하기 때문에 cluster-test.yaml을 실행합니다.

- cluster.yaml

    프로덕션 스토리지 클러스터에 대한 공통 설정이 포함되어 있다. 3개 이상의 노드가 필요.

- cluster-test.yaml

    중복 구성되지 않은 테스트 클러스터에 대한 설정.

- clister-minimal.yaml

    ceph-mon, ceph-mgr이 하나만 있는 최소의 클러스터 설정합니다.

osd는 노드수와 구성된 장치및 디렉토리 수에 따라 다릅니다.

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%202.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%202.png)

operator.yaml 실행하면 rook-ceph-operator-xxxxxx-xxxx 파드가 생성되지만 상태가 계속 Pending 에서 멈춘다면 어떻게 해야될까..?

- taint가 설정되어있는지 확인해보아야 합니다.

**Shared File System 시작이다..**

지금 까지 Rook, Ceph 를 설치 하였다. 이제 본격적으로 설치해야되는게 Shared File System이다. 그리고 POD에 퍼시스턴트 볼륨을 마운트하여 데이터가 영구적으로 남는지 확인을 해야합니다.

    kubectl apply -f filesystem-test.yaml

결과)

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%203.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%203.png)

이것도 엄청 심플하다... 실행하면 아래 처럼 CephFileSystem이 생성됩니다.

filesystem-test.yaml 옵션은 링크에 접속해서 확인하기 바랍니다.

[https://rook.io/docs/rook/v1.2/ceph-filesystem-crd.html](https://rook.io/docs/rook/v1.2/ceph-filesystem-crd.html)

이것으로 Shared File System 설치는 끝났다.. 하지만 쿠버네티스에 연결할려면 드라이브 처럼 storageclass 가 필요한거같습니다.

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: csi-cephfs
    provisioner: rook-ceph.cephfs.csi.ceph.com
    parameters:
      # clusterID is the namespace where operator is deployed.
      clusterID: rook-ceph
    
      # CephFS filesystem name into which the volume shall be created
      fsName: myfs
    
      # Ceph pool into which the volume shall be created
      # Required for provisionVolume: "true"
      pool: myfs-data0
    
      # Root path of an existing CephFS volume
      # Required for provisionVolume: "false"
      # rootPath: /absolute/path
    
      # The secrets contain Ceph admin credentials. These are generated automatically by the operator
      # in the same namespace as the cluster.
      csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
      csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
      csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
      csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
    
      # (optional) The driver can use either ceph-fuse (fuse) or ceph kernel client (kernel)
      # If omitted, default volume mounter will be used - this is determined by probing for ceph-fuse
      # or by setting the default mounter explicitly via --volumemounter command-line argument.
      # mounter: kernel
    reclaimPolicy: Delete
    mountOptions:
      # uncomment the following line for debugging
      #- debug
    

csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph

이부분이 중요해 보인다. 일단 실행해봅니다.

    /rook/cluster/examples/kubernetes/ceph/csi/cephfs/
    
    kubectl apply -f storageclass.yaml

**최종 모습**

POD)

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%204.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%204.png)

CephFileSystem)

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%205.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%205.png)

Service)

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%206.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%206.png)

Secrets)

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%207.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%207.png)

### POD에 Share File System Mount 해보기.

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: cephfs-pvc
    spec:
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 1Gi
      storageClassName: csi-cephfs
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: kube-registry
      namespace: kube-system
      labels:
        k8s-app: kube-registry
        kubernetes.io/cluster-service: "true"
    spec:
      replicas: 3
      selector:
        matchLabels:
          k8s-app: kube-registry
      template:
        metadata:
          labels:
            k8s-app: kube-registry
            kubernetes.io/cluster-service: "true"
        spec:
          containers:
          - name: registry
            image: registry:2
            imagePullPolicy: Always
            resources:
              limits:
                cpu: 100m
                memory: 100Mi
            env:
            # Configuration reference: https://docs.docker.com/registry/configuration/
            - name: REGISTRY_HTTP_ADDR
              value: :5000
            - name: REGISTRY_HTTP_SECRET
              value: "Ple4seCh4ngeThisN0tAVerySecretV4lue"
            - name: REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY
              value: /var/lib/registry
            volumeMounts:
            - name: image-store
              mountPath: /var/lib/registry
            ports:
            - containerPort: 5000
              name: registry
              protocol: TCP
            livenessProbe:
              httpGet:
                path: /
                port: registry
            readinessProbe:
              httpGet:
                path: /
                port: registry
          volumes:
          - name: image-store
            persistentVolumeClaim:
              claimName: cephfs-pvc
              readOnly: false

PersistentVolumeClaim을 1Gi Request로 생성하고 name은 cephfs-pvc로 만듭니다.

docker registry pod를 생성하는데 cephfs-pvc볼륨을 연결하고 /var/lib/registry경로에 마운트합니다.

그리고 replaceas를 3개로 뛰어 이미지 저장이 공유되는지 확인합니다.

또는 마운트된 경로에 파일을 추가하여 POD간에 공유가 되는지 확인합니다.

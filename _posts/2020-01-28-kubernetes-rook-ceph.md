---
layout: post
title:  "쿠버네티스 가상스토리지(Ceph) 설치"
description: "Rook-Ceph는 오픈소스 클라우드 네이티브 스토리지 오케스트레이터로, 클라우드 네이티브 환경과 기본적으로 통합 할 수있는 다양한 스토리지 솔루션 세트에 대한 플랫폼, 프레임 워크 및 지원합니다."
date:   2020.01.28.
writer: "김준우"
categories: Kubernetes
---




## 소개

이번에 설치해볼 가상스토리지는 Ceph입니다. 설치에 필요한 도구로 Rook을 사용할 예정입니다. Rook은 오픈소스 클라우드 네이티브 스토리지 오케스트레이터로, 클라우드 네이티브 환경과 기본적으로 통합할 수 있는 다양한 스토리지 솔루션 세트에 대한 플랫폼, 프레임 워크 및 지원합니다. Rook을 통해 Ceph 가상스토리지를 구성하고 공유 파일 시스템을 적용하도록 하겠습니다.



## Rook 구성도

Rook은 쿠버네티스 POD에서 실행되며, Ceph, EdgeFS등 가상솔루션을 POD로 배포하여 관리하는 도구입니다. agent를 통해 Ceph, EdgeFS등 가상솔루션을 관리하고, OSD를 통해 데이터를 영구저장합니다.

![/images/2020-01-28-kubernetes-rook-ceph/Untitled.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled.png)

## Ceph vs EdgeFS

Rook에서 제공하는 가상스토리지는 EgdeFS와 Ceph 외에도 다양하게 지원하지만 안정적인 버전은 아래 두가지만 지원합니다.

- EdgeFS: 데이터베이스처럼 대용량 스토리지가 필요할때 사용됩니다.

- Ceph: 공유 확장에 특화되어 있는 스토리지가 필요할때 사용됩니다.

공유나 확장에 특화되어 있는 Ceph를 설치하도록 하겠습니다.

### Ceph 스토리지 유형

Ceph는 Block, Object, File 기반으로 데이터를 사용할 수 있습니다. 각 유형에 따라서 사용하는 기능에 차이가 있습니다.

- Block Stroage: 단일 POD에 storage 제공합니다.
- Object Storage: 애플리케이션이 쿠버네티스 클러스터 내부 또는 외부에서 액세스 할수있느 데이터를 IO 할수있고, S3 API를 스토리지 클러스터에 노출을 제공합니다.
- Shared Stroage: 여러 POD에서 공유할 수있는 파일 시스템기반 스토리지입니다.


### Shared Storage 구성도
이번에 설치해볼 Shared Storage 구성입니다. Ceph 클러스터에 데이터를 저장하고 Ceph를 통해 POD들과 데이터가 공유가 됩니다.

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%201.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%201.png)


## Ceph 설치

### 테스트 환경
쿠버네티스가 설치된 환경이 필요합니다. 설치방법은 [쿠버네티스 고가용성 클러스터 구성](https://danawalab.github.io/kubernetes/2020/01/28/kubernetes-ha-cluster.html) 블로그를 통해 설치진행 바랍니다.

```
OS: ubuntu:18.04
Kubernetes: 1.16.4
```


### Ceph 소스 다운로드 및 POD 배포

Rook은 별도의 설치가 없고 github으로 Ceph, EdgeFS 가상스토리지 솔루션을  쿠버네티스에 배포할 수 있도록 제공하고 있습니다.

```
$ git clone --single-branch --branch release-1.2 https://github.com/rook/rook.git

```

아래 경로에 보면 common.yaml, operator.yaml이 있습니다. kubectl 명령어를 통해 배포 합니다.
```
$ kubectl apply -f rook/cluster/examples/kubernetes/ceph/common.yaml
$ kubectl apply -f rook/cluster/examples/kubernetes/ceph/operator.ymal
$ kubectl apply -f rook/cluster/examples/kubernetes/ceph/cluster.yaml
```

cluster.yaml을 배포하게 되면 POD 중 OSD가 Running 상태인지 확인이 필요합니다. 변경되는 시간이 약 1 ~ 3 분 정도 소요됩니다.    
테스트 환경에 따라 OSD 갯수가 차이가 있을 수 있습니다.
```
$ kubectl get pod -n rook-ceph
```

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%202.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%202.png)


OSD Running 상태까지가 ceph 클러스터 구성이 완료되었고, Shared Storage 유형을 적용해보겠습니다.


### Shared Storage 유형 적용

아래 명령어를 실행하게 되면 MDS POD가 배포가 됩니다. 약 1 ~ 3분 정도 소요됩니다. Shared Storage 옵션을 변경할 수 있는데 자세한 내용은 [ceph-filesystem-crd](https://rook.io/docs/rook/v1.2/ceph-filesystem-crd.html) 통해 확인바랍니다.

```
$ kubectl apply -f rook/cluster/examples/kubernetes/ceph/filesystem.yaml
```

### StorageClass Driver 적용

StorageClass는 POD에서 Ceph 데이터 풀에 접근하기 위해 사용되는 드라이버입니다. kubectl 명령어를 통해 적용합니다.

```
$ kubectl apply -f rook/cluster/examples/kubernetes/ceph/csi/cephfs/storageclass.yaml
```

### Shared Storage 확인
정상적으로 적용이 되었는지 확인하는 방법압니다. 아래 결과 처럼 myfs가 생성되어 있습니다.
```
$ kubectl get CephFileSystem -A
```

결과)

![/images/2020-01-28-kubernetes-rook-ceph/Untitled%203.png](/images/2020-01-28-kubernetes-rook-ceph/Untitled%203.png)


### Shared Storage 테스트

kube-registry를 배포하면 PVC, docker-registry POD가 배포됩니다. 컨테이너 내부에 접근하여 /var/lib/registry 디렉터리에 dummy 파일을 생성 하여 다른 POD에서 공유되어 보여지는지 확인합니다.

```
$ kubectl apply -f rook/cluster/examples/kubernetes/ceph/csi/cephfs/kube-registry.yaml

```


## 정리

Rook이라는 도구를 이용해서 Ceph를 설치해보았습니다. Ceph를 관리하려면 러닝커브가 있지만 Rook이라는 도구를 이용해서 Ceph 관리와 쿠버네티스와 연동을 쉽게 할 수 있습니다. 



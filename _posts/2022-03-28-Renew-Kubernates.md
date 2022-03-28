---
layout: post
title:  "Kubernetes 인증서 만료시 갱신 방법"
description: "쿠버네티스 인증서 갱신이 되지 않았을 경우 발생하는 오류와 해결방법을 알아봅니다"
date:   2022.03.28.
writer: "반윤성"
categories: Kubernetes
---

## Kubernetes cluster의 인증 기간

Kubernetes cluster 인증서의 수명은 1년입니다.

Kubernetes 인증서가 만료되면, 결과적으로 kubelet 서비스가 실패합니다.

실제로 현재 운영중인 gitlab-runner가 정상적으로 작동하지 않았던 현상이 있습니다.

`ERROR: Build failed (system failure): timedout waiting for pod to start`

또한 kubectl 명령어 수행 시 `Unable to connect to the server: x509: certificate has expired or is not yet valid.` 과 같은 메시지가 출력되기도합니다.

## Kubernetes expired ?

어느 순간 Kubernetes의 pod가 정상적으로 동작하지 않거나 서버로부터 응답이 없다면
인증서가 만료 되었는지 확인해보아야 합니다.

k8s 마스터 인증서 기간 확인을 위해 다음과 같이 입력합니다.

```
[12:53:32][danawa@kube3 /etc/kubernetes/pki]
$ openssl x509 -in apiserver.crt -noout -text |grep ' Not '
            Not Before: Mar 25 08:12:56 2020 GMT
            Not After : Mar 25 08:29:03 2022 GMT
```

이 글을 쓰는 현재 시간은 2022년 3월 28일 입니다. 위 내용을 통해 인증서 파일이 
만료되었다는 것을 확인할 수 있고, 이때부터 관련된 기능들이 정상적으로 동작하지 않습니다.


## 인증서 갱신 방법

#### 1. 인증서가 존재하는 pki 디렉토리를 백업합니다.

```
cp -f /etc/kubernetes/pki /etc/kubernetes/pki-backup
```

#### 2. 인증서를 일괄적으로 업데이트 합니다.

```
kubeadm alpha certs renew all
```

#### 3. 갱신 여부를 다시 체크합니다.
```
[12:56:54][danawa@kube3 /etc/kubernetes/pki]
$ openssl x509 -in apiserver.crt -noout -text |grep ' Not '
            Not Before: Mar 25 08:12:56 2020 GMT
            Not After : Mar 28 02:26:40 2023 GMT
```

정상적으로 1년 연장 되었습니다.

#### 4. 사용자 구성을 업데이트 하기 위해 kubernetes 디렉토리를 백업합니다.
```
cp -f /etc/kubernetes /etc/kubernetes-backup
```

#### 5. 사용자 구성을 업데이트합니다.
```
kubeadm alpha kubeconfig user --client-name=admin
kubeadm alpha kubeconfig user --org system:masters --client-name kubernetes-admin > /etc/kubernetes/admin.conf
kubeadm alpha kubeconfig user --client-name system:kube-controller-manager > /etc/kubernetes/controller-manager.conf
kubeadm alpha kubeconfig user --org system:nodes --client-name system:node:$(hostname) > /etc/kubernetes/kubelet.conf
kubeadm alpha kubeconfig user --client-name system:kube-scheduler > /etc/kubernetes/scheduler.conf
```

#### 6. /root/.kube/config를 교체합니다.
```
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

#### 7. 서비스를 재시작합니다.
```
systemctl daemon-reload&&systemctl restart kubelet

# systemctl 서비스에서 사용할 경우
systemctl restart kube-apiserver
systemctl restart kube-scheduler

# 도커 컨테이너를 사용할 경우
docker restart [apiserver 컨테이너]
docker restart [scheduler 컨테이너]
docker restart [controller 컨테이너]
```

## 정리

Kubernetes의 인증서는 갱신 방법을 살펴보았습니다. 갱신이 되지않는다면 정상적으로 서비스를 운용할수가 없기때문에,
관련 내용을 염두해두고 있어야 신속하게 troubleshooting 할 수 있을 것으로 보입니다.

## 참고 자료
- https://www.ibm.com/docs/en/fci/1.0.3?topic=kubernetes-renewing-114x-cluster-certificates
---
layout: post
title:  "Kubernetes 서비스와 인그레스 용도구분"
description: "이번 포스팅에서는 쿠버네티스의 서비스와 인그레스의 차이점에 대해서 알아보겠습니다. 쿠버네티스를 처음 접하는 분들은, 간단하게 API 서버를 만들어서 `POD` 를 띄우기까지는 성공하지만, 해당 IP와 포트로 접속이 안되어 당황을 하게 됩니다."
date:   2020.01.23.
writer: "송상욱"
categories: Kubernetes
---

이번 포스팅에서는 쿠버네티스의 서비스와 인그레스의 차이점에 대해서 알아보겠습니다.

쿠버네티스를 처음 접하는 분들은, 간단하게 API 서버를 만들어서 `POD` 를 띄우기까지는 성공하지만, 해당 IP와 포트로 접속이 안되어 당황을 하게 됩니다. 그후 `Service` 라는 개념을 알게되고, 서비스 타입을 `NodePort` 또는 `LoadBalancer`로 설정해서 사용합니다. 그리고 `Ingress` 개념에 대해서도 배우게 되는데, 여기까지 오면 서비스와 인그레스중 어떤걸 사용해야 하는지 헷갈리기 시작합니다. 그래서 여기서는 앱을 외부에 노출하고자 할때, 서비스와 인그레스의 사용법을 구분하고자 합니다.

## 구동환경에 따른 구분

`Service` 를 처음 만들면 기본적으로 `ClusterIP` 타입으로 생성되며 외부로 노출이 불가능합니다.
`Service`의 `NodePort` 타입은 온프레미스 환경에서 호스트 IP가 노출이 가능한 경우에 사용합니다.
`Service`의 `LoadBalancer` 타입은 AWS, GCP, Azure등 `Cloud Load Balancer`가 제공되는 퍼블릭 클라우드 환경에서만 사용할수 있습니다.


## Service와 Ingress 의 용도구분

`Service`는 클러스터 외부로 `PORT`를 노출하는 기능과 부하분산기능을 수행합니다. 그리고 이점은 `Ingress` 도 동일합니다. 차이점은 `Ingress`는 `L7`이고 `Service` 는 `L4`라는 점입니다.

<p align="center">
<img src="/images/2020-01-23-kubernetes-service-ingress/2019-08-22-translation-kubernetes-nodeport-vs-loadbalancer-vs-ingress4.png">
</p>
<p align="center">
인그레스-서비스 네트워크 구조
<br/>
출처: https://<i></i>medium.com/google-cloud
</p>


서비스는 여러개로 복제된 한 종류의 `POD`를 로드밸런싱합니다. 아래는 `50000`포트를 오픈한 `POD`의 `hello-world`라는 서비스의 YMAL 설정입니다.

```
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  type: NodePort
  selector:
    greeting: hello
    department: world
  ports:
  - protocol: TCP
    port: 60000
    targetPort: 50000
```

반면에 인그레스는 `여러 서비스`에 대해서 `라우팅`의 역할을 담당합니다. 예를 들면, 아래의 인그레스 설정은 `/` 로 요청시 `hello-world` 라는 서비스로 연결하고, `/kube` 로 요청시 `hello-kubernetes` 라는 서비스로 연결합니다.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          serviceName: hello-world
          servicePort: 60000
      - path: /kube
        backend:
          serviceName: hello-kubernetes
          servicePort: 80
```

요약하면, 일반적인 경우 온프레미스 환경에서는 `Service`의 `NodePort`타입을 사용하고, 퍼블릭 클라우드 환경에서는 `Service`의 `LoadBalancer`타입을 사용하면 대부분 서비스 운영이 가능합니다.

그러나, 만약 MSA (Micro Service Architecture) 로 개발되어 서비스간의 라우팅이 필요한 구조에서는 `Ingress` 를 사용합니다.

이상으로 쿠버네티스의 `서비스`와 `인그레스`에 대해서 알아봤습니다.

## 참고문헌

- https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress?hl=ko
- https://bcho.tistory.com/1262?category=731548
- https://blog.leocat.kr/notes/2019/08/22/translation-kubernetes-nodeport-vs-loadbalancer-vs-ingress

## 채용

다나와에서는 쿠버네티스를 기반으로한 하이브리드 클라우드 플랫폼을 구축하고 있습니다. 아래와 같은 기술에 관심이 많은 액티브한 개발자를 모집중에 있습니다.

- 쿠버네티스
- 엘라스틱서치
- 프로메테우스, 그라파나
- 마이크로 서비스 아키텍처

다나와 채용 https://recruit.danawa.com/ 에서 지금 당장 지원해주세요!

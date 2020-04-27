---
layout: post
title:  "Knative 사용을 위한 비교 테스트"
description: "Knative와 Ingress+Deployment, Kubeless 비교"
date:   2020.04.24.
writer: "하선호"
categories: Kubernetes
---

## 개요

사내에서 기존 물리서버에서 동작하는 암호화 방식 서비스를 Knative로 서비스 하기 앞서 어느정도 성능이 나오는지 테스트가 필요하여 진행하게 되었습니다.
Knative만 아니라 Ingress+Deployment, Kubeless로도 구성하여 테스트 해보고 어떤 차이점이 있는지 알아보고자 합니다.
따라서 이 블로그에서는 SpringBoot 서비스를 직접 설치 방식, Knative, Ingress+Deployment, Kubeless로 각각 구성했을 때 성능에 대한 테스트를 진행하겠습니다.


## 구성
 
```
1. SPRINGBOOT APPLICATION with Micrometer
2. Jmetter
3. Prometheus + Grafana
```

## 테스트

먼저 키워드를 암호화하는 로직을 SPRINGBOOT REST API로 작성 후에 Jmetter를 통해 성능 테스트를 진행하였습니다. 측정은 해당 SPRING BOOT에 Micrometer를 적용하여 Prometheus와 Grafana를 통해 측정하였고 모든 테스트는 5분간 호출하는 방식으로 진행하였습니다.

### 1. 직접 설치 방식
직접 서버에 SPRINGBOOT RESTAPI를 가동하여 테스트를 진행하였습니다.

|THREAD|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|30|37|9578|3|
|50|38|9589|5|
|70|38|9605|7|

|THREAD 30|
|:---:|
|![/images/2020-04-24-Knative-Compare-Test/springboot-th30.png](/images/2020-04-24-Knative-Compare-Test/springboot-th30.png)|


### 2. Knative
- Knative 설치 후 SPRINGBOOT를 배포하여 테스트를 진행하였습니다.

|THREAD|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|30|70|4061|7|
|50|68|4295|11|
|70|69|4396|15|

|THREAD 30|
|:---:|
|![/images/2020-04-24-Knative-Compare-Test/knative-th30.png](/images/2020-04-24-Knative-Compare-Test/knative-th30.png)|

### 3. Ingress+Deployment
- Deployment를 직접 생성하여 배포 후 테스트를 진행하였습니다.

|THREAD|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|30|74|9638|2|
|50|73|10540|4|
|70|78|10619|6|

|THREAD 30|
|:---:|
|![/images/2020-04-24-Knative-Compare-Test/ingress-th30.png](/images/2020-04-24-Knative-Compare-Test/ingress-th30.png)|

### 4. Kubeless
- 로직 코드를 Kubeless를 통해 배포 후 테스트를 진행하였습니다.

|THREAD|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|30|12|717|40|
|50|12|1194|41|
|70|12|1671|41|

|THREAD 30|
|:---:|
|![/images/2020-04-24-Knative-Compare-Test/kubeless-th30.png](/images/2020-04-24-Knative-Compare-Test/kubeless-th30.png)|

## 결과
각 테스트 결과를 종합하면 다음과 같습니다.
  
THREAD 30

|타입|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|직접설치|37|9578|3|
|Knative|70|4061|7|
|Ingress|74|9638|2|
|Kubeless|12|717|40|

THREAD 50

|타입|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|직접설치|38|9589|5|
|Knative|68|4295|11|
|Ingress|73|10540|4|
|Kubeless|12|1194|41|

THREAD 70

|타입|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|직접설치|38|9605|7|
|Knative|69|4396|15|
|Ingress|78|10619|6|
|Kubeless|12|1671|41|

  

위에 표를 보면 응답속도는 직접설치 방식, Knative, Ingress, Kubeless가 각각 3ms, 7ms, 2ms, 40ms로 Ingress, 직접설치, Knative, Kubeless 순으로 측정이 되었습니다.

평균적인 CPU 사용량은 모든 THREAD에서 Ingress 방식이 평균 75%로 가장 높았고 그 다음으로 knative가 약 69%로 높았습니다. 둘 모두 컨테이너에서 사용되는 CPU 사용량이 있어 높은 것 같습니다.

또한 Knative 보다 Ingress에서 응답속도가 빠른 이유는 Knative의 경우 Istio-proxy에서 소요되는 시간이 줄어든 것으로 보입니다.

 ![/images/2020-04-24-Knative-Compare-Test/knative-performance-issues.png](/images/2020-04-24-Knative-Compare-Test/knative-performance-issues.png)\

Kubeless의 경우 CPU 사용량이 테스트중 가장 낮을뿐더러 일정하였지만 응답시간은 가장 높았습니다. 그 이유로는 Kubeless에서 java-runtime시 구현되는 handler.java 내 코드에서 쓰레드 수가 고정으로 설정되어 있어 CPU 사용량이 과도하게 증가하지 않는 것 같습니다.

  ```
  kubeless/runtimes - Handler.java 
  line - 75
  ---------------------------------------------------------------------------
  server.setExecutor(java.util.concurrent.Executors.newFixedThreadPool(50));
  ```

## 결론

구조만 보면 외부와 직접 통신하는 직접설치 방식보다 컨테이너 방식의 서비스들이 더 낮은 TPS를 가지게 될 것 같았지만 의외로 INGRESS+DEPLOYMENT로 테스트 하였을 때 높은 수치가 나왔습니다. 이 부분은 원인 파악이나 추가적으로 더 테스트가 필요할 것 같아 보입니다.
성능을 떠나서 Knative는 개발자가 신경써야 할 관리요소들을 많이 줄일 수 있습니다. Knative를 사용하기에 참고가 되는 테스트를 진행한 것 같습니다.

  
## 참고 자료
- https://knative.dev/docs/serving/debugging-performance-issues/
- https://github.com/kubeless/runtimes
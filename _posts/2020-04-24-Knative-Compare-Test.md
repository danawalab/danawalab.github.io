---
layout: post
title:  "Knative 사용을 위한 비교 테스트"
description: "Knative와 Ingress+Deployment, Kubeless 비교"
date:   2020.04.24.
writer: "하선호"
categories: Kubernetes
---

## 개요
- 사내에서 기존 물리서버에서 동작하는 서비스를 Knative로 서비스 하기 앞서 성능에 대한 테스트가 필요하여 진행하게 되었습니다.
- Knative만 아니라 On-Premise에서 직접 동작하는 서비스를 Ingress+Deployment, Kubeless로도 구성하여 테스트하고자 합니다.
- 따라서 이 블로그에서는 SpringBoot 서비스를 On-Premise, Knative, Ingress+Deployment, Kubeless로 각각 구성했을 때 성능에 대한 테스트를 진행하겠습니다.


## 구성
 
```
1. SPRINGBOOT APPLICATION with Micrometer
2. Jmetter
3. Prometheus + Grafana
```

## 테스트

### 테스트 방식
- 키워드를 암호화하는 로직을 SPRINGBOOT REST API로 생성하였습니다.
- Jmetter를 통해 성능 테스트를 진행하였습니다.
- 성능 측정은 SPRINGBOOT에 Micrometer를 적용하여 Prometheus에서 해당 API 톰캣의 Metric 정보를 가져갈 수 있게 한 후 Grafana를 통해 측정하였습니다.
- Knative, Ingress+Deployment, Kubeless는 SpringBoot를 도커로 빌드하여 배포하였습니다.
- Knative - Istio, IngressController로는 Nginx를 설치하여 사용하였습니다
- 모든 테스트는 5분동안 진행되었습니다.

### On premise
- 직접 서버에 SPRINGBOOT RESTAPI를 가동하여 테스트를 진행하였습니다.

|THREAD|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|30|37|9578|3|
|50|38|9589|5|
|70|38|9605|7|

|30|50|70|
|:---:|:---:|:---:|
|![/images/2020-04-24-Knative-Compare-Test/springboot-th30.png](/images/2020-04-24-Knative-Compare-Test/springboot-th30.png)|![/images/2020-04-24-Knative-Compare-Test/springboot-th50.png](/images/2020-04-24-Knative-Compare-Test/springboot-th50.png)|![/images/2020-04-24-Knative-Compare-Test/springboot-th70.png](/images/2020-04-24-Knative-Compare-Test/springboot-th70.png)


### Knative
- Knative 설치 후 SPRINGBOOT를 배포하여 테스트를 진행하였습니다.

|THREAD|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|30|70|4061|7|
|50|68|4295|11|
|70|69|4396|15|

|30|50|70|
|:---:|:---:|:---:|
|![/images/2020-04-24-Knative-Compare-Test/knative-th30.png](/images/2020-04-24-Knative-Compare-Test/knative-th30.png)|![/images/2020-04-24-Knative-Compare-Test/knative-th50.png](/images/2020-04-24-Knative-Compare-Test/knative-th50.png)|![/images/2020-04-24-Knative-Compare-Test/knative-th70.png](/images/2020-04-24-Knative-Compare-Test/knative-th70.png)

### Ingress+Deployment
- Deployment를 직접 생성하여 배포 후 테스트를 진행하였습니다.

|THREAD|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|30|74|9638|2|
|50|73|10540|4|
|70|78|10619|6|

|30|50|70|
|:---:|:---:|:---:|
|![/images/2020-04-24-Knative-Compare-Test/ingress-th30.png](/images/2020-04-24-Knative-Compare-Test/ingress-th30.png)|![/images/2020-04-24-Knative-Compare-Test/ingress-th50.png](/images/2020-04-24-Knative-Compare-Test/ingress-th50.png)|![/images/2020-04-24-Knative-Compare-Test/ingress-th70.png](/images/2020-04-24-Knative-Compare-Test/ingress-th70.png)

### Kubeless
- 로직 코드를 Kubeless를 통해 배포 후 테스트를 진행하였습니다.

|THREAD|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|30|12|717|40|
|50|12|1194|41|
|70|12|1671|41|

|30|50|70|
|:---:|:---:|:---:|
|![/images/2020-04-24-Knative-Compare-Test/kubeless-th30.png](/images/2020-04-24-Knative-Compare-Test/kubeless-th30.png)|![/images/2020-04-24-Knative-Compare-Test/kubeless-th50.png](/images/2020-04-24-Knative-Compare-Test/kubeless-th50.png)|![/images/2020-04-24-Knative-Compare-Test/kubeless-th70.png](/images/2020-04-24-Knative-Compare-Test/kubeless-th70.png)

## 결과
- 각 테스트 결과를 종합하면 다음과 같습니다.
  
THREAD 30

|타입|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|On-Prem|37|9578|3|
|Knative|70|4061|7|
|Ingress|74|9638|2|
|Kubeless|12|717|40|

THREAD 50

|타입|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|On-Prem|38|9589|5|
|Knative|68|4295|11|
|Ingress|73|10540|4|
|Kubeless|12|1194|41|

THREAD 70

|타입|평균 CPU(%)|TPS|평균 응답시간(ms)|
|:------:|:---:|:---:|:---:|
|On-Prem|38|9605|7|
|Knative|69|4396|15|
|Ingress|78|10619|6|
|Kubeless|12|1671|41|

- 모든 쓰레드에서 응답속도는 On-promise, Knative, Ingress, Kubeless 각각 3ms, 7ms, 2ms, 40ms로 Ingress, On-promise 방식이 빠르게 측정되었습니다.
  
- 평균적인 CPU 사용량은 모든 THREAD에서 Ingress 방식이 평균 75%로 가장 높았고 그 다음으로 knative가 약 69%로 높았습니다. 둘 모두 컨테이너에서 사용되는 CPU 사용량이 있어 높은 것 같습니다.

- Knative 보다 Ingress에서 응답속도가 빠른 이유는 Knative의 경우 Istio-proxy에서 소요되는 시간이 줄어든 것으로 보입니다.

  - knative.dev/docs/serving/debugging-performance-issues \
 ![/images/2020-04-24-Knative-Compare-Test/knative-performance-issues.png](/images/2020-04-24-Knative-Compare-Test/knative-performance-issues.png)\


- Kubeless의 경우 CPU 사용량이 테스트중 가장 낮을뿐더러 일정하였지만 응답시간은 가장 높았습니다. 그 이유로는 Kubeless에서 java-runtime시 구현되는 handler.java 내 코드에서 쓰레드 수가 고정으로 설정되어 있어 CPU 사용량이 과도하게 증가하지 않는 것 같습니다.
  ```
  kubeless/runtimes - Handler.java 
  line - 75
  ---------------------------------------------------------------------------
  server.setExecutor(java.util.concurrent.Executors.newFixedThreadPool(50));
  ```

## 결론
 
각각의 구성의 특징 때문에 기존 On-promise 방식보다는 TPS가 낮았습니다. 다만 Knative, Kubless와 같은 Serverless나 컨테이너 환경을 사용함으로써 얻는 이점(서버 설정 및 관리요소 제거)과 서비스 성능 사이를 잘 판단하여 사용해야할 것 같습니다.
  
## 참고 자료
- https://knative.dev/docs/serving/debugging-performance-issues/
- https://github.com/kubeless/runtimes
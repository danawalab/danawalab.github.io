---
layout: post
title:  "이벤트서비스 GCP 전환"
description: "다나와 이벤트 서비스를 클라우드로 전환하기 위한 부하 테스트를 진행하였습니다. 다나와 이벤트서비스는 이벤트가 없으면 아주 적은 트래픽이 발생 하지만, 이벤트가 시작되면 동시에 2만 요청까지 발생됩니다. 대량 처리가 가능한 서버를 구축을 하게 되면 비용이 많이 소모되며 이벤트가 없으면 자원낭비가 발생합니다. 그래서 이벤트 서비스는 구글 클라우드(GCP)에서 제공하는 Cloud Run을 사용해보기 위해 부하 테스트 및 과금 정책에 대해 공유 합니다."
date:   2020.02.18.
writer: "김준우"
categories: GCP
---
## 소개

다나와 이벤트 서비스를 클라우드로 전환하기 위한 부하 테스트를 진행하였습니다. 다나와 이벤트서비스는 이벤트가 없으면 아주 적은 트래픽이 발생 하지만, 이벤트가 시작되면 동시에 2만 요청까지 발생됩니다. 대량 처리가 가능한 서버를 구축을 하게 되면 비용이 많이 소모되며 이벤트가 없으면 자원낭비가 발생합니다. 그래서 이벤트 서비스는 구글 클라우드(GCP)에서 제공하는 Cloud Run을 사용해보기 위해 부하 테스트 및 과금 정책에 대해 공유 합니다.

## Cloud Run 이란?

완전 관리형 환경 또는 Anthos에서 스테이트리스(stateless) 컨테이너를 실행할 수 있습니다.  구글에서 가지고 있는 커다란 쿠버네티스 클러스터에 서비스를 배포할 수 있고, Anthos라고 직접 구성한 클러스터에 연동하여 서비스를 배포할 수 있습니다. 모든 언어, 라이브러리, 바이너리까지 지원이되고, 자동확장기능을 지원합니다. 내부적으로 오픈소스인 Knative 기반으로 빌드됩니다.

## stress 테스트 시나리오

- Cloud Run 샘플 프로젝트 배포
- 대량 요청 전송
- 모니터링 및 결과 레포팅

### Cloud Run 샘플 프로젝트 구성

샘플 프로젝트는 동시 접속을 위해 500ms 지연시키는 페이지를 구성합니다. 지연시키는 이유는 동시접속시 부하를 가중시키는 효과를 보기 위함입니다.

index.php

```
<? usleep(500000); ?>
```

### nGrinder 구성

부하 테스트로 네이버에서 개발된 nGrinder 를 사용합니다. GCP VM을 사용하여 컨트롤러 1대와 에이전트 90대를 구성하여 동시에 요청을 보낼수 있도록 구성합니다. 

- 명령어는 gcloud가 구성되어 있어야 사용가능합니다.

docker hub에 ngrinder controller 와 agent를 제공하고 있어 설치 과정없이 바로 Controller를 생성합니다.

```
gcloud beta compute --project=event-web-poc instances create-with-container controller --zone=asia-northeast1-b --machine-type=n1-standard-4 --subnet=default --network-tier=PREMIUM --metadata="google-logging-enabled=true,startup-script=docker run -d -p 80:80 -p 16001:16001 -p 12000-12009:12000-12009 --name=agent ngrinder/controller" --maintenance-policy=MIGRATE --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server,ngrinder-port --image=cos-stable-80-12739-68-0 --image-project=cos-cloud --container-image=ngrinder/controller --container-restart-policy=always
```

저는 윈도우 PC를 사용하고 있어 bat파일을 만들어 90번 반복하여 agent를 생성합니다.

create-agent.bat
```
@echo off
setlocal

:LOOP
set /a no+=1
echo "Loop.. %no%" 

call gcloud beta compute --project=event-web-poc instances create-with-container agent-%no% --zone=asia-northeast1-b --machine-type=n1-standard-4 --subnet=default --network-tier=PREMIUM --metadata="google-logging-enabled=true,startup-script=docker run -d -p 16001:16001 -p 12000-12009:12000-12009 --add-host=controller:<Controller IP> --name=agent ngrinder/agent" --maintenance-policy=MIGRATE --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server,ngrinder-port --image=cos-stable-80-12739-68-0 --image-project=cos-cloud --container-image=ngrinder/agent --container-restart-policy=always 

if %no% GEQ 90 goto QUIT

goto LOOP

:QUIT
```


### 대량 요청 전송

아래 이미지는 Agent 90대에서 동시 21,420건 요청을 5분간 전송한 결과입니다. 점점 요청에 적응을 하며 대응하는 모습을 볼 수 있었습니다.

![/images/2020-02-18-GCP-Cloud-Run-stress-testing/Untitled.png](/images/2020-02-18-GCP-Cloud-Run-stress-testing/Untitled.png)

## Cloud Run 과금 정책

GCP는 사용한 만큼 과금되는 정책을 사용합니다. AWS의 Lambda와 비슷한 기능을 제공하지만 AWS Lambda는 자원사용한 만큼 과금이 되기 때문에 Cloud Run에서도 컨테이너 사용량 만큼 과금을 할 줄 알았는데 요청에 따라 과금을 한다는 문서를 찾았습니다. cloud run에서는 요청이 발생하면 100ms 단위로 과금이 되고, 중복된 요청 시간에 대해서는 1회 과금이 됩니다. 일시적으로 많은 요청을 할수록 유리하다는 걸 알게 되었습니다.

![/images/2020-02-18-GCP-Cloud-Run-stress-testing/Untitled%201.png](/images/2020-02-18-GCP-Cloud-Run-stress-testing/Untitled%201.png)

## 정리

GCP의 Cloud Run은 일시적으로 많은 요청이 있는 서비스에 적합하다는 결과를 보았습니다. GCP에서는 동시 2만 이상의 요청에도 장애없이 응답하였고, 기본적으로 제공하는 Cloud Run의 할당량이 컨테이너 1000개, 최대 동시 접속 80개로 계산하게 되면 이론상 8만건 처리가 가능합니다. 추가적으로 컨테이너 할당량을 늘려 사용이 가능하기 때문에 이벤트 서비스에 적합한 결과를 보였습니다.

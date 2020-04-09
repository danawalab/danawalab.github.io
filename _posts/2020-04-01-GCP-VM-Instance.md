---
layout: post
title:  "이벤트 서비스 GCP VM Instance 배포"
description: "얼마전에 이벤트 서비스를 GCP의 Cloud Run에 배포하여 서비스예정이였지만, 최종 변경방식은 GCP의 Compute Engine을 통해 배포하는방식으로 진행하기로 하였습니다. 이벤트 서비스를 Compute Engine으로 배포하고 Compute Engine에 로드밸런서를 연결하여 블루/그린 방식의 배포 과정을 포스팅 하도록 하겠습니다. 추가로 다나와에서는 GitLab의 Runner 사용이 증가하고 있어 이번 배포 과정에서도 Runner를 통해 배포하도록 하겠습니다."
date:   2020.04.01.
writer: "김준우"
categories: GCP
---
## 소개

얼마전에 이벤트 서비스를 GCP의 Cloud Run에 배포하여 서비스예정이였지만, 최종 변경방식은 GCP의 Compute Engine을 통해 배포하는방식으로 진행하기로 하였습니다. 이벤트 서비스를 Compute Engine으로 배포하고 Compute Engine에 로드밸런서를 연결하여 롤링 방식의 배포 과정을 포스팅 하도록 하겠습니다. 추가로 다나와에서는 GitLab의 Runner 사용이 증가하고 있어 이번 배포 과정에서도 Runner를 통해 배포하도록 하겠습니다.

GCP에서 사용하는 서비스

- Compute Engine
- Cloud Source Repositories
- Load Balancer

## 구성맵

### 빌드

1. 빌드구간에는 Cloud Source Repositories에 배포할 소스를 푸시를 합니다.
2. 인스턴스에 사용할 템플릿을 생성합니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled.png](/images/2020-04-01-GCP-VM-Instance/Untitled.png)

배포

1. 인스턴스 그룹의 인스턴스를 교체합니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled%201.png](/images/2020-04-01-GCP-VM-Instance/Untitled%201.png)

## 구성절차 및 방법

gitlab-ci.yml 내용입니다. 스크립트 부분은 포스팅을 위해 수정하였습니다.

### GitLab PipeLine

### Build

gitlab-ci.yml의 build입니다.   

**GCP_SOURCE_NAME:** GItLab의 VARIABLE을 설정하였습니다. 그래서 환경변수에서 조회가 가능합니다. cloud source repositories에 변경된 소스를 푸시합니다.   

**STARTUP_SCRIPT:** 타입을 파일로 설정하여 —metadata-from-file옵션을 지정하였습니다. 환경변수로 ${STARTUP_SCRIPT}을 확인해보면 파일의 경로가 출력됩니다. 명령어를 실행하면 템플릿이 생성됩니다. 

    Build:
      stage: build
      when: manual
      script:
    	  - gcloud source repos clone ${GCP_SOURCE_NAME}
    	  - git push
        - gcloud beta compute instance-templates create "event" \
    	  --machine-type=n1-standard-1 --network-tier=PREMIUM \
    		--maintenance-policy=MIGRATE \
    	  --service-account=<서비스 계정> \
    		--scopes=https://www.googleapis.com/auth/cloud-platform \
    		--image=centos-7-v20200309 \
    		--image-project=centos-cloud \
    		--boot-disk-size=200GB \
    		--boot-disk-type=pd-standard \
    		--tags=http-server,https-server \
    		--metadata-from-file startup-script="${STARTUP_SCRIPT}"

### Deploy

gitlab-ci.yml의 deploy입니다.   

RUNNING으로 시작하는 변수는 기존 배포된 정보를 조회하여 인스턴스 그룹명을 알아낸뒤 새로운 템플릿으로 교체합니다.

    GCP Rolling Deploy:
      stage: deploy
      when: manual
      script:
        - gcloud auth activate-service-account --key-file ${GCP_CREDENTIAL}
        - gcloud config set project ${GCP_PROJECT}
        - export RUNNING_INSTAINCE_GROUP=$(gcloud compute backend-services list --format="table(BACKENDS)"|tail -n 1|cut -d "/" -f 3)
        - gcloud compute instance-groups managed rolling-action start-update ${RUNNING_INSTAINCE_GROUP} --version template="${CI_PROJECT_NAME}-${CI_COMMIT_SHORT_SHA}" --zone=${GCP_ZONE} --quiet

### replicas

gitlab-ci.yml의 replicas입니다.   

기존 인스턴스의 수를 변경할때 사용합니다.

    GCP Instance Replicas:
      stage: replicas
      when: manual
      script:
        - echo "Do your deploy here"
        - gcloud auth activate-service-account --key-file ${GCP_CREDENTIAL}
        - gcloud config set project ${GCP_PROJECT}
        - export RUNNING_INSTAINCE_GROUP=$(gcloud compute backend-services list --format="table(BACKENDS)"|tail -n 1|cut -d "/" -f 3)
        - gcloud compute instance-groups managed resize ${RUNNING_INSTAINCE_GROUP} --size=${REPLICAS} --zone ${GCP_ZONE}

### GitLab의 variables

Runner에서 사용한 변수를 전부 variables에 등록해둔 상태입니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled%202.png](/images/2020-04-01-GCP-VM-Instance/Untitled%202.png)

### GCP의 Cloud Source Repositories

새로 생성되는 인스턴스에서 소스를 gitlab에서 가져가지 않고 gcp의 프로젝트 범위에서만 접근가능한 저장소를 만들었습니다. 

![/images/2020-04-01-GCP-VM-Instance/Untitled%203.png](/images/2020-04-01-GCP-VM-Instance/Untitled%203.png)

### 배포 테스트

Build → GCP Instance Staging → GCP Rolling Deploy 순서로 구성되어 있습니다. 순서대로 플레이 버튼을 클릭하면 됩니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled%204.png](/images/2020-04-01-GCP-VM-Instance/Untitled%204.png)

GCP Blue/Green Deploy 까지 버튼을 누르면 배포가 완료 됩니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled%205.png](/images/2020-04-01-GCP-VM-Instance/Untitled%205.png)

소스에 컴밋 리비전을 출력하도록 하였습니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled%206.png](/images/2020-04-01-GCP-VM-Instance/Untitled%206.png)

리비전값이 변경된걸 확인할수 있습니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled%207.png](/images/2020-04-01-GCP-VM-Instance/Untitled%207.png)

## 정리

gitlab-ci.yml 코드기반으로 배포를 진행해서 버전관리가 되고, StartUp-Script기반으로 인스턴스를 생성하면 별도의 템플릿을 만드는 작업이 편해질걸로 보입니다. 인스턴스 그룹으로 생성을하게 되면 상태에 따라서 인스턴스 갯수를 자동조절하고, 장애가 발생한 인스턴스는 자동복구가 되는 기능을 사용하게 되면 대량의 요청이 오는 서비스에 대응이 될걸로 보입니다. 이상으로 이벤트 서비스 GCP 배포 방법을 소개하였습니다. 감사합니다.

## 참고링크

- [Google Cloud]([https://cloud.google.com/](https://cloud.google.com/))
- [GitLab CI/CD 사용하기]([https://danawalab.github.io/gitlab/2020/03/11/GitLab-GitLab_CI_CD.html](https://danawalab.github.io/gitlab/2020/03/11/GitLab-GitLab_CI_CD.html))
- [이벤트서비스 GCP 전환 PoC]([https://danawalab.github.io/gcp/2020/02/18/GCP-Cloud-Run-stress-testing.html](https://danawalab.github.io/gcp/2020/02/18/GCP-Cloud-Run-stress-testing.html))
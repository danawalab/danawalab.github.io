# 이벤트 서비스 GCP VM Instance 배포

## 소개

얼마전에 이벤트 서비스를 GCP의 Cloud Run에 배포하여 서비스예정이였지만, 최종 변경방식은 GCP의 Compute Engine을 통해 배포하는방식으로 진행하기로 하였습니다. 이벤트 서비스를 Compute Engine으로 배포하고 Compute Engine에 로드밸런서를 연결하여 블루/그린 방식의 배포 과정을 포스팅 하도록 하겠습니다. 추가로 다나와에서는 GitLab의 Runner 사용이 증가하고 있어 이번 배포 과정에서도 Runner를 통해 배포하도록 하겠습니다.

GCP에서 사용하는 서비스

- Compute Engine
- Cloud Source Repositories
- Load Balancer

## 구성맵

### 빌드

1. 빌드구간에는 Cloud Source Repositories에 배포할 소스를 푸시를 합니다.
2. 인스턴스에 사용할 템플릿을 생성합니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled.png](/images/2020-04-01-GCP-VM-Instance/Untitled.png)

### 스테이징

1. Instance Group을 생성
2. Instance의 StartUp-Script로 환경 구성및 Cloud Source Repositoies에서 소스 Pull

![/images/2020-04-01-GCP-VM-Instance/2020-04-01_09h30_04.png](/images/2020-04-01-GCP-VM-Instance/2020-04-01_09h30_04.png)

배포

1. 로드 밸런서 스위칭

![/images/2020-04-01-GCP-VM-Instance/2020-04-01_09h30_14.png](/images/2020-04-01-GCP-VM-Instance/2020-04-01_09h30_14.png)

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

### Staging

gitlab-ci.yml의 staging입니다.   

**GCP_CREDENTIAL:** 서비스계정의 키파일을 GitLab variable에 등록 해두었습니다.

**GCP_PROJECT:** gcloud 명령어마다 프로젝트 옵션을 추가하는게 번거로워 config에 설정합니다.

**REPLICAS:** variable에 등록 해두면 기본 인스턴스 수가 지정됩니다.

**GCP_ZONE:** 기본 영역을 지정합니다.

**set-autoscaling:** 오토 스케일링을 추가하면 인스턴스 상태에 따라서 GCP 기능을 사용가능합니다.

**set-named-ports:** 를 지정하게 되면 로드밸런서에서 인스턴스 그룹의 포트로 상태를 확인합니다.

    GCP Instance Staging:
      stage: staging
      when: manual
      script:
        - gcloud auth activate-service-account --key-file ${GCP_CREDENTIAL}
        - gcloud config set project ${GCP_PROJECT}
        - gcloud compute instance-groups managed create event \
    				--base-instance-name event \
    				--template event \
    				--size ${REPLICAS} \
    				--zone ${GCP_ZONE}
        - gcloud beta compute instance-groups managed set-autoscaling event \
    				--zone ${GCP_ZONE} --cool-down-period "60" --max-num-replicas "10" \ 
    				--min-num-replicas "1" --target-cpu-utilization "0.6" --mode "on"
        - gcloud compute instance-groups managed set-named-ports event \
    				--named-ports=http-in:80,https-in:443 \
    				--zone=${GCP_ZONE}

### Deploy

gitlab-ci.yml의 deploy입니다.   

RUNNING으로 시작하는 변수는 기존 배포된 정보를 조회하고 있습니다. RENEWAL으로 시작하는 변수는 이번에 배포하는 정보입니다. 각 정보는 조회하여 로드밸런서는 교체하는 작업을 진행합니다.  중간에 while 문을 사용하여 백앤드 서비스의 상태가 helath인지 체크하는 부분은 필수입니다.

    GCP Blue/Green Deploy:
      stage: deploy
      when: manual
      script:
        - gcloud auth activate-service-account --key-file ${GCP_CREDENTIAL}
        - gcloud config set project ${GCP_PROJECT}
        - export RUNNING_INSTAINCE_GROUP=$(gcloud compute backend-services list --format="table(BACKENDS)"|tail -n 1|cut -d "/" -f 3)
        - echo "${RUNNING_INSTAINCE_GROUP}"
        - export RUNNING_BACKEND_SERVICE="${RUNNING_INSTAINCE_GROUP}-backend-service"
        - echo "${RUNNING_BACKEND_SERVICE}"
        - export RUNNING_LB="${RUNNING_INSTAINCE_GROUP}-lb"
        - echo "${RUNNING_LB}"
        - export RENEWAL_INSTANCE_GROUP="event"
        - echo "${RENEWAL_INSTANCE_GROUP}"
        - export RENEWAL_BACKEND_SERVICE="${RENEWAL_INSTANCE_GROUP}-backend-service"
        - echo "${RENEWAL_BACKEND_SERVICE}"
        - export RENEWAL_LB="${RENEWAL_INSTANCE_GROUP}-lb"
        - echo "${RENEWAL_LB}"
        - gcloud compute backend-services create ${RENEWAL_BACKEND_SERVICE} \
    				--connection-draining-timeout 0 \
    				--global --health-checks event-health-check \
    				--port-name http-in --protocol http --quiet
        - gcloud compute backend-services add-backend ${RENEWAL_BACKEND_SERVICE} \
    				--instance-group ${RENEWAL_INSTANCE_GROUP} \
    				--instance-group-zone ${GCP_ZONE} --global --quiet
        - gcloud compute url-maps create ${RENEWAL_LB} \
    				--default-service ${RENEWAL_BACKEND_SERVICE} --quiet
        - export health="PENDING"
        - while [ "$health" != "HEALTHY" ]
    				do 
    					export health="$(gcloud compute backend-services get-health ${RENEWAL_BACKEND_SERVICE} \
    															--global --quiet|grep "healthState:"|cut -d ":" -f 2|tr -d ' '
    					)"
    					echo $health
    					sleep 1
    				done
        - sleep 120
        - gcloud compute target-http-proxies update event-http-target-proxy --url-map ${RENEWAL_LB} --quiet
        - gcloud compute url-maps delete ${RUNNING_LB} --global --quiet
        - gcloud compute backend-services delete ${RUNNING_BACKEND_SERVICE} --global --quiet
        - gcloud compute instance-groups managed delete ${RUNNING_INSTAINCE_GROUP} --zone ${GCP_ZONE} --quiet
        - gcloud compute instance-templates delete ${RUNNING_INSTAINCE_GROUP} --quiet

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

![/images/2020-04-01-GCP-VM-Instance/Untitled-1.png](/images/2020-04-01-GCP-VM-Instance/Untitled-1.png)

### GCP의 Cloud Source Repositories

새로 생성되는 인스턴스에서 소스를 gitlab에서 가져가지 않고 gcp의 프로젝트 범위에서만 접근가능한 저장소를 만들었습니다. 

![/images/2020-04-01-GCP-VM-Instance/Untitled-2.png](/images/2020-04-01-GCP-VM-Instance/Untitled-2.png)

### 배포 테스트

Build → GCP Instance Staging → GCP Blue/Green Deploy 순서로 구성되어 있습니다. 순서대로 플레이 버튼을 클릭하면 됩니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled-3.png](/images/2020-04-01-GCP-VM-Instance/Untitled-3.png)

GCP Blue/Green Deploy 까지 버튼을 누르면 배포가 완료 됩니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled-4.png](/images/2020-04-01-GCP-VM-Instance/Untitled-4.png)

소스에 컴밋 리비전을 출력하도록 하였습니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled-5.png](/images/2020-04-01-GCP-VM-Instance/Untitled-5.png)

리비전값이 변경된걸 확인할수 있습니다.

![/images/2020-04-01-GCP-VM-Instance/Untitled-6.png](/images/2020-04-01-GCP-VM-Instance/Untitled-6.png)

## 정리

gitlab-ci.yml 코드기반으로 배포를 진행해서 버전관리가 되고, StartUp-Script기반으로 인스턴스를 생성하면 별도의 템플릿을 만드는 작업이 편해질걸로 보입니다. 인스턴스 그룹으로 생성을하게 되면 상태에 따라서 인스턴스 갯수를 자동조절하고, 장애가 발생한 인스턴스는 자동복구가 되는 기능을 사용하게 되면 대량의 요청이 오는 서비스에 대응이 될걸로 보입니다. 이상으로 이벤트 서비스 GCP 배포 방법을 소개하였습니다. 감사합니다.

## 참고링크

- [Google Cloud]([https://cloud.google.com/](https://cloud.google.com/))
- [GitLab CI/CD 사용하기]([https://danawalab.github.io/gitlab/2020/03/11/GitLab-GitLab_CI_CD.html](https://danawalab.github.io/gitlab/2020/03/11/GitLab-GitLab_CI_CD.html))
- [이벤트서비스 GCP 전환 PoC]([https://danawalab.github.io/gcp/2020/02/18/GCP-Cloud-Run-stress-testing.html](https://danawalab.github.io/gcp/2020/02/18/GCP-Cloud-Run-stress-testing.html))
---
layout: post
title:  "GitLab CI/CD 사용하기"
description: "GitLab에서 제공하는 Runner를 사용하여 저장소에 Push를 하게 되면 자동으로 프로젝트를 빌드, 배포하는 과정을 포스팅하였습니다. "
date:   2020.03.11.
writer: "김준우"
categories: GitLab
---
## 소개

GitLab에서 제공하는 Runner를 사용하여 저장소에 Push를 하게 되면 자동으로 프로젝트를 빌드, 배포하는 과정을 포스팅하였습니다. 

![/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled.png](/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled.png)

## Gitlab 기본 구성

GitLab, Docker, Kubernetes 설치과정은 생략하고 진행하겠습니다.    

### 설치 버전

- gitlab: 12.8.5
- Docker 19.03.7
- kubernetes: 1.16.4

[Download and install GitLab]([https://about.gitlab.com/install/](https://about.gitlab.com/install/))

[Install Docker]([https://docs.docker.com/install/](https://docs.docker.com/install/))

[install-kubeadm]([https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/))

### GitLab과 Kubernetes 연동

GitLab에서 Kubernetes 연동은 프로젝트단위, 그룹단위, root 단위별로 연동이 가능합니다. 연동하는방식은 동일하며 저희는 Root계정으로 쿠버네티스 연동하여 전체 프로젝트에서  Shared Runner를 사용해볼 예정입니다.

### Root 계정으로 접속

![/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h44_02.png](/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h44_02.png)

### Kubernetes 메뉴 접속

첫화면에서 상단의 관리자영역에 접속하여 왼쪽메뉴에 보면 있습니다.

![/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h44_55.png](/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h44_55.png)

### Kubernetes 연동 설정

GCP에서 제공하는 서비스를 사용해도 되지만 이번에는 저희가 직접 구축한 Kubernetes 클러스터에 연결해보도록 하겠습니다. kubernetes에 설치된 서버에서 아래 명령어를 호출하여 결과를 GitLab에 입력해주세요.

![/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h45_15.png](/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h45_15.png)

API URL 조회

    $ kubectl cluster-info | grep 'Kubernetes master' | awk '/http/ {print $NF}'

CA Certificate 조회

    $ kubectl get secret $(kubectl get secrets|grep default-token|cut -d " " -f 1) -o jsonpath="{['data']['ca\.crt']}" | base64 --decode

Service Token

    $ cat <<EOF> gitlab-admin-service-account.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: gitlab-admin
      namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: gitlab-admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: gitlab-admin
      namespace: kube-system
    EOF
    
    $kubectl apply -f gitlab-admin-service-account.yaml --username=admin --password=admin

입력 후 저장을 누르면 아래 화면이 나옵니다. Helm Tiller의 Install를 눌러주세요.

![/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h46_30.png](/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h46_30.png)

설치가 완료되었으면 GitLab Runner도 동일하게 Install 버튼을 눌러주세요.

![/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h57_25.png](/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-10_09h57_25.png)

### 설치확인

Runner까지 설치가 되었으면 Runners 메뉴를 접속하여 Runner가 생성이 되었는지 확인합니다. 기본적으로 Runner는 Shared 상태입니다. 이것으로 연동은 root 가아닌 다른계정으로 다시 로그인합니다.

![/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled%201.png](/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled%201.png)

## CI/CD 구성

### 샘플 프로젝트 생성

CI/CD를 테스트 해보기위해선 샘플 프로젝트가 필요합니다. Simple Spring Boot으로 Hello World 출력하는 프로젝트를 생성하겠습니다. 

[ExampleController.java](http://examplecontroller.java) 

    @RestController
    public class exampleController {
    
        @GetMapping("/")
        public ResponseEntity<?> helloWorld(@RequestParam(defaultValue = "world") String foo) {
            return new ResponseEntity<>(String.format("Hello %s !", foo), HttpStatus.OK);
        }
    
    }

gitlab-ci.yml 파일 생성

GitLab CI/CD는 gitlab-ci.yml 설정한 대로 수행하게 됩니다. 간략하게 설명하게 되면 docker-build때는 image는 docker:latest 컨테이너에서 실행하여 script의 라인을 실행하게 됩니다. 그럼 docker build 와 images 조회가 됩니다. docker-deploy는 image는 docker:latest 사용하고 stage는 deploy 입니다. script에서는 docker run을 호출하고 있습니다.

    docker-build:
      image: docker:latest
      stage: build
      before_script:
        - echo "run.."
      script:
        - docker -H $CI_REGISTRY build -t "$CI_REGISTRY_IMAGE" .
        - docker -H $CI_REGISTRY images
      only:
        - master
    
    docker-deploy:
      image: docker:latest
      stage: deploy
      script:
        - docker -H $CI_REGISTRY run -d --rm -p 8080:8080 --name "$CI_REGISTRY_IMAGE" "$CI_REGISTRY_IMAGE"
      only:
        - master

Dockerfile 파일 생성

Dockerfile에서는 프로젝트를 package하여 이미지에 등합니다. CMD에서 실행명령어를 작성하였습니다.

    FROM maven:3.5-jdk-11 as BUILD
    
    COPY . /usr/src/app
    RUN mvn --batch-mode -f /usr/src/app/pom.xml clean package
    
    FROM openjdk:11-jdk
    ENV PORT 4567
    EXPOSE 4567
    COPY --from=BUILD /usr/src/app/target /opt/target
    WORKDIR /opt/target
    
    CMD ["java", "-jar", "demo-0.0.1-SNAPSHOT.jar"]

환경변수 등록

settings의 CI/CD 페이지에서 중간에 보이는 variables 를 펼쳐서 아래 항목에 docker 정보를 입력합니다. 

 CI_REGISTRY:  docker ip:port

CI_REGISTRY_IMAGE:  이미지명

![/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled%202.png](/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled%202.png)

프로젝트 최종 모습

![/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled%203.png](/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled%203.png)

## CI/CD 테스트 하기

프로젝트를 저장소에 Push를 하게되면 자동으로 job이 생성되고 pipline이 실행됩니다. 

아래 이미지는 빌드 진행과정입니다.

![/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-11_12h45_53.png](/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-11_12h45_53.png)

빌드가 끝나면 자동으로 배포가 진행됩니다. 아래 이미지는 배포 중인 이미지 입니다.

![/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-11_12h48_14.png](/images/2020-03-11-GitLab-GitLab_CI_CD/2020-03-11_12h48_14.png)

생성된 컨테이너에 접속해봅니다 . 우리가 원했던 Hello world가 나옵니다.

![/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled%204.png](/images/2020-03-11-GitLab-GitLab_CI_CD/Untitled%204.png)

## 정리

이번에 GitLab과 Kubernetes를 연동하여 빌드와 배포를 진행해보았습니다. 이번 과정중에 중요한건 gitlab-ci.yml 파일 이였던거 같습니다. 작성하는 스크립트는 bash 스크립트를 사용할줄아는 개발자면 어렵지 않게 작성이 가능해 보였습니다. 빌드와 배포를 파일 하나로 관리할 수 있다는 장점이 있었고, gitlab-ci.yml파일도 동일하게 버전관리를 할 수 있는 장점이 있었습니다.  나중에 기회가 되면 Knative 연동하는 부분을 포스팅해보도록 하겠습니다.

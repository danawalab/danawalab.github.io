---
layout: post
title:  "GitLab CI/CD cache with Kubernetes"
description: "이번 포스팅은 GitLab 과 k8s 연동 후 CI/CD 시 cache 사용방법에 대해 공유합니다. GitLab 에서는 기본 캐시가 비활성화 되어 있습니다. 공식문서에서는 s3, gcs를 사용하하는 방법만 제공하고 있습니다. 임시 캐시하기위해서 퍼블릭 저장소를 사용하기엔 무리가 있어 보여 s3와 API 100% 동일하다는 minio를 사용하여 적용해보도록 하겠습니다."
date:   2020.04.14.
writer: "김준우"
categories: GitLab
---
## 소개

안녕하세요. 다나와 김준우입니다. 이번 포스팅은 GitLab 과 k8s 연동 후 CI/CD 시 cache 사용방법에 대해 공유합니다. GitLab 에서는 기본 캐시가 비활성화 되어 있습니다. 공식문서에서는 s3, gcs를 사용하하는 방법만 제공하고 있습니다. 임시 캐시하기위해서 퍼블릭 저장소를 사용하기엔 무리가 있어 보여 s3와 API 100% 동일하다는 minio를 사용하여 적용해보도록 하겠습니다.

### 진행 순서

GitLab에서 Runner로 빌드 요청을한다.

Runner는 Build pod를 생성한다.

Build Pod는 Minio에 캐시를 업로드 또는 다운로드한다.

![/images/2020-04-14-GitLab-CI-CD-cache-with-Kubernetes/Untitled.png](/images/2020-04-14-GitLab-CI-CD-cache-with-Kubernetes/Untitled.png)

### 미니오란?

미니오는 아마존 웹서비스의 저장소인 s3와 동일한 api를 제공하는 오브젝트 스토리지입니다. 미니오는 사진, 비디오, 파일, 컨테이너 이미지같은 구조화 되지 않은 데이터를 저장할 수 있고, 객체 당 최대 크기가 5TB 까지 가능합니다. 

## 구성방법

저희는 미니오를 도커로 간단하게 실행하여 사용하겟습니다. 

    docker run -d --rm --name minio \
           -e "MINIO_ACCESS_KEY=MINIOKEY" \
           -e "MINIO_SECRET_KEY=MINIOSECRET" \
           -p 9000:9000 \
           minio/minio server /data

[http://127.0.0.1:9000](http://127.0.0.1:9000) 접속합니다. gitlab 버킷을 생성합니다.

![/images/2020-04-14-GitLab-CI-CD-cache-with-Kubernetes/Untitled%201.png](/images/2020-04-14-GitLab-CI-CD-cache-with-Kubernetes/Untitled%201.png)

깃랩과 쿠버네티스 연동과정은 생략 하도록 하겠습니다. 지난번 포스팅을 참고하시면 됩니다.

k8s의 Runner Deployment를 수정합니다.

    $ kubectl edit deployment runner-gitlab-runner -n gitlab-managed-apps

env에 아래 내용 추가

    ... (생략)
    env:
    - name: CACHE_TYPE
      value: s3
    - name: CACHE_SHARED
      value: "true"
    - name: CACHE_S3_SERVER_ADDRESS
      value: 127.0.0.1:9000
    - name: CACHE_S3_ACCESS_KEY
      value: MINIOKEY
    - name: CACHE_S3_SECRET_KEY
      value: MINIOSECRET
    - name: CACHE_S3_BUCKET_NAME
      value: gitlab
    - name: CACHE_S3_INSECURE
      value: "true"
    - name: S3_CACHE_PATH
      value: /
    - name: BucketLocation
    ... (생략)

GitLab에 메이븐 프로젝트를 생성 후 .gitlab-ci.yml 파일에 아래내용을 생성합니다.

    image: maven:3.3.9-jdk-8
    cache:
      key: maven
      paths:
        - .m2/repository
    
    deploy:jdk8:
      stage: build
      only:
        - master
      script:
        - mvn package

소스를 푸시할때마다 프로젝트와 키를 기반으로 캐시를 다운받고 빌드가 끝날때 캐시를 다시 업로드 하는 로그를 확인 할 수 있습니다.

![/images/2020-04-14-GitLab-CI-CD-cache-with-Kubernetes/Untitled%202.png](/images/2020-04-14-GitLab-CI-CD-cache-with-Kubernetes/Untitled%202.png)

![/images/2020-04-14-GitLab-CI-CD-cache-with-Kubernetes/Untitled%203.png](/images/2020-04-14-GitLab-CI-CD-cache-with-Kubernetes/Untitled%203.png)

## 정리

캐시는 빌드나 배포에 속도를 줄이는데 사용하기 적합해 보입니다. 구글 서치 또는 스택오버플로우에 내용을 보면 s3 또는 gcs를 사용하거나 아니면 볼륨을 만들어 마운트하는 방식을 사용하기도 하지만, 이번 minio를 활용하여 쿠버네티스 클러스터와 분리하여 저장할 수 있고, MINIO와 쿠버네티스의 네트워크망을 같이 사용하여  빠르게 업로드/다운로드가 가능하였습니다.
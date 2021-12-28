---
layout: post
title:  "Github Actions 사용법"
description: "github actions를 사용하여 지속적인 빌드방법에 대해 알아보겠습니다. 다나와에서는 오픈소스로 dsearch 시리즈, service-management 등 범용적인 소프트웨어를 오픈소스로 공개하고 있습니다. 지금까지는 오픈소스를 CI/CD는 사내 Gitlab CI/CD를 통해 github의 소스를 다운받아 빌드 후 이미지를 사내 저장소에 저장하였습니다.  이슈가 있던건 아니지만 프로젝트 브랜치 전략, CI/CD 개선을 시작하며 오픈소스의  이미지도 github package로 오픈하기로 결정하였습니다. 이전 포스팅을 통해 package로 이미지 배포 방법에 대해 알아보았고, 이번에는 github actions를 통해 프로젝트 빌드, 이미지 빌드 후 package로 저장하는 방법에 대해 알아보도록 하겠습니다."
date:   2021-12-28
writer: "김준우"
categories: Common
---
## 개요

github actions를 사용하여 지속적인 빌드방법에 대해 알아보겠습니다. 다나와에서는 오픈소스로 dsearch 시리즈, service-management 등 범용적인 소프트웨어를 오픈소스로 공개하고 있습니다. 지금까지는 오픈소스를 CI/CD는 사내 Gitlab CI/CD를 통해 github의 소스를 다운받아 빌드 후 이미지를 사내 저장소에 저장하였습니다.  이슈가 있던건 아니지만 프로젝트 브랜치 전략, CI/CD 개선을 시작하며 오픈소스의  이미지도 github package로 오픈하기로 결정하였습니다. 이전 포스팅을 통해 package로 이미지 배포 방법에 대해 알아보았고, 이번에는 github actions를 통해 프로젝트 빌드, 이미지 빌드 후 package로 저장하는 방법에 대해 알아보도록 하겠습니다.

## workflow 추가 방법

dsearch-server 오픈소스의 actions를 추가 해보면서 사용법에 대해 알아보도록 하겠습니다.

### 템플릿 선택

프로젝트에서 actions 탭을 보면 기본제공하는 템플릿이 있습니다. Docker image, maven build 등등 제공합니다.  저희는 Docker Image를 선택합니다.

![Untitled](/images/2021-12-28-Github-Actions-사용법/Untitled.png)

### workflows 에디터

템플릿을 클릭하게 되면 프로젝트에 .github/workflows/docker-image.yml 파일이 생기게 됩니다. actions는 이렇게 여러개의 yml 파일이 workflow가 되고, yml내용 중에 jobs를 통해 단계별로 실행과정을 정의하게 됩니다.

![Untitled](/images/2021-12-28-Github-Actions-사용법/Untitled%201.png)

기본 템플릿 라인별 해석해보았습니다. 특이점으로 uses 통해 또다른 템플릿의 action을 호출할 수 있었습니다. 

```solidity
name: workflow의 이름입니다. 
on: workflow 이벤트 방식을 정의합니다. (push, MR, PR, schedule 등 이벤트 제공합니다)
jobs: 단계적인 job을 정의합니다.
build: 잡의 이름입니다. 
runs-on: 잡을 실행할 환경입니다. (windows, macos, ubuntu 등을 제공합니다)
steps: 잡의 명령 집합입니다.
uses: 또 다른 action을 호출할 수 있습니다. (소스 체크아웃)
name: 명령어 이름입니다.
run: 실제 실행할 명령어라인입니다.
```

### JDK 활성화

dsearch-server는 java언어로 개발되었습니다. 그렇기 때문에 빌드때 필요한 JDK 를 추가합니다.

직접 run필드를 통해 jdk 를 설치해도 되지만 저는 marketplace에 Setup Java JDK 액션을 사용합니다.

![Untitled](/images/2021-12-28-Github-Actions-사용법/Untitled%202.png)

```solidity
    - name: Setup Java JDK            // 액션 이름
      uses: actions/setup-java@v2.5.0 // 사용하는 액션의 명칭과 버전
      with:
        java-version: 8.0.312+7       // 사용할 JDK version
        distribution: adopt           // 사용할 JDK vender
```

### Maven Cache 활성화

메이번 빌드시 dependency 를 캐싱합니다. 

```solidity
- name: Maven Cache
  uses: skjolber/maven-cache-github-action@v1.1
    with:
      step: <restore or save>  // resotre 캐시가져옴, save 캐시 저장

```

### pom.xml 버전 추출

run은 개별적인 실행라인입니다. 변수로 다양하게 사용하기 위해서는 예약어인 $GITHUB_ENV에 저장 한뒤 다른 run 라인에서 사용할 수 있습니다.

```solidity
// EVN 값 저장
- name: extract version for pom.xml
   run: echo "VERSION=$(grep -oPm2 "(?<=<version>)[^<]+" pom.xml | sed -n 2p)" >> $GITHUB_ENV
// ENV 값 조회
- name: echo version
   run: echo "${{ env.VERSION }}"
```

### 메이븐 빌드

이미지에 추가할 jar를 빌드합니다.

```solidity
- name: Spring Boot Build with Maven
   run: mvn package -Dskip.test=true
```

### 컨테이너 저장소 로그인

[ghcr.io](http://ghcr.io) 컨테이너 저장소에 이미지를 추가하기 위해서는 인증으로 토큰을 추가해야합니다. 하지만 토큰을 코드에 코드에 하드코딩하게 되면 보안에 취약하기 때문에 github에서는 secrets을 제공합니다.

프로젝트에서 settings에 보면 sercrets 메뉴가 있습니다. New repository secret을 클릭하여 CR_TOKEN이름으로 packges 토큰을 추가합니다.

![Untitled](/images/2021-12-28-Github-Actions-사용법/Untitled%203.png)

다시 yml로 돌아와 docker login 액션을 추가합니다.

```solidity
- name: Docker Login
  uses: docker/login-action@v1.12.0
  with:
    registry: ghcr.io
    username: danawalab
    password: ${{ secrets.CR_TOKEN }}
```

### 도커 이미지 빌드&푸시

빌드와 푸시의 액션을 통해 이미지를 빌드한 후 [ghcr.io](http://ghcr.io) 로 이미지를 푸시합니다.

```solidity
- name: Docker Build & Push Action
  uses: mr-smithers-excellent/docker-build-push@v5.6
    with:
      registry: ghcr.io
      buildArgs: VERSION=${{env.VERSION}},PROJECT_NAME=${{ env.DOCKER_IMAGE }}
      image: ${{ env.DOCKER_IMAGE }}
      tags: ${{ env.VERSION }}
```

### 전체 코드
단계별 코드외 추가한 내용은 env 항목입니다. env를 최상단에 위치하게 되면 모든 job영역에서 해당 env값을 참조 할 수 있게 됩니다.
```solidity
# 액션 이름
name: Dsearch Server CI

env:
  DOCKER_IMAGE: dsearch-server

# 마스터 푸시, 풀 리퀘스트 이벤트 발생하면 아래 잡들을 실행함.
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
# 빌드 잡 정의시작.
  build:  
#   실행 플랫폼은 ubuntu사용, runs: windows, mac os, ubuntu 있음.
    runs-on: ubuntu-latest
    steps:
#     소스코드 checkout
    - uses: actions/checkout@v2

#     JDK 활성화
    - name: Setup Java JDK
      uses: actions/setup-java@v2.5.0
      with:
        java-version: 8.0.312+7
        distribution: adopt

#     java maven 캐시 불러움
    - name: Maven Cache
      uses: skjolber/maven-cache-github-action@v1.1
      with:
        step: restore

#     pom.xml 버전을 추출합니다.
    - name: extract version for pom.xml
      run: echo "VERSION=$(grep -oPm2 "(?<=<version>)[^<]+" pom.xml | sed -n 2p)" >> $GITHUB_ENV

#     Maven 빌드
    - name: Spring Boot Build with Maven
      run: mvn package -Dskip.test=true
    
#     java maven 캐시 저장
    - name: Maven Cache
      uses: skjolber/maven-cache-github-action@v1.1
      with:
        step: save
    
    - name: file listing
      run: ls -al
      
#     ghcr.io 로그인
    - name: Docker Login
      uses: docker/login-action@v1.12.0
      with:
        registry: ghcr.io
        username: joonwoo8888
        password: ${{ secrets.CR_TOKEN }}

#     도커 이미지 빌드&저장
    - name: Docker Build & Push Action
      uses: mr-smithers-excellent/docker-build-push@v5.6
      with:
        registry: ghcr.io
        buildArgs: VERSION=${{env.VERSION}},PROJECT_NAME=${{ env.DOCKER_IMAGE }}
        image: ${{ env.DOCKER_IMAGE }}
        tags: ${{ env.VERSION }}
```

### 액션 실행해보기

작성한 yml파일을 마스터 브랜치에 푸시하면 actions탭에는 workflow가 실행되게 됩니다. 

![Untitled](/images/2021-12-28-Github-Actions-사용법/Untitled%204.png)

job의 진행 로그도 확인 할 수 있습니다.

![Untitled](/images/2021-12-28-Github-Actions-사용법/Untitled%205.png)

계정내의 Packages 탭으로 이동하여 이미지가 정상적으로 등록 되었는지 확인 할 수 있습니다.

![Untitled](/images/2021-12-28-Github-Actions-사용법/Untitled%206.png)

## 마무리

dsearch-server 오픈소스를 액션을 통해 이미지를 빌드하고 패키지에 저장하는 방법에 대해 알아보았습니다. github actions는 자주 사용하는 명령어에 대해 마켓플레이스가 있어 편하게 가져다 사용할 수 있었고 UI 또한 에디터와 우측에 마켓플레이스를 제공하기 때문에 어렵지 않게 사용할 수 있었습니다. 이렇게 간단하게 액션을 등록 해보았고, 조금 더 고도화하여 브랜치별로 빌드 유형을 다르게 구성해보도록 하겠습니다.
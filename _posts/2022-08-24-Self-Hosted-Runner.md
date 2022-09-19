---
layout: post
title:  "Github Actions에 Self-hosted Runner 등록하기" 
description: "Github Actions에 Self-hosted Runner를 등록하는 방법에 대해 알아보겠습니다"
date:   2022.08.24.
writer: "선요한"
categories: Common
---

## Self-hosted Runner란 	

Self-hosted Runner란 Github Actions에서 사용자가 지정하는 로컬 컴퓨팅 자원으로 빌드를 수행하도록 설정하는 기능입니다. 주로 배포작업이 많아 배포가 지체되거나 서버 비용이 부담되는 경우 유용한 기능입니다. 쏙 프로젝트의 경우 macOS 기반의 Github-Hosted Runner의 비용이 부담이 되었기 때문에 iOS앱을 배포하는 목적으로 맥북을 Self-hosted Runner로 등록하여 사용했습니다.

 

## 설정 방법

Github Actions을 사용하고자 하는 저장소에서 Settings - Actions - Runners로 이동합니다.

![image](https://user-images.githubusercontent.com/54565079/186301036-88316be4-d540-4fd2-ad9f-447ae04fc4ed.png)

설정하고자 하는 로컬 머신에 해당되는 OS를 선택하면 OS 별로 설정하는 방법이 Download와 Configure란에 설명되어 있습니다. MacOS 기준으로는 다음과 같습니다. 

- Self-hosted Runner로 등록하고자 하는 기기에서 폴더 생성

```
$ mkdir actions-runner && cd actions-runner
```

- 가장 최신의 Runner 패키지 다운로드

```
$ curl -o actions-runner-osx-x64-2.295.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.295.0/actions-runner-osx-x64-2.295.0.tar.gz
```

- 다운로드 받은 압축파일 압축해제

```
tar xzf ./actions-runner-osx-x64-2.295.0.tar.gz
```

- 저장소 연결

```
./config.sh --url [저장소 주소] --token [토큰 값]
```

Github가 로컬머신에 접속하는 방식이 아닌 로컬머신에서 github 저장소로 접속하는 방식이기 때문에 github 저장소 주소와 액세스 토큰으로 설정해야 합니다. 

토큰의 경우 개인 계정 Settings - Developer settings - Personal access tokens - Generate new token 에서 admin:enterprise - manage_runners:enterprise로 발급받을 수도 있습니다. 





## 사용 방법

정상적으로 github에 등록이 되면 github의 Runners에서도 목록을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/54565079/186331778-bb3be1b4-315c-477d-8c90-d8a0b0c90e7f.png)

등록한 Self-hosted Runner를 활성화시키기 위해서는 해당 로컬 기기의 actions-runner 폴더에서 run.sh 프로그램을 실행시킵니다. 

```
./run.sh
```

이제 workflows 디렉토리 안에 actions 사용시 해당 self-hosted runner를 통해 빌드되도록 yaml 파일을 만들어주면 됩니다.  

![image](https://user-images.githubusercontent.com/54565079/186335727-bc9a4d4e-a30b-4d84-8f88-126722b5fa32.png)

이 yaml 파일의 내용은 다음과 같습니다.

- develop 브랜치에 푸쉬나 풀 리퀘스트하는 경우에 동작
- macOS로 등록된 self-hosted runner를 사용
- 가장 최근 커밋 메세지에 `#ios`  혹은 `#all` 이 있는 경우에만 동작
- 저장소의 프로젝트를 로컬 머신의 프로젝트와 동기화한 이후에 fastlane의 `ios_build_and_upload`라는 lane을 실행



쏙 앱의 경우 등록된 self-hosted runner가 한개라서 macOS 혹은 self-hosted라고 명시해도 되지만 등록된 self-hosted runner가 많을 경우 

```
runs-on: [self-hosted, linux, x64]
```

이런식으로 특징들을 조합해서 지정할 수 있습니다. 만약에 이 세가지 조합으로도 부족하다면 

```
runs-on: [self-hosted, linux, x64, ssok]
```

이런식으로 사용자 지정 라벨을 추가로 붙일 수도 있습니다. 



## 결과

develop 브랜치에 #all 혹은 #ios라는 메세지를 포함한 커밋메세지로 푸쉬나 풀리퀘스트 하는 경우 actions는 workflows에 명시된대로 macOS의 조건에 부합하는 self-hosted runner를 찾아서 workflow에 명시한 작업들을 실행하게 됩니다. 

![image](https://user-images.githubusercontent.com/54565079/186334904-e40940cd-dbc9-4e80-a223-0536829d8277.png)



등록한 self-hosted runner가 잘 동작하하는 것을 확인할 수 있습니다. 




## 정리

Self-Hosted Runner의 장점으로는 Github-hosted Runner와 달리 사용 비용이 전혀 없기 때문에 배포작업이 많은 경우 유용한 기능이라고 할 수 있습니다. 하지만 사용하는 시점에 Github와 연결이 되어있어야 하고 로컬머신에 대한 관리가 추가로 필요하다는 단점이 있습니다. 배포하는 상황에 따라서 적절히 로컬 머신을 연결하여 배포용 서버로 사용하면 좋을 것 같습니다. 




## 참고 자료
- <https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners>
- <https://mildwhale.github.io/2021-04-24-build-machine-with-m1-macmini/>
- <https://www.hahwul.com/2022/07/05/macos-github-action-runner/>

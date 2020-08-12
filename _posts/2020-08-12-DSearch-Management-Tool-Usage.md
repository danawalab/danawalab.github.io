---
layout: post
title:  "디서치 관리도구 사용법  #1 - 소개"  # 리스트에 보여질 제목
description: "디서치 관리도구 사용법에 대해 전체적으로 메뉴를 소개하도록 하겠습니다. 하단에 링크를 첨부할 예정이니 좀 더 자세히 알고 싶으시다면 링크를 타고 이동하셔서 글을 읽어주시면 감사하겠습니다." 
date:   2020.08.12. 
writer: "선지호"  
categories: Elastic 
---

## 소개

이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)를 소개하는 시간을 가지도록 하겠습니다.

![/images/2020-08-12-DSearch-Management-Tool-Usage/Untitled.png](/images/2020-08-12-DSearch-Management-Tool-Usage/Untitled.png)

## DSearch 관리도구 소개

엘라스틱서치를 조금 더 간편하게, 또 한눈에 알아볼 수 있게 사용하기 위해 다나와에서 개발한 엘라스틱서치 전용 관리도구입니다.

디서치는 각각 콘솔과 서버 두개의 파트로 나누어져 있으며, 이 관리도구를 통해 엘라스틱서치를 등록하고 사용하는 방법을 소개하고자 합니다.

### 목차
- 사전
- 분석도구
- 랭킹튜닝
- 클러스터
- 컬렉션
- 템플릿
- 인덱스
- JDBC
- 레퍼런스UI
- 서버
- API
- 클러스터설정


### DSearch 관리도구 콘솔 환경

nvm 버전: 1.1.7
npm 버전: 6.14.4
yarn 버전: 1.22.4

### DSearch 관리도구 콘솔 설치 및 실행 방법

```jsx
$ yum install git -y
$ rm -rf yarn.lock
$ rm -rf package-lock.json
$ rm -rf node_modules
$ npm cache clean -f
$ npm install
$ npm install -g yarn
$ yarn build --reset-cache
$ npm start
```

### DSearch 관리도구 서버 환경

자바 버전: 1.8 이상
엘라스틱서치 버전: 7.6.2

### DSearch 관리도구 서버 설치 및 실행 방법

```jsx
$ git clone https://github.com/danawalab/dsearch.git
$ cd dsearch
$ mvnw clean
$ mvnw package -DskipTests
$ cd target
$ java -jar fastcatx-server.jar
```


### 로그인 방법

첫 화면은 이렇습니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_1.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_1.png)

여기에 각 데이터를 입력하고, 로그인을 누릅니다. 
서버는 디서치 관리도구 서버의 접속 경로를 입력해 주셔야 합니다.
루트 계정은 

이메일: admin@example.com / 비밀번호: admin  

입니다
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_2.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_2.png)

이렇게 엘라스틱서치 클러스터 등록 및 관리 페이지가 나옵니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_3.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_3.png)


### 클러스터 추가 방법

신규 클러스터 추가 버튼을 클릭하면, 
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_4.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_4.png)
이렇게 화면이 나옵니다.

여기에 이름, 엘라스틱서치 클러스터 접속 경로, 엘라스틱서치와 연결된 키바나의 접속경로를 입력(optional)하고 연결테스트를 클릭해 줍니다.
연결테스트를 클릭하지 않는다면 추가 버튼이 활성화 되지 않습니다.
만약 색상을 다른 색상으로 변경하고 싶다면 색상 버튼을 클릭하여 변화를 줄 수 있습니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_5.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_5.png)
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_6.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_6.png)
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_7.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_7.png)


### 역할 
기본 역할은 SYSTEM_MANAGER 입니다. 모든 권한을 가지고 있습니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_8.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_8.png)

역할을 추가하고 싶으시다면 우측 상단의 버튼을 클릭하여 추가 버튼을 눌러주세요.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_10.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_10.png)

아래 이미지와 같이 역할에 대한 권한 및 이름을 주어 새로운 역할을 만들 수 있습니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_11.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_11.png)

만약 역할 삭제가 팔요하다면, 역할이 사용자에게 부여되어 있을 때, 삭제버튼이 활성화 되지 않으므로 사용자에게 부여된 역할을 변경 및 사용자 삭제를 먼저 하시고 역할 삭제를 진행하셔야 합니다. 

### 사용자

기본 사용자는 admin 입니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_9.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_9.png)

사용자 추가를 원하신다면 우측 상단의 버튼을 클릭하여 추가 버튼을 눌러주세요.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_12.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_12.png)

역할 버튼을 눌러 역할을 부여 할 수 있습니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_13.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_13.png)

사용자 생성이 끝나면, 임시 비밀번호가 부여됩니다. 이는 1번밖에 보여지지 않으므로 복사 및 다른 곳에 적어두는 것을 추천 드립니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_14.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_14.png)

비밀번호를 변경하기 위해서는 우측 상단의 이메일이 나와있는 버튼을 클릭해서 정보 수정을 들어가셔야 합니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_15.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_15.png)

이곳에서 비밀번호로 발급된 임시 비밀번호를 입력하고, 변경할 비밀번호를 입력해서 사용자 비밀번호를 수정하실수 있습니다.
![/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_16.png](/images/2020-08-12-DSearch-Management-Tool-Usage/2020-08-12_16.png)

## 정리

이번에는 다나와에서 만든 디서치 관리도구를 설치하고, 사용하는 방법에 대해 포스팅 해보았습니다. 엘라스틱 서치를 사용하면서 느꼈던 여러 불편한 부분들이 키바나를 통해 많이 해소되었지만, 그럼에도 불구하고 해소되지 못한 부분이 있고, 직관적이지 않은 부분이 많았기에 디서치 관리도구를 사용하며 이를 최소화하려고 했습니다. 많은 분들이 이용하고, 피드백을 주셨으면 좋겠습니다.  
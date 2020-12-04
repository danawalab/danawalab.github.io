---
layout: post
title:  "서비스 운영플랫폼 #4 - 그룹"
description: "서비스 운영플랫폼의 그룹은 서비스를 묶은 개념입니다. 그룹을 생성하고, 서버, 서비스를 추가하여 사용자한테 관리 역할을 부여할 수 있습니다." 
date:   2020.12.04. 19:00:00
writer: "김준우"  
categories: Management
---
## 소개

서비스 운영플랫폼의 그룹은 서비스를 묶은 개념입니다. 그룹을 생성하고, 서버, 서비스를 추가하여 사용자한테 관리 역할을 부여할 수 있습니다.

## 그룹 설명

그룹 메뉴를 클릭하게 되면 소속된 그룹목록이 표시 됩니다. 관리자로 접근시 생성된 모든 그룹이 표시됩니다.

![/images/2020-12-04-service-management-4/Untitled.png](/images/2020-12-04-service-management-4/Untitled.png)

그룹생성은 목록에서 그룹추가 버튼을 클릭하면 아래 이미지 처럼모달이 표시됩니다. 이름과 설명을 입력 후 추가 버튼을 클릭하면 됩니다. 그룹 이름은 시스템에 유니크해야됩니다.

![/images/2020-12-04-service-management-4/Untitled%201.png](/images/2020-12-04-service-management-4/Untitled%201.png)

생성한 그룹을 클릭하면 아래 이미지 처럼 그룹의 상세 정보가 보여집니다. 이름과 설명이 표시되고 우측엔 그룹수정, 그룹삭제를 할 수 있는 메뉴버튼이 있습니다. 중앙에는 서비스탭, 서버탭, 권한탭으로 구성되어 있습니다.

![/images/2020-12-04-service-management-4/Untitled%202.png](/images/2020-12-04-service-management-4/Untitled%202.png)

### 서비스탭 추가

아래 이미지는 서비스 추가 버튼을 클릭했을때 컨테이너 타입의 서비스 추가 화면입니다.

- 이름: 서비스 식별 이름
- 서버: 어드민 사용자가 할당 해준 서버 중 하나를 선택
- 타입: 도커 컴포즈 방식, 프로세스 타입 선택
- 변수: yaml내용에서 키위치에 값 내용을 맵핑
- yaml: 도커 컴포즈 내용

![/images/2020-12-04-service-management-4/Untitled%203.png](/images/2020-12-04-service-management-4/Untitled%203.png)

프로세스 타입의 서비스를 등록하는 화면입니다. 

- 이름: 서비스 식별 이름
- 서버: 어드민 사용자가 할당 해준 서버 중 하나를 선택
- 타입: 도커 컴포즈 방식, 프로세스 타입 선택
- PID: 프로세스 PID 조회하는 명령어
- 시작스크립트:  프로세스 시작하는 스크립트
- 종료스크립트: 프로세스 종료하는 스크립트
- 로그: 로그 수집할 경로

![/images/2020-12-04-service-management-4/Untitled%204.png](/images/2020-12-04-service-management-4/Untitled%204.png)

![/images/2020-12-04-service-management-4/Untitled%205.png](/images/2020-12-04-service-management-4/Untitled%205.png)

서비스를 추가하고 나면 아래 이미지 처럼 설정한 정보가 표시되고, 프로세스 상태를 조회하여 각 정보를 표시 합니다. 실행 버튼을 통해 시작, 종료를 할 수있습니다. 컨테이너 타입은 업데이트 후 재시작 기능 까지 지원합니다.

![/images/2020-12-04-service-management-4/Untitled%206.png](/images/2020-12-04-service-management-4/Untitled%206.png)

### 서버탭

서버탭에서는 어드민 사용자가 그룹에 할당한 서버 목록을 표시합니다. 

![/images/2020-12-04-service-management-4/Untitled%207.png](/images/2020-12-04-service-management-4/Untitled%207.png)

서버의 상세화면에서는 서버의 정보와 빠른 명령어버튼을 클릭하여 서버의 상태를 조회할 수 있습니다. 터미널 열기를 통해 WebUI를 통해 터미널 접속도 가능합니다.

![/images/2020-12-04-service-management-4/Untitled%208.png](/images/2020-12-04-service-management-4/Untitled%208.png)

### 권한탭

권한탭은 그룹에 할당된 사용자를 표시합니다. 그룹에 포함되지 않은 사용자를 추가할 수 있습니다. 그룹에 소속된 사용자면 권한 상관없이 사용자를 추가 할 수 있습니다.

![/images/2020-12-04-service-management-4/Untitled%209.png](/images/2020-12-04-service-management-4/Untitled%209.png)

## 정리

그룹메뉴는 서비스 운영플랫폼의 핵심기능을 가지고 있습니다. 명령어를 원격으로 사용하여 컨테이너, 프로세스를 관리합니다.
---
layout: post
title:  "서비스 운영플랫폼 #3 - 설정"
description: "서비스 운영플랫폼에서 설정은 관리자 전용 메뉴입니다. 설정에서는 그룹에 소속되지 않은 서비스를 전부 조회 또는 검색하는 서비스탭과 서버를 추가하여 그룹에 할당하는 서버탭, 사용자를 관리하는 사용자 탭으로 구성되어 있습니다. 서비스 운영플랫폼을 시작하면 어드민 계정이 자동생성됩니다. 아이디는 admin@example.com, 비밀번호는 admin 입니다." 
date:   2020.12.04. 18:00:00
writer: "김준우"  
categories: Management
---
## 소개

서비스 운영플랫폼에서 설정은 관리자 전용 메뉴입니다. 설정에서는 그룹에 소속되지 않은 서비스를 전부 조회 또는 검색하는 서비스탭과 서버를 추가하여 그룹에 할당하는 서버탭, 사용자를 관리하는 사용자 탭으로 구성되어 있습니다. 서비스 운영플랫폼을 시작하면 어드민 계정이 자동생성됩니다. 아이디는 admin@example.com, 비밀번호는 admin 입니다.

## 서비스탭

설정 메뉴를 클릭하게 되면 첫페이지는 서비스탭을 보여주고 있습니다. 서비스탭에는 사용자들이 만든 모든 서비스를 리스트로 보여지게 됩니다. 검색 키워드를 입력 후 검색 또는 엔터를 입력하였을때 그룹명, 서비스명, 서버, 타입 에 일치하는 키워드가 있는 서비스만 필터링됩니다. 그룹, 서비스, 서버 각각의 이름을 클릭하면 해당 상세 화면으로 이동하게 됩니다.

![/images/2020-12-04-service-management-3/Untitled.png](/images/2020-12-04-service-management-3/Untitled.png)

## 서버탭

서버탭 초기화면은 등록되어 있는 서버리스트가 보여집니다. 서비스 미할당 서버를 체크하기 되면 서비스에 할당되지 않은 서버만 필터랑하여 확인할 수 있습니다.

![/images/2020-12-04-service-management-3/Untitled%201.png](/images/2020-12-04-service-management-3/Untitled%201.png)

초기화면에서 서버추가 버튼을 클릭하게 되면 모달이 보여지게 됩니다. 서버추가 정보를 입력한 뒤 연결테스트를 하여 접속 가능한지 확인 후 추가 버튼을 클릭하면 됩니다. 

이름: 서버의 명칭을 입력합니다.

할당그룹: 현재 생성된 그룹을 선택할 경우 서버 추가와 함께 그룹에 할당까지 즉시됩니다.

아이피: 원격 접속할 대상 아이피를 입력합니다.

포트: 원격 대상 ssh 포트를 입력합니다.

도커포트: 컨테이너 서비스를 사용할 경우 도커 포트를 입력합니다. 

계정: 원격 대상 ssh 접속할 사용자명을 입력합니다.

비밀번호: 원격 대상 ssh 접속할 사용자의 비밀번호를 입력합니다.

![/images/2020-12-04-service-management-3/Untitled%202.png](/images/2020-12-04-service-management-3/Untitled%202.png)

연결테스트를 클릭하였을때 정상적으로 연결이 되면 아래 처럼 초록 알림이 표시 됩니다.

![/images/2020-12-04-service-management-3/Untitled%203.png](/images/2020-12-04-service-management-3/Untitled%203.png)

서버 목록에서 방금 추가한 서버내용을 확인 할 수 있습니다.

![/images/2020-12-04-service-management-3/Untitled%204.png](/images/2020-12-04-service-management-3/Untitled%204.png)

서버목록에서 서버를 클릭하게 되면 서버의 상세 화면으로 이동하게 됩니다. 서버의 정보를 표시하고 있고, 시스템 탭에서 간편 명령어 버튼을 클릭하여 서버의 정보를 조회할 수 있습니다.

![/images/2020-12-04-service-management-3/Untitled%205.png](/images/2020-12-04-service-management-3/Untitled%205.png)

터미널 열기 버튼을 클릭하게 되면 WebUI 기반 터미널을 실행할 수 있습니다.

![/images/2020-12-04-service-management-3/Untitled%206.png](/images/2020-12-04-service-management-3/Untitled%206.png)

그룹탭을 클릭하게 되면 현재 할당된 그룹을 조회 할 수 있습니다.  그룹추가 입력칸에 그룹을 검색 후 추가 버튼을 클릭하게 되면 그룹에 서버를 할당할 수 있습니다. 목록에서 제거 버튼을 클릭하게 되면 해당그룹에서 서버를 제거 할 수 있습니다.

![/images/2020-12-04-service-management-3/Untitled%207.png](/images/2020-12-04-service-management-3/Untitled%207.png)

서버 수정을 클릭하여 서버의 정보를 수정할 수 있습니다. 

![/images/2020-12-04-service-management-3/Untitled%208.png](/images/2020-12-04-service-management-3/Untitled%208.png)

비밀번호 변경을 클릭하게됨 모달이 표시되고, 변경할 비밀번호를 입력 후 변경 버튼을 클릭합니다.

![/images/2020-12-04-service-management-3/Untitled%209.png](/images/2020-12-04-service-management-3/Untitled%209.png)

## 사용자탭

사용자탭을 클릭하게 되면 모든 사용자 목록을 표시하게 됩니다. 사용자를 검색할 수 있고, 일반 사용자를 어드민 사용자로 변경이 가능합니다. 초기화 버튼을 클릭하여 비밀번호를 초기화 할 수 있습니다. 삭제 버튼을 클릭하게 되면 사용자를 강제로 제거하게 됩니다.

![/images/2020-12-04-service-management-3/Untitled%2010.png](/images/2020-12-04-service-management-3/Untitled%2010.png)

## 정리

서비스 운영플랫폼의 설정을 보았습니다. 설정은 관라지만 접근가능 모든 그룹, 서버, 사용자를 관리 할 수 있습니다.

요약

- 서비스: 사용자가 생성한 모든 서비스를 조회하는 기능을 제공
- 서버: 서버추가, 서버를 그룹에 할당하는 기능을 제공
- 사용자: 사용자를 관리하는 기능을 제공
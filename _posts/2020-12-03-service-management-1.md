---
layout: post
title:  "서비스 운영플랫폼 #1 - 소개"
description: "서비스 운영플랫폼을 소개하도록 하겠습니다. 서비스 운영플랫폼은 도커 컴포즈 또는 프로세스를 cli 환경이 아닌 웹UI를 통해 관리할 수 있습니다." 
date:   2020.12.03. 18:00:00
writer: "김준우"  
categories: Mangement
---
## 소개

안녕하세요. 김준우입니다. 서비스 운영플랫폼을 소개하도록 하겠습니다. 서비스 운영플랫폼은 도커 컴포즈 또는 프로세스를 cli 환경이 아닌 웹UI를 통해 관리할 수 있습니다.

## 시스템 구성도

아래 이미지 처럼 서비스 운영플랫폼을 통해 워커서버한테 원격으로 명령어를 요청보내 서비스를 관리합니다. 현재 사용하던 서버에는 별도의 에이전트를 설치하지 않고, 즉시 연결할 수 있습니다.

![/images/2020-12-03-service-management-1/Untitled.png](/images/2020-12-03-service-management-1/Untitled.png)

## 관계도

관리자는 서버를 생성하여 그룹에 할당이 가능하며, 일반 사용자는 소속된 그룹에서 서비스를 사용할 수 있습니다.

![/images/2020-12-03-service-management-1/Untitled%201.png](/images/2020-12-03-service-management-1/Untitled%201.png)

## 필요성

컨테이너와 일반 프로세스가 혼재된 운영시스템을 관리하기 위한 플랫폼입니다.

대규모 컨테이너 클러스터를 위한 쿠버네티스 또는 스웜 기술이 존재하지만, 해당 플랫폼을 도입시 모든 운영패턴을 그 플랫폼으로 바꿔나가야 합니다.

즉, 플랫폼의 도입은 단순히 소프트웨어의 도입을 넘어서, 운영패턴과 정책도 함께 도입하는 것을 의미합니다.

우리는 현재의 운영패턴을 기반으로 도커 컨테이너가 가미된 하이브리드형 관리 플랫폼이 필요한 시점입니다.

기존의 안정적인 운영방식을 기반으로 신규 운영패턴을 가미한 안정성 높은 하이브리드 관리플랫폼을 개발하려 합니다..

## 목차

1. 소개
2. 사용자
3. 설정
4. 그룹
5. 홈

## 기술스택

- [nodejs](https://nodejs.org/ko/)
- [react](https://ko.reactjs.org/)
- [nextjs](https://nextjs.org/)
- [sqlite](https://www.sqlite.org/index.html)
- [sequelize](https://sequelize.org/)

## Github

- [https://github.com/danawalab/service-management](https://github.com/danawalab/service-management)
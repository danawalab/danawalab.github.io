---
layout: post
title: "검색파트의 반복 작업 자동화 과정"
description: "주기적으로 들어오는 반복적인 작업을 자동화한 사례를 소개합니다."
date: 2024.03.08.
writer: "곽명환"
categories: Common
---

## 소개

안녕하세요. 다나와에서 검색 서비스를 개발하고 있는 곽명환입니다.  
오늘은 주기적으로 들어오는 요청의 자동화에 대해서 소개해보도록 하겠습니다.

## 문제점

검색 서비스 담당팀은 정기적으로 변화하는 처리 키워드와 비 처리 키워드를 업데이트하며, 이를 검색 팀에 처리 요청합니다. 처음에는 관련 위키를 참조하며 대응했으나, 몇 가지 문제점을 파악했습니다.

- 첫째, 정기적인 요청에도 불구하고, 모든 프로세스를 기억하는 것이 아니라서, 매번 관련 문서를 찾아보며 과정을 확인하고 결과를 검증해야 했습니다.

- 둘째, 반복되는 작업에 적잖은 시간을 소비해야 했습니다.

#### 소프트웨어 개발에서 자동화는 개발자들이 반드시 주목해야 할 중요한 요소입니다. 
#### 자동화는 생산성과 일관성을 향상시켜 개발자가 더 효율적으로 작업을 수행하고 더 많은 업무를 처리할 수 있게 합니다.

기존 처리 과정을 살펴보면, 다음과 같은 단계를 거쳤습니다.

## 기존 처리 방식

### 엑셀 파일 수신
처리할 키워드가 포함 된 엑셀 파일을 다운로드 받습니다.
### SSH 프로토콜을 통한 검색 서버 접속
SSH를 통해 검색 서버에 접속한 뒤, 파일을 다운로드 받아 키워드 리스트를 업데이트합니다.
### 처리 작업
업데이트된 키워드 리스트를 프로그램을 실행하여 추가적인 작업을 수행합니다.
### SSH를 통한 파일 업로드
검색 서버에 접속하고, 파일을 업로드합니다.
### API 호출
서버의 특정 API를 호출하여 최신화된 파일을 적용합니다.

---

이 과정은 비교적 간단하지만 여러 대의 검색 서버가 운영되고 있어 시간이 소요되며, 무엇보다 이러한 과정들이 번거롭게 느껴집니다.

## 개발도구

자동화를 위해 저는 코틀린 + 스프링 부트로 서버를 개발했는데요.  
개발 도구는 다음과 같습니다.
- Kotlin 1.8.22, Spring Boot 3.1.5
- Docker, Gitlab ci

## 처리 과정

해당 시스템의 처리 과정은 아래 이미지에 자세히 설명되어 있습니다.

![image1](/images/2024-03-08-adult-keyword-automation-image/image1.png) 

처리 과정 중 주요 문제점은 SSH 접근 부분이었습니다. 로컬 서버에서 검색 API 서버를 호스팅하는 SSH 서버에 접근할 수 없었던 문제였습니다. 

SSH 접근 문제를 해결하기 위한 방법을 모색 중, SSH 터널링이라는 방식을 알게되어 조사 후 테스트 해봤습니다.

![ssh-tunneling1](/images/2024-03-08-adult-keyword-automation-image/ssh-tunneling.png) 
[출처](https://www.tunnelsup.com/how-to-create-ssh-tunnels/)

## 예제 코드

JSch 라이브러리를 사용해 SSH 터널링을 설정하는 예제 코드입니다.

### use함수 예제(gpt 활용)

![use1](/images/2024-03-08-adult-keyword-automation-image/use-example1.png) 

![use2](/images/2024-03-08-adult-keyword-automation-image/use-example2.png) 

Jsch 라이브러리에서는 자원 해제를 지원해주는 use 함수가 구현되어있지 않습니다.

코틀린의 확장 함수를 직접 구현하여 try-finally 블록을 간결하게 처리함으로써 코드의 가독성을 향상시켰습니다

![image2](/images/2024-03-08-adult-keyword-automation-image/image2.png) 

![image3](/images/2024-03-08-adult-keyword-automation-image/image3.png) 

SSH 연결 후 수행할 작업(예: 파일 업로드, 백업 파일 활성화 등)은 전략 패턴을 적용하여 구현함으로써, 작업 선택의 유연성을 높였습니다.  
또한 SSH 커넥션 설정과 이후 작업을 명확히 분리하여 구조를 개선했습니다.

![image4](/images/2024-03-08-adult-keyword-automation-image/image4.png)

## 결론

이러한 절차를 통해 기존 작업을 자동화할 수 있으며, 상황에 맞는 패턴을 활용해 코드의 확장성 또한 강화했습니다.

감사합니다.
---
layout: post
title:  "디서치 관리도구 6월 업데이트"
description: DSearch 콘솔과 서버에 기능이 추가 되었습니다.
date:   2021.06.22.
writer: "선지호"
categories: Dsearch
---

## 개요
DSearch 콘솔과 서버에 기능이 추가 되었습니다.

- JDBC 수정창에서 연결테스트를 진행할 수 있습니다.
- 멀티 스케쥴 크론 설정을 할 수 있습니다.
- 사전-개요 탭 전체 선택 버튼이 추가되었습니다.

## 변경사항

### 1. JDBC 수정창 연결테스트

JDBC 수정창에 연결테스트 버튼이 추가 되었습니다.

![/images/2021-06-21-Dsearch-Console-Changes/1.png](/images/2021-06-21-Dsearch-Console-Changes/1.png){:.border}  

비밀번호를 정상적으로 입력하고 연결테스트 버튼을 클릭하면 아래와 같이 토스트 메시지가 나옵니다.

![/images/2021-06-21-Dsearch-Console-Changes/2.png](/images/2021-06-21-Dsearch-Console-Changes/2.png){:.border}  

하지만, 비밀번호를 틀리게 입력하고 버튼을 클릭하면, 아래와 같은 토스트 메시지가 나옵니다.

![/images/2021-06-21-Dsearch-Console-Changes/3.png](/images/2021-06-21-Dsearch-Console-Changes/3.png){:.border}  

### 2. 멀티 스케쥴 크론 설정

분 단위의 멀티 스케쥴이 필요한 경우가 있었습니다.

하지만 기존의 크론 설정으로는 분 단위의 상세한 스케쥴 설정은 할 수 없었기에, 멀티 스케쥴 기능을 최대 5개까지의 설정이 가능하도록 추가하였습니다.

![/images/2021-06-21-Dsearch-Console-Changes/4.png](/images/2021-06-21-Dsearch-Console-Changes/4.png){:.border}  

![/images/2021-06-21-Dsearch-Console-Changes/5.png](/images/2021-06-21-Dsearch-Console-Changes/5.png){:.border}  

![/images/2021-06-21-Dsearch-Console-Changes/8.png](/images/2021-06-21-Dsearch-Console-Changes/8.png){:.border}  

### 3. 사전-개요 탭 전체 선택 버튼 추가

사전-개요탭에서 사전 기능이 전체적으로 업데이트를 진행해야 하는 경우가 있었습니다.
하지만 일일히 버튼을 추가 하여 진행을 해야 했기에 사전이 많은 경우 시간이 오래 걸렸습니다.

따라서 전체 선택 버튼을 추가 하였습니다.

![/images/2021-06-21-Dsearch-Console-Changes/6.png](/images/2021-06-21-Dsearch-Console-Changes/6.png){:.border}  

![/images/2021-06-21-Dsearch-Console-Changes/7.png](/images/2021-06-21-Dsearch-Console-Changes/7.png){:.border}  
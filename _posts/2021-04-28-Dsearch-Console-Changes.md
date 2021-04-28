---
layout: post
title:  "디서치 관리도구 변경사항 - 4월 업데이트"
description: DSearch 콘솔과 서버에 기능이 추가 되었습니다.
date:   2021.04.28.
writer: "선지호"
categories: Dsearch
---

## 개요
DSearch 콘솔과 서버에 기능이 추가 되었습니다.

- 사전 초기화 기능이 추가 되었습니다.
- 사전파일로 업로드 기능이 추가 되었습니다.
- 클러스터 점검 시작 시 스케줄이 정지 되고, 클러스터 점검 완료 시 자동으로 스케줄이 등록 됩니다.
- 관리도구에서 사용되는 데이터에 대한 백업과 복구가 가능해 졌습니다.

## 변경사항

### 1. 사전 초기화 기능

사전 탭 - 수정 - 더보기 클릭 시 나오는 화면입니다.
![/images/2021-04-28-Dsearch-Console-Changes/reset1.png](/images/2021-04-28-Dsearch-Console-Changes/reset1.png)

삭제 하기 전 사전의 상태 입니다.
![/images/2021-04-28-Dsearch-Console-Changes/reset2.png](/images/2021-04-28-Dsearch-Console-Changes/reset2.png)

사전 초기화 버튼을 클릭 하게 되면 경고창이 나옵니다.
![/images/2021-04-28-Dsearch-Console-Changes/reset3.png](/images/2021-04-28-Dsearch-Console-Changes/reset3.png)

초기화 버튼을 클릭하면, 모두 삭제 된 것을 확인 할 수 있습니다.
![/images/2021-04-28-Dsearch-Console-Changes/reset4.png](/images/2021-04-28-Dsearch-Console-Changes/reset4.png)

### 2. 사전 파일로 업로드 기능

사전 탭 - 수정 버튼 - 더보기 - 사전파일 업로드 클릭 시 나오는 화면 입니다.
덮어쓰기와 이어쓰기 중 선택 할 수 있습니다.
![/images/2021-04-28-Dsearch-Console-Changes/restore1.png](/images/2021-04-28-Dsearch-Console-Changes/restore1.png)

삭제 한 내용을 이전에 다운로드로 저장해 두었다면, 정상적으로 사전파일이 복구 된 것을 확인 할 수 있습니다.
![/images/2021-04-28-Dsearch-Console-Changes/restore2.png](/images/2021-04-28-Dsearch-Console-Changes/restore2.png)

### 3. 클러스터 점검 시 기능 추가

클러스터 점검 버튼 클릭 시 나오는 화면입니다. 
![/images/2021-04-28-Dsearch-Console-Changes/check1.png](/images/2021-04-28-Dsearch-Console-Changes/check1.png)

점검을 클릭하게 되면 아래와 같은 화면이 나오게 됩니다.
![/images/2021-04-28-Dsearch-Console-Changes/check2.png](/images/2021-04-28-Dsearch-Console-Changes/check2.png)

점검 시작을 하게 되면, 아래와 같이 1분 주기로 설정되어 있는 컬렉션 색인이 중지가 됩니다.
![/images/2021-04-28-Dsearch-Console-Changes/check4.png](/images/2021-04-28-Dsearch-Console-Changes/check4.png)

실제로 중지가 되어 색인이 되지 않은 화면 입니다.
![/images/2021-04-28-Dsearch-Console-Changes/check3.png](/images/2021-04-28-Dsearch-Console-Changes/check3.png)


### 4. 관리도구 데이터 백업 및 복구

관리도구 데이터를 파일로 백업 및 파일 업로드를 할 수 있는 화면입니다.
![/images/2021-04-28-Dsearch-Console-Changes/migration1.png](/images/2021-04-28-Dsearch-Console-Changes/migration1.png)

다운로드 버튼을 클릭하면, 아래와 같은 화면이 나옵니다.
체크박스로 원하는 데이터만 다운받을 수 있도록 선택 할 수 있습니다.
![/images/2021-04-28-Dsearch-Console-Changes/migration2.png](/images/2021-04-28-Dsearch-Console-Changes/migration2.png)

정상적으로 다운로드가 된다면, 아래와 같은 화면이 나옵니다.
다운로드에 대한 개략적인 내용이 표시됩니다.
![/images/2021-04-28-Dsearch-Console-Changes/migration3.png](/images/2021-04-28-Dsearch-Console-Changes/migration3.png)

다운로드 받은 데이터로 업로드를 진행하기 위한 화면 입니다.
![/images/2021-04-28-Dsearch-Console-Changes/migration4.png](/images/2021-04-28-Dsearch-Console-Changes/migration4.png)

정상적으로 업로드가 된다면, 아래와 같은 화면이 나옵니다.
업로드에 대한 개략적인 내용이 표시됩니다.
![/images/2021-04-28-Dsearch-Console-Changes/migration5.png](/images/2021-04-28-Dsearch-Console-Changes/migration5.png)

만약 정상적인 데이터가 아니라면, 아래와 같은 내용이 표시 됩니다.
![/images/2021-04-28-Dsearch-Console-Changes/migration6.png](/images/2021-04-28-Dsearch-Console-Changes/migration6.png)

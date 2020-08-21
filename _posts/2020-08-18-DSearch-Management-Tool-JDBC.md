---
layout: post
title:  "디서치 관리도구 사용법 #9 - JDBC"
description: "이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 JDBC 메뉴에 대해 소개하는 시간을 가지도록 하겠습니다." 
date:   2020.08.18. 14:00:00
writer: "김준우"  
categories: Elastic 
---
## 소개

이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 JDBC 메뉴에 대해 소개하는 시간을 가지도록 하겠습니다.

## JDBC 메뉴란?

JDBC메뉴에서는 데이터베이스 접속 정보를 관리하게됩니다.  접속정보를 통해 데이터베이스에 연결을 테스트 기능을 통해 접속정보 오타를 줄일 수 있습니다.

## JDBC 메뉴 사용법

### 처음 화면

처음 메뉴를 눌렀을 때 나오는 화면입니다.

![/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled.png](/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled.png)

### JDBC 추가 화면

처음화면에서 JDBC 추가 버튼을 클릭하게 되면 팝업이 노출되고, JDBC 입력을 할 수 있게 됩니다.

현재(2020.08.18 기준) 제공하고 있는 DB드라이버는 Altibase, Oracle, MySql를 지원하고 있습니다. 

![/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%201.png](/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%201.png)

![/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%201.png](/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%201-1.png)

### JDBC 연결테스트

입력란에 JDBC 정보를 입력 후 연결테스트 버튼을 누르게 되면 아래 연결 결과를 표시하게 됩니다.  

이미지에서는 연결테스트 실패를 하였지만, 정상적으로 연결이 되었을 경우, 연결테스트 성공 메시지가 표시 됩니다. 추가 버튼을 누르게 되면 정보가 저장되고, 팝업창이 사라지게 됩니다.

![/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%202.png](/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%202.png)

또한, 페이지에 접속 했을 때 등록한 JDBC 데이터베이스 접속 정보를 통해 연결테스트를 할 수 있게 지원합니다.

![/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%202.png](/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%202-1.png)

### JDBC 수정/삭제

JDBC 정보를 추가한뒤 수정이 필요하면 각 라인에 있는 수정 버튼 클릭하게 되면 수정 팝업이 노출되게 됩니다. 

수정: 입력 항목에 데이터를 수정한뒤 저장 버튼을 클릭하면 변경된 내용으로 정보가 수정됩니다.

삭제: 삭제 버튼을 클릭하게 되면 해당 JDBC 정보는 영구적으로 삭제가 됩니다.

![/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%203.png](/images/2020-08-18-DSearch-Management-Tool-JDBC/Untitled%203.png)

## 정리

다나와에서 개발한 디서치 관리도구의 JDBC 메뉴를 소개하였습니다. 대부분 테이터베이스를 이용하기 때문에 접속 정보를 매번 입력하는 불편함을 해소할 수 있는 메뉴입니다. 

디서치를 한번도 안 써본 개발자의 입장이 되어 최대한 자세하게 작성해 보았습니다.

이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
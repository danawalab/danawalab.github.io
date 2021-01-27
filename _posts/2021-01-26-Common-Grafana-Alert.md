---
layout: post
title:  "그라파나에서 텔레그램으로 알람받기"
description: 그라파나에 구성한 대시보드 패널에 알람 규칙을 추가하여 텔레그램으로 알람 받기"
date:   2021.01.26.
writer: "최현복"
categories: Common
---

## 개요

- 그라파나에 구성한 대시보드 패널에 알람 규칙을 추가하여 텔레그램으로 알람을 받아서 위험 감지에 사용 할 수 있습니다.



## 구성 방법

### 1. 알람 채널 추가
- Alerting > Notification channels 메뉴를 선택합니다.

  ![/images/2021-01-26-Common-Grafana-Alert/alert_01.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_01.PNG) 


- New channel 을 클릭합니다.

  ![/images/2021-01-26-Common-Grafana-Alert/alert_02.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_02.PNG)  


- Type 을 Telegram 으로 선택합니다.

  ![/images/2021-01-26-Common-Grafana-Alert/alert_03.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_03.PNG)  

  ![/images/2021-01-26-Common-Grafana-Alert/alert_04.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_04.PNG) 
  - Name, Default : 모든 알람 받기
  - Include image : 이미지 포함 여부
  - Disable Resolve Message : 해결 알람 받지 않기
  - Send reminders : 알림 상기시키기
  - Telegram API settings (BOT API Token, Chat ID) : 텔레그램에 봇을 추가하여 해당 토큰과 사용자의 챗 ID를 기입
    
    설정하고 Save 버튼을 눌러서 저장합니다.


- 추가된 Channel 입니다.

  ![/images/2021-01-26-Common-Grafana-Alert/alert_05.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_05.PNG) 





### 2. 대시보드 패널에 알람 추가

- 초당 요청량과 평균 응답속도 패널에 특정 수치로 알람을 설정 해보겠습니다.
  
  ![/images/2021-01-26-Common-Grafana-Alert/alert_06.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_06.PNG) 



- 추가 하고싶은 패널의 수정 버튼을 누릅니다.

  하단의 Alert 설정에서 Create Alert 버튼을 누릅니다.
  
  ![/images/2021-01-26-Common-Grafana-Alert/alert_07.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_07.PNG)



- 그래프 오른쪽 하트를 위 아래로 드래그하여 알람수치를 조정 할 수 있습니다.
  
  ![/images/2021-01-26-Common-Grafana-Alert/alert_08.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_08.PNG)



- 알람 이름과 규칙을 정합니다.
  
  Evaluate every 10s For 30s

  30초 동안 10초 마다 체크 

- Conditions은 여러개를 추가 할 수 있습니다.

  WHEN avg () OF query (A, 10s, now) IS ABOVE 400

  패널에 추가된 쿼리 A 수치가 10초 전부터 현재까지 평균 값이 400을 넘는 상태

  ![/images/2021-01-26-Common-Grafana-Alert/alert_09.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_09.PNG)



- 평균 응답속도에 대해서도 알람을 설정하겠습니다.
 
  Evaluate every 10s For 0
  
  10초 마다 체크 하도록 설정했습니다.

  WHEN max () OF query (B, 10s, now) IS ABOVE 0.03

  OR max () OF query (C, 10s, now) IS ABOVE 0.03

  쿼리 B, C 중에서 10초 전부터 현재까지 최대값이 0.03초를 넘는 상태에 대해서 알람 설정을 했습니다.

  ![/images/2021-01-26-Common-Grafana-Alert/alert_10.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_10.PNG)

  Conditions 에는 다양한 값으로 조건을 설정 할 수 있습니다. 

  ![/images/2021-01-26-Common-Grafana-Alert/alert_10_1.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_10_1.PNG)





### 3. 알람 확인 및 활용

- 알림 리스트를 대시보드에 패널로 추가할 수 있습니다. 

  ![/images/2021-01-26-Common-Grafana-Alert/alert_11.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_11.PNG)



- 대시보드에 알람 및 리스트가 추가된 모습입니다.

  ![/images/2021-01-26-Common-Grafana-Alert/alert_13.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_13.PNG)


- Telegram 알람 확인

  앞서 Notification Channel 에서 Include image 를 설정하였기 때문에 이미지와 함께 알람이 오게 됩니다.
  
  초당 요청량은 'Evaluate every 10s For 30s' 30초 동안 10초 마다 확인 하도록 설정 하였기 때문에 

  PENDING(보류중) 알람이 오지않고 보류 상태가 됩니다.

  평균 응답 속도에는 'Evaluate every 10s For 0' 10초 마다 상태를 확인해서 보내도록 설정하여 
  
  보류하지 않고 바로 알람이 오게 됩니다.

  초록색 : OK
  
  주황색 : PENDING

  빨간색 : ALERTING

  ![/images/2021-01-26-Common-Grafana-Alert/alert_14.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_14.PNG)



- 알람 설정에서 State history 버튼을 눌러서 
  상태 히스토리를 확인 할 수 있습니다.
  ![/images/2021-01-26-Common-Grafana-Alert/alert_15.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_15.PNG)



- Alerting 메뉴에서 Alert Rules 을 선택하면
  알람 설정 수정으로 바로가거나 시작 및 일시중지 제어 할 수 있습니다.
  ![/images/2021-01-26-Common-Grafana-Alert/alert_16.PNG](/images/2021-01-26-Common-Grafana-Alert/alert_16.PNG)





## 결론

그라파나를 통해서 모니터링에 사용하고 있었습니다.

추가적으로 알람 설정 기능을 통해서 특이사항이 발생할 경우

담당자에게 보다 신속하게 전달하여 위험한 상황을 미리 대비하여 안정적인 서비스를 운영할 수 있습니다.



## 참고 자료
- https://grafana.com/docs/grafana/latest/alerting/


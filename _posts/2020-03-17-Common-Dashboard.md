---
layout: post
title:  "그라파나를 통한 모니터링 대시보드 구성"
description: 프로메테우스가 수집한 노드 익스포터의 매트릭 정보와 PROMQL을 통해 그라파나 대시보드 구성하기"
date:   2020.03.17.
writer: "하선호"
categories: Common
---

## 그라파나?

- 그라파나는 데이터를 시각화하여 분석 및 모니터링을 용이하게 해주는 오픈소스 분석 플랫폼입니다. 여러 데이터 소스를 연동하여 사용 할 수 있으며 시각화 된 데이터들을 대시보드로 만들 수 있습니다.

  >[LIVE DEMO](https://play.grafana.org/d/000000012/grafana-play-home?orgId=1)



- 앞서 설치한 그라파나와 프로메테우스의 메트릭 정보를 조회하여 시각화 하는 방법에 대해 알아보겠습니다.


## 구성 방법

### 1. 데이터 소스 생성
- Create a data source 메뉴를 선택합니다.
![/images/2020-03-17-Common-Dashborad/grafana_1.PNG](/images/2020-03-17-Common-Dashboard/grafana_1.PNG) 

- 데이터 소스로 프로메테우스를 선택합니다.
![/images/2020-03-17-Common-Dashborad/grafana_2.PNG](/images/2020-03-17-Common-Dashboard/grafana_2.PNG) 

- 프로메테우스 서버 정보를 입력한 후 하단의 Save&Test를 클릭합니다.
  - 연결테스트 후 이상이 없다면 저장됩니다.

![/images/2020-03-17-Common-Dashborad/grafana_3.PNG](/images/2020-03-17-Common-Dashboard/grafana_3.PNG) 

### 2. 대시 보드 생성

- 홈으로 돌아와 Build a dashboard 메뉴를 선택합니다.

![/images/2020-03-17-Common-Dashborad/grafana_4.PNG](/images/2020-03-17-Common-Dashboard/grafana_4.PNG) 

- 패널 선택장이 나옵니다. 대시보드는 이 패널들을 구성하고 배치하면 됩니다.
  - 메트릭 조회를 위해 Add Query를 선택합니다

![/images/2020-03-17-Common-Dashborad/grafana_5PNG](/images/2020-03-17-Common-Dashboard/grafana_5.PNG) 

- Query 선택창에서 앞에 생성한 Data Source를 불러옵니다.
- Metric에선 노드익스포터에서 수집한 항목을 선택합니다.
  
![/images/2020-03-17-Common-Dashborad/grafana_7PNG](/images/2020-03-17-Common-Dashboard/grafana_7.PNG) 

```
잠시 돌아와서 위에 선택한 메트릭 항목은 아래와 같은 구조를 가지고 있습니다.
따라서 Metric name으로만 조회시 프로메테우스 서버가 수집한 같은 이름 항목 전부를 조회 합니다. Label name으로 구분하여 조회 할 수 있습니다.
```

- Prometheus Metric

![/images/2020-03-17-Common-Dashborad/metrics.PNG](/images/2020-03-17-Common-Dashboard/metrics.PNG) 

- 이처럼 프로메테우스 서버에서도 메트릭을 조회할 수 있습니다.
- node_cpu_seconds_total{job="kube2",mode="system"}
  - ex) kube2 host에서 system 영역 cpu 사용률

![/images/2020-03-17-Common-Dashborad/grafana_9.PNG](/images/2020-03-17-Common-Dashboard/grafana_9.PNG) 
   
- 다시 돌아와서 생성한 패널 쿼리에 적용해보겠습니다.

![/images/2020-03-17-Common-Dashborad/grafana_10.PNG](/images/2020-03-17-Common-Dashboard/grafana_10.PNG) 

- node_cpu_seconds_total는 계속 누적이 되고 있는 값이기 때문에 rate함수를 사용합니다.
- rate(node_cpu_seconds_total{job="kube2",mode="system"}[10m])
  - 10분동안 CPU 사용율 변화수치를 초당으로 변환

![/images/2020-03-17-Common-Dashborad/grafana_11.PNG](/images/2020-03-17-Common-Dashboard/grafana_11.PNG)


- CPU 코어의 평균을 구하여 해당 서버의 CPU 평균 사용률을 표시합니다.
-avg(rate(node_cpu_seconds_total{job="kube2",mode="system"}[10m])) by (job)

![/images/2020-03-17-Common-Dashborad/grafana_12.PNG](/images/2020-03-17-Common-Dashboard/grafana_12.PNG)

- 설정에서 그래프 종류를 선택할 수 있습니다.

![/images/2020-03-17-Common-Dashborad/grafana_6.PNG](/images/2020-03-17-Common-Dashboard/grafana_6.PNG)

이처럼 자신의 필요한 메트릭 정보를 집계하여 패널로 만든 후 대시보드를 구성하면 됩니다.

## 외부 대시보드 포맷 사용
- 매트릭 정보를 직접 집계하여 사용하거나 필요한 데이터를 정의하는게 나름 번거로운 작업이라 그라파나 홈페이지에 다른 여러 사용자들이 구성해놓은 대시보드 포맷을 사용하면 더 쉽게 대시보드를구현할 수 있습니다.

[https://grafana.com/grafana/dashboards](https://grafana.com/grafana/dashboards)

![/images/2020-03-17-Common-Dashborad/grafana_13.PNG](/images/2020-03-17-Common-Dashboard/grafana_13.PNG)

- import 화면에서 다운받은 JSON 파일을 upload하거나 COPY ID를 입력하여 적용합니다.

![/images/2020-03-17-Common-Dashborad/grafana_14.PNG](/images/2020-03-17-Common-Dashboard/grafana_14.PNG)


## 결론
프로메테우스, 그라파나를 통해 비교적 쉽게 모니터링을 위한 프로세스들을 만들 수 있었습니다.
여러 익스포터를 통해 필요한 매트릭 수집을 확장할 수 있다는게 큰 장점인 것 같으며
PULL 방식이라 부하에 따른 장애나 성능감소를 걱정하지 않아도 된다고 합니다.
그라나파를 통해 매트릭 데이터를 보기 쉽게 만들 수 있었으며 다양한 데이터 소스를 지원하는 만큼 여러 분야에서 활용하면 좋을 것 같습니다.

## 참고 자료

- https://devthomas.tistory.com/15
- https://grafana.com/
- 프로메테우스 - 오픈소스 모니터링 시스템 (출판사 : 책만 / 2019-10)
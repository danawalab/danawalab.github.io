---
layout: post
title: "Elastic Cloud를 활용한 모니터링 및 이상탐지"
description: Elastic Cloud를 활용한 모니터링 및 이상탐지
date:   2021.08.30. 
writer: "선지호"
categories: Elastic 
---

## 소개 : Elasticseach Cloud

클라우드 서비스인 AWS, Google Cloud 및 Azure에서 호스트형 Elasticsearch와 Kibana, 각종 Elasticsearch 관련 애플리케이션을 한꺼번에 관리 및 배포할 수 있게 하는 서비스 입니다

## 사용법

순서는 이렇습니다.

1. 새 이메일 생성(Optional)
2. 해당 이메일로 Elasticsearch Cloud에 가입 (https://cloud.elastic.co/login)
3. deployment 생성
4. Machine learning 셋팅
5. 모니터링 애플리케이션(APM, filebeat, metricbeat, ...etc) 구성 및 셋팅
6. 이상 탐지 기능 셋팅


## 1. 새 이메일 생성

기존에 사용하던 이메일을 사용하셔도 무방하나, 저는 새로 이메일을 생성하였습니다.

## 2. 해당 이메일로 Elasticsearch Cloud에 가입

[Elastic Cloud](https://cloud.elastic.co/login, "Elastic Cloud")


위의 링크로 접속하여 Elasticsearch Cloud에 가입을 합니다.

## 3. deployment 생성

Create Deployment를 눌러 Deployment의 명칭을 입력해 줍니다.
조금더 상세히 옵션을 조절하고 싶다면 Edit Settings를 통해 옵션을 조절해 줍니다.
![/images/2021-08-30-es-monitoring-and-alert/1.png](/images/2021-08-30-es-monitoring-and-alert/1.png)

![/images/2021-08-30-es-monitoring-and-alert/2.png](/images/2021-08-30-es-monitoring-and-alert/2.png)


생성된 계정과 비밀번호를 저장하고, continue를 눌러 계속 진행해 줍니다. 
약 5~10분 정도 걸립니다.
![/images/2021-08-30-es-monitoring-and-alert/3.png](/images/2021-08-30-es-monitoring-and-alert/3.png)

![/images/2021-08-30-es-monitoring-and-alert/4.png](/images/2021-08-30-es-monitoring-and-alert/4.png)

![/images/2021-08-30-es-monitoring-and-alert/5.png](/images/2021-08-30-es-monitoring-and-alert/5.png)

아래 창이 뜬다면, 사용할 Elastic Cloud의 노드들이 모두 준비되었습니다
![/images/2021-08-30-es-monitoring-and-alert/6.png](/images/2021-08-30-es-monitoring-and-alert/6.png)

## 4. Machine learning 셋팅

Elastic Cloud에서 머신러닝을 사용하기 위해서는 설정이 필요합니다.

Elastic Cloud deployments에 접속하여 Edit 버튼을 클릭해 줍니다.
![/images/2021-08-30-es-monitoring-and-alert/7.png](/images/2021-08-30-es-monitoring-and-alert/7.png)

![/images/2021-08-30-es-monitoring-and-alert/8.png](/images/2021-08-30-es-monitoring-and-alert/8.png)


그리고, 스크롤을 조금 내리면 나오는 Machine Learning nodes 영역이 있는데, 여기서 사용할 노드를 선택하여 줍니다.
그리고 save를 누르고 약 3~5분정도 기다리면 Machine Learning을 위한 셋팅이 완료 됩니다.
![/images/2021-08-30-es-monitoring-and-alert/9.png](/images/2021-08-30-es-monitoring-and-alert/9.png)

![/images/2021-08-30-es-monitoring-and-alert/10.png](/images/2021-08-30-es-monitoring-and-alert/10.png)

![/images/2021-08-30-es-monitoring-and-alert/11.png](/images/2021-08-30-es-monitoring-and-alert/11.png)

## 5. 모니터링 애플리케이션 구성 및 셋팅

이번 포스팅에서는 모니터링 애플리케이션으로 사용되고 있는 Metricbeat를 사용하여 구성해보겠습니다.

저는 사용하는 서버가 리눅스 Centos이기 때문에 리눅스 버전으로 사용해보겠습니다.

먼저, 최신 버전인 7.14.0 버전을 사용하여 해당 서버에 받아 압축을 해제합니다.
시스템 메트릭만 가져갈 것이기 때문에 따로 modules는 활성화 하지 않겠습니다.

```jsx
$ curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.14.0-linux-x86_64.tar.gz
$ tar xzvf metricbeat-7.14.0-linux-x86_64.tar.gz
$ mv metricbeat-7.14.0-linux-x86_64/ metricbeat
$ cd metricbeat
```

그리고, 압축을 푼 폴더 안의 yml 파일을 수정합니다.

- metricbeat.yml
```yml
metricbeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false

setup.template.settings:
  index.number_of_shards: 1
  index.codec: best_compression

setup.kibana.host: "부여받은 kibana endpoint"
setup.kibana.protocol: "https"
setup.dashboards.enabled: true

dashboards.enabled: true
output.elasticsearch:
  username: "elastic"
  password: "부여받은 패스워드"
  hosts: ["부여받은 elasticsearch endpoint"]
  protocol: "https"
```

- metricbeat.yml
```jsx
metricbeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false

setup.template.settings:
  index.number_of_shards: 1
  index.codec: best_compression

setup.kibana.host: "부여받은 kibana endpoint"
setup.kibana.protocol: "https"
setup.dashboards.enabled: true

dashboards.enabled: true
output.elasticsearch:
  username: "elastic"
  password: "부여받은 패스워드"
  hosts: ["부여받은 elasticsearch endpoint"]
  protocol: "https"
```

- modules.d/system.yml
```jsx
- module: system
  period: 10s
  metricsets:
    - cpu
    - load
    - memory
    - network
    - process
    - process_summary
    - socket_summary
    #- entropy
    #- core
    #- diskio
    #- socket
    #- service
    #- users  
  process.include_top_n:
    by_cpu: 5      # include top 5 processes by CPU
    by_memory: 5   # include top 5 processes by memory
  # Configure the mount point of the host▒~Ys filesystem for use in monitoring a host from within a container
  #system.hostfs: "/hostfs"

- module: system
  period: 1m
  metricsets:
    - filesystem
    - fsstat
  processors:
  - drop_event.when.regexp:
      system.filesystem.mount_point: '^/(sys|cgroup|proc|dev|etc|host|lib|snap|data)($|/)'
- module: system
  period: 15m
  metricsets:
    - uptime
```


마지막으로, 해당 Metricbeat를 백그라운드로 실행해 줍니다.

```jsx
$ nohup ./metricbeat -e  &
$ tail -f nohup.out
```

그리고 아래와 같은 로그가 뜰 때 까지 기다려줍니다.

```jsx 
2021-08-30T11:40:04.121+0900    INFO    [publisher_pipeline_output]     pipeline/output.go:143  Connecting to backoff(elasticsearch(https://your-elastic-endpoint.elastic-cloud.com:9243))
```

이제, 메트릭이 정상적으로 수집되는지 kibana를 접속하고, Dev Tools로 이동하여 아래 쿼리를 입력해 정상적으로 수집 되는지 확인하면 됩니다.

```jsx
GET metricbeat-7.14.0/_search
{
  "track_total_hits": true, 
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ]
}
```

## 6. 이상 탐지 기능 셋팅

### 머신러닝 Job 설정

왼쪽 탭에서 Machine Learning 을 찾아 클릭하여 이동합니다.
![/images/2021-08-30-es-monitoring-and-alert/12.png](/images/2021-08-30-es-monitoring-and-alert/12.png)


이후, Create Job을 통하여 Machine Learning을 할 작업을 만들어줍니다.
![/images/2021-08-30-es-monitoring-and-alert/13.png](/images/2021-08-30-es-monitoring-and-alert/13.png)


저희는 Metricbeat를 사용하였기 때문에 metricbeat-* 인덱스 패턴을 사용할 것입니다.
만약 따로 Saved Search나 index pattern을 등록하시고 싶으시다면 stack management로 이동하여 등록해 주시면 됩니다.
![/images/2021-08-30-es-monitoring-and-alert/14.png](/images/2021-08-30-es-monitoring-and-alert/14.png)


실행할 Job을 생성할 때, 선택을 할 수 있습니다.
직접 구성할 것인지(Single metric, Multi-metric ... etc) 혹은 이미 구성된 설정을 이용할 것인지 선택할 수 있습니다.
이번에는 직접 구성해 보겠습니다.
![/images/2021-08-30-es-monitoring-and-alert/15.png](/images/2021-08-30-es-monitoring-and-alert/15.png)


아래와 같이 수집된 시간을 선택합니다
![/images/2021-08-30-es-monitoring-and-alert/16.png](/images/2021-08-30-es-monitoring-and-alert/16.png)


그리고 해당 인덱스의 필드를 선택하는데, 수집되는 내용을 분석하여 그래프를 그려주기 때문에 한눈에 추이를 볼 수 있습니다.
공식 사이트를 이용하여 수집되는 필드가 무엇이 있는지 확인해 보는 것이 좋습니다.
![/images/2021-08-30-es-monitoring-and-alert/17.png](/images/2021-08-30-es-monitoring-and-alert/17.png)

※ 공식 사이트 링크 : [링크](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-modules.html, "Metricbeat 공식 사이트")

Job의 이름을 입력해 줍니다.
![/images/2021-08-30-es-monitoring-and-alert/18.png](/images/2021-08-30-es-monitoring-and-alert/18.png)


마지막으로 Validation인데, 수집된 시간이 적어 Time range쪽에서 warning이 있는 것을 확인 할 수 있습니다.
보통 24시간정도 수집하고 Job을 만드는 것을 추천드립니다.
![/images/2021-08-30-es-monitoring-and-alert/19.png](/images/2021-08-30-es-monitoring-and-alert/19.png)


마지막으로 지금 까지 구성한 Job에 대한 확인 창입니다.
Create job버튼을 눌러줍니다.
![/images/2021-08-30-es-monitoring-and-alert/20.png](/images/2021-08-30-es-monitoring-and-alert/20.png)



### 이상 탐지 알림 설정
이번에는 이상 탐지를 위한 알림을 설정해 보겠습니다.

Stack Management로 이동하여 Rules and Connectors를 클립해 줍니다.
![/images/2021-08-30-es-monitoring-and-alert/21.png](/images/2021-08-30-es-monitoring-and-alert/21.png)


Create connectors를 눌러 아래 창으로 이동합니다.
저는 Slack을 선택하여 진행하겠습니다.
![/images/2021-08-30-es-monitoring-and-alert/22.png](/images/2021-08-30-es-monitoring-and-alert/22.png)


Connector의 이름을 입력하고, 아래 Create a Slack Webhook URL을 눌러 링크창으로 이동합니다.
![/images/2021-08-30-es-monitoring-and-alert/23.png](/images/2021-08-30-es-monitoring-and-alert/23.png)


공식문서의 내용을 읽고, 아래 이미지와 같이 알림을 받을 채널을 하나 선택하여 줍니다.
![/images/2021-08-30-es-monitoring-and-alert/24.png](/images/2021-08-30-es-monitoring-and-alert/24.png)


정상적으로 진행이 되었다면, webhook url이 생성됩니다.
이 url을 복사하여 Connector 설정창으로 이동하여 입력하여 줍니다.
![/images/2021-08-30-es-monitoring-and-alert/25.png](/images/2021-08-30-es-monitoring-and-alert/25.png)

![/images/2021-08-30-es-monitoring-and-alert/26.png](/images/2021-08-30-es-monitoring-and-alert/26.png)


마지막으로, Message를 입력하여 테스트를 진행하여 줍니다.
테스트가 정상적으로 진행되었다면, Slack 채널로 해당 메시지가 전송되어야 합니다.
![/images/2021-08-30-es-monitoring-and-alert/27.png](/images/2021-08-30-es-monitoring-and-alert/27.png)


Connector가 생성되었으면 이제 Machine Learning Job으로 이동해줍니다.
알림을 적용할 Job의 오른쪽 설정 버튼을 눌러 Create alert rule 버튼을 눌러줍니다.
![/images/2021-08-30-es-monitoring-and-alert/28.png](/images/2021-08-30-es-monitoring-and-alert/28.png)


생성할 알람 Rule Name을 입력하고, Severity를 설정하여 임계점을 설정해 줍니다.
(너무 낮은 Severity는 알람이 많이 오게 됩니다.)
![/images/2021-08-30-es-monitoring-and-alert/29.png](/images/2021-08-30-es-monitoring-and-alert/29.png)
![/images/2021-08-30-es-monitoring-and-alert/30.png](/images/2021-08-30-es-monitoring-and-alert/30.png)


마지막으로 Actions의 Slack을 눌러 위에서 만들었던 Connector를 연결하여 저장해줍니다.
![/images/2021-08-30-es-monitoring-and-alert/31.png](/images/2021-08-30-es-monitoring-and-alert/31.png)


이제, Slack 채널에 설정한 알람이 뜨게 됩니다!
![/images/2021-08-30-es-monitoring-and-alert/32.png](/images/2021-08-30-es-monitoring-and-alert/32.png)

## 마무리

Elasticsearch 모니터링을 사용하면서, 알림이 오는 방법을 찾고 있었는데 손 쉽게 하는 방법이 있어 정리 하여 보았습니다.

Trial 라이센스로 모든 기능을 한번씩 이용해 볼 수 있다는 점에서 굉장히 경험이 많이 되었습니다.


## 참고 자료
- https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation-configuration.html
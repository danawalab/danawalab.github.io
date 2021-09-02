---
layout: post
title: "레디스 모니터링을 위한 대표적인 툴 2가지 알아보기"
description: 레디스 모니터링을 위한 대표적인 툴 2가지 알아보기
date:   2021.09.02. 
writer: "선지호"
categories: Common 
---

## 시작 하기 전에

많은 회사 혹은 사람들이 레디스를 사용하고 있습니다.
이에, 레디스를 단순 사용만 할 뿐 아니라 레디스를 모니터링 할수 있는 툴을 알려드리고자 합니다.

## 모니터링 툴 

이번에 소개할 레디스 모니터링 툴은 아래 두가지 입니다

1) Redis-Stat

2) Grafana Plugin

## Redis-Stat

Redis-Stat은 대표적인 오픈소스 Redis 모니터링 도구입니다.

[Redis-Stat 링크](https://github.com/junegunn/redis-stat, "Redis-Stat")


적용방법은 굉장히 간단합니다.

redis-stat을 설치하고, 실행하면 됩니다.

하지만, 도커를 사용해서 관리하는 편이 조금 더 간편했기에 이번엔 도커를 사용해서 구축해보았습니다

```
docker run \
--name redis-stat \
-p 63790:63790 \
-d insready/redis-stat \
--server <redis 위치, localhost:6379> <또 다른 Redis 위치 ...>
```

위의 도커를 실행하면 자동으로 Redis-stat이 구축이 됩니다.
이제, 한번 접속해 봅시다.

브라우저에 localhost:63790 을 입력하면 아래와 같은 화면이 나옵니다.

간단한 CPU 및 메모리 사용량이 그래프로 표시되고, 2초 간격으로 데이터를 수집합니다.

![/images/2021-09-02-redis-monitoring-tools/1.png](/images/2021-09-02-redis-monitoring-tools/1.png)

## Grafana Plugin

이번에는 대표적인 오픈소스 메트릭 데이터 시각화 도구인 그라파나를 이용하는 방법입니다.

보통 그라파나는 프로메테우스와 같은 메트릭 수집기를 같이 이용하기에 docker-compose를 이용하여 한번 작성해보았습니다.

설치하는 방법은 환경변수에 설치할 플러그인을 명시해주거나, 따로 zip파일을 받아 플러그인 폴더에 압축해제를 해두면 됩니다.

```
version: '2'
services:
  grafana:
    image: grafana/grafana:latest
    user: "root"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=danawa
      # 첫번째 방법
      - GF_INSTALL_PLUGINS=redis-datasource
      # 두번째 방법. 아래 플러그인 폴더를 하나 추가.
      # - /data/grafana/plugins:/var/lib/grafana/plugins  
    restart: always
    ports:
      - 3000:3000
```

위의 도커를 실행하고, 이제 레디스 데이터 소스를 작성해주시면 됩니다

![/images/2021-09-02-redis-monitoring-tools/2.png](/images/2021-09-02-redis-monitoring-tools/2.png)

![/images/2021-09-02-redis-monitoring-tools/3.png](/images/2021-09-02-redis-monitoring-tools/3.png)

![/images/2021-09-02-redis-monitoring-tools/4.png](/images/2021-09-02-redis-monitoring-tools/4.png)

![/images/2021-09-02-redis-monitoring-tools/5.png](/images/2021-09-02-redis-monitoring-tools/5.png)

모두 등록이 되면 data source에 아래와 같이 나옵니다.

![/images/2021-09-02-redis-monitoring-tools/7.png](/images/2021-09-02-redis-monitoring-tools/7.png)

작성한 뒤, 미리 정의된 대시보드를 가져옵니다.

[Grafana Redis Dashboard 링크](https://grafana.com/grafana/dashboards/12776, "Grafana Redis Dashboard")

json 파일을 다운받으셔서 import하시거나 ID를 넣어 진행해주시면 됩니다

![/images/2021-09-02-redis-monitoring-tools/6.png](/images/2021-09-02-redis-monitoring-tools/6.png)

그러면 그라파나 대시보드에 아래와 같은 깔끔한 UI가 나오게 됩니다.

![/images/2021-09-02-redis-monitoring-tools/8.png](/images/2021-09-02-redis-monitoring-tools/8.png)

## 장점 비교 

redis-stat
- 간편하게 레디스를 등록하여 모니터링 할 수 있습니다.

Grafana
- 기존에 그라파나로 모니터링 시스템을 갖추어 두었다면 이를 활용하여 추가로 등록할 수 있습니다.

## 마무리

이번에는 레디스 모니터링 툴 두가지에 대한 소개를 해보았습니다.

현재 처한 상황에 따라 다르겠지만, 상황에 따라 위의 두가지 레디스 모니터링 툴을 고민보면서 도입해 볼 수 있을 것 같습니다.

## 참고 자료
- https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation-configuration.html
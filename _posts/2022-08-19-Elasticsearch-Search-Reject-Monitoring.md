---
layout: post
title:  "Elasticsearch Reject Exception 모니터링"
description: "다나와의 ES Reject Exception 모니터링 방법을 소개합니다."
date:   2022.08.19
writer: "김윤기"
categories: Elastic
---

## Reject Exception ? 

Elasticsearch는 Thread를 통해서 색인이나 검색 요청을 처리합니다.

하지만 자원이 부족하거나 요청량에 부하가 생겨서 더 이상 처리할 Thread가 남아있지 않는 경우 
ES는 요청에 대해서 `Reject Exception`을 발생시킵니다.
그 중 search(검색) 요청에 대해서는 Search Reject Exception을 발생시킵니다.

이 글에서는 실시간으로 Search Reject Exception 발생을 모니터링하는 방안을 소개합니다.

### Reject Exception 확인 방법

ES는 Reject Exception을 기록하고 있으며 
/_cat/thread_pool API를 통해서 사용자들에게 제공합니다.

`cat API`는 여러가지로 활용이 가능합니다.

`v` 매개변수를 사용하면 verbose output을 제공합니다.
`s` 매개변수를 사용하여 정렬을 사용할 수 있습니다.
`format` 매개변수를 사용하여 response의 형태를 지정할 수 있습니다. (text, json, yaml 등)

Search Reject Exception의 갯수를 확인하는 API는 다음과 같습니다.
```
GET /_cat/thread_pool/search?v&s=node_name:asc 

node_name name   active queue rejected
node1     search      4     0    55729
node2     search      4     0    61829
node3     search      4     0    50248
node4     search      7     0    62803
node5     search      4     0    56450
```
rejected의 값이 각 노드에서 발생한 Search Reject Exception의 갯수입니다.

하지만 cat API로 확인한 Reject Exception 갯수는 ES 노드가 기동된 이후로 발생했던 모든 Reject Exception의 `누적값`입니다.
Reject가 언제 발생했는지 추적이 어렵고 바로 대응하기에 어렵다는 단점이 있습니다.

Reject가 발생한 즉시 인지하기 위한 모니터링 방안이 필요했습니다.

## Reject Exception 모니터링

### 주기적으로 API 호출하여 기록하기

먼저 cat API를 주기적으로 호출하여 데이터를 수집해야 합니다. 

다나와에서 자체 개발한 매트릭 스케줄러를 사용합니다.
매트릭 스케줄러는 설정한 시간 주기로 API를 호출하여 데이터를 수집하고 influxDB에 저장하는 프로그램입니다. 
매트릭 스케줄러의 자세한 정보는 다른 포스팅에서 참고 바랍니다.

```
매트릭 스케줄러 설정 파일
{
      "name": "search_reject",
      "url": "http://es.danawa.com:9200/_cat/thread_pool/search?s=node_name&format=json",
      "token": "",
      "cron": "0 */1 * * * *",
      "measurement": "search_reject",
      "mappings": [
        {"field":  "name", "api_field":  "node_name", "tag": true, "default_value": ""},
        {"field":  "count", "api_field":  "rejected", "tag": false}
      ]
}
```
node_name과 rejected 두 필드만 매트릭으로 기록합니다.
1분마다 매트릭을 influxDB로 전송합니다.

### influxQL을 사용하여 그라파나에 노출하기

influxQL을 사용하여 influxDB의 데이터를 가져와서 그라파나 패널에 노출합니다.

![/images/2022-08-19-Elasticsearch-Search-Reject-Monitoring/1.png](/images/2022-08-19-Elasticsearch-Search-Reject-Monitoring/1.png)
```
SELECT last("count") as count FROM "autogen"."search_reject" WHERE ("name" = 'node1') AND $timeFilter GROUP BY time(1m) tz('Asia/Seoul')
```
last function을 사용하여 1분마다 마지막 API 호출의 count 값을 가져옵니다.

Search Reject가 발생하지 않는다면 그래프와 같이 수평선의 형태로 나타납니다.
이로써 Reject Exception의 추적이 가능해졌습니다.

하지만 Reject Exception이 발생하는지 확인하기 위해 그라파나만 들여다 볼 수 없습니다.

![/images/2022-08-19-Elasticsearch-Search-Reject-Monitoring/2.png](/images/2022-08-19-Elasticsearch-Search-Reject-Monitoring/2.png)
```
SELECT difference("count") as count FROM "autogen"."search_reject" WHERE ("name" = 'node1') AND $timeFilter GROUP BY "name"
```
difference function을 사용하여 이전 값과 비교하여 1분마다의 증감을 기록합니다.
이전 값과 비교하므로 여태까지 누적된 Reject Exception 갯수와 상관없이 최근 1분동안 발생한 갯수를 확인 할 수 있습니다.

Reject가 발생하지 않는다면 0의 값으로 일정합니다.

### 그라파나의 alert 기능을 사용하여 Search Reject 발생 전달하기

difference 값이 0보다 커진다는 것은 최근 1분간 Reject exception의 발생을 의미합니다.

![/images/2022-08-19-Elasticsearch-Search-Reject-Monitoring/2.png](/images/2022-08-19-Elasticsearch-Search-Reject-Monitoring/2.png)

alert 조건을 0.5 이상으로 설정하였습니다. Reject Exception이 1건이라도 발생하면 텔레그램으로 메시지를 전송합니다. 

드디어 Reject Exception의 실시간 모니터링이 가능해졌습니다.


## 결론

Search Reject Exception이 발생한다는 것은 사용자에게 검색 결과를 제공하지 못했다는 의미입니다.
그만큼 중요한 성능 지표이며, 실시간으로 모니터링 할 필요성이 있습니다. 

물론 노드 증설이나 서버 성능 향상을 통하여 Reject가 일어나지 않도록 하는 게 가장 바람직합니다. 

하지만 현실적으로 불가능할 경우 Reject 발생 시점만이라도 알게 된다면 더 많은 데이터가 유실되기 전 긴급하게 조치할 기회가 있지 않을까요? 


### 참고 자료

- https://danawalab.github.io/common/2021/04/09/Commom-Monitoring-System.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html
- https://docs.influxdata.com/influxdb/v1.8/query_language/
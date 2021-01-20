---
layout: post
title:  "엘라스틱서치 Left Join API 개선사항"
description: "지난번 엘라스틱서치 API방식으로 전환이 후 CPU 사용률이 높아 성능개선한 부분에 대해 공유합니다. 그리고 추가된 밀터필드 조건 기능도 소개드립니다. Left Join API의 구성 및 사용법은 아래 링크를 통해 확인 부탁드립니다." 
date:   2021.01.20. 13:00:00
writer: "김준우"  
categories: Elastic
---
## 소개

안녕하세요. 다나와 김준우입니다. 지난번 엘라스틱서치 API방식으로 전환이 후 CPU 사용률이 높아 성능개선한 부분에 대해 공유합니다. 그리고 추가된 밀터필드 조건 기능도 소개드립니다. Left Join API의 구성 및 사용법은 아래 참고링크를 통해 확인 부탁드립니다.

## 성능 저하 원인

Left Join API 로직에서 parent와 child 결과를 조합하는 중첩반복문에서 CPU 상승의 원인 되었습니다. parent의 hit 마다 innerHit가 있어, child 결과를 관계된필드의 값과 매칭하여 innerHit의 리스트형으로 추가하였습니다. 이중 반복이 되면서 parent 갯수 * child 갯수 만큼 반복을 수행하게 되었습니다.

Parent Hit * Child Hit = 반복 횟수

테스트에서 반복되는 수치는API 1회 호출당  250,000 회 수행하여 CPU가 상승하였습니다.

![/images/2021-01-20-elasticsearch-left-join-api-Improve/Untitled.png](/images/2021-01-20-elasticsearch-left-join-api-Improve/Untitled.png)

## 성능 개선 방법

Child Hit결과를 list형에서 Map으로 변환하였습니다. Map은 키와 값을 가지는 자료구조입니다. 키는 left join에 사용되는 조인 필드의 값을 문자열로 조합하여  키로 사용합니다. 그리고 Parent Hit를 반복 수행하며 child 에서 키로 사용했던 로직처럼 조인 필드의 값을 기초로 키를 조합합니다. 그러면 Child 의 리스트를 Map에서 1회 접근으로 목록을 가져올수 있게 됩니다. 그리고 Parent의 innerHit에 추가합니다. 이렇게 child와 parent를 각각 반복하게 되면 Parent Hit 갯수 + Child Hit 갯수만큼만 반복하게 됩니다. 테스트에서의 갯수를 예를 들면 API 1회 호출에 1000 회만 반복수행하면 결과를 완성하게 됩니다.

## 성능 테스트
지난번 포스팅과 동일한 조건과 환경에서 진행하였습니다.

Thread: 16   
평균 TPS:  16.89641

|횟수|Label|Samples|Average|Min|Max|Std.Dev.|Error|Throughput|Received KB/sec|Sent KB/sec|Avg. Bytes|
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|1|HTTP Request|2958|972|332|5565|414.72|0.00%|16.38209|33478.5|6.57|2092650|
|2|HTTP Request|3017|954|332|17012|527.74|0.00%|16.6886|34104.89|6.69|2092650|
|3|HTTP Request|3191|902|331|5042|415.35|0.00%|17.61854|36005.31|7.07|2092650|


Thread: 32   
평균 TPS: 15.65094666666667

|횟수|Label|Samples|Average|Min|Max|Std.Dev.|Error|Throughput|Received KB/sec|Sent KB/sec|Avg. Bytes|
|---|---|---|---|---|---|---|---|---|---|---|---|
|1|HTTP Request|2676|2156|454|17661|1327.75|0.00%|14.7555|30154.4|5.92|2092650|
|2|HTTP Request|2911|1980|368|10796|1219.18|0.00%|16.06849|32837.62|6.44|2092650|
|3|HTTP Request|2926|1972|413|9936|1239.8|0.00%|16.12885|32960.98|6.47|2092650|

CPU 사용량

Thread 16: 20%   

![/images/2021-01-20-elasticsearch-left-join-api-Improve/improve-16.png](/images/2021-01-20-elasticsearch-left-join-api-Improve/improve-16.png)

Thread 32: 21%

![/images/2021-01-20-elasticsearch-left-join-api-Improve/improve-32.png](/images/2021-01-20-elasticsearch-left-join-api-Improve/improve-32.png)



## 추가 기능

### 조인 멀티 필드 지원

SQL의 Join 사용시 ON절에 멀티 필드를 사용하여 child 테이블에 row를 필터링할수있습니다.  join 필드에서 parent, child 필드를 리스트형으로 사용하며 순서에맞게 매칭되도록 하였습니다. 아래 예제에선 ref == ref and pk == pk 형식으로 매칭되게 됩니다. 

쿼리 예시

```jsx
{
  "query": {
    "match_all": {}
  },
  "size": 50,
  "join": {
    "index": "child-index",
    "parent": ["ref", "pk"],
    "child": ["ref", "pk"],
    "query": {
      "match_all": {}
    }
  }
}
```

## 정리

중첩반복문사용을 개선하며 CPU 사용률이 확연하게 낮아지는 효과를 확인할 수 있었습니다. 이후 중첩 반복문을 사용할때면 성능에 대해 고려하며 사용해야될거같습니다.


## 참고링크

[1. 엘라스틱서치 Left Join Plugin 소개](/elastic/2020/08/10/ElasticSearch-Left-join-plugin.html)

[2.엘라스틱서치 플러그인 이슈](/elastic/2020/12/30/elasticsearch-plugins-issue.html)

[3. 엘라스틱서치 Left 조인 (확장 API 방식)](/elastic/2021/01/06/elasticsearch-left-join-proxy.html)

---
layout: post
title:  "FASTCAT-ElasticSearch 테스트 - 동적색인"
description: "ElasticSearch과 FASTCAT의 동적색인 비교 테스트."
date:   2020.07.09.
writer: "하선호"
categories: Elastic
---

## 1. 개요
앞에 진행한 색인테스트는 `bulk index`를 이용한 전체색인을 비교한 테스트였으며 이번에 비교할 테스트는 수시로 요청되는 색인에 대한 테스트입니다. 전체색인은 서비스할 상품 전체에 대해 수집하여 색인이고 동적색인은 상품 하나 혹은 특정 상품군 대해 추가적인 Insert, Update, Delete가 발생할 때마다 요청되는 색인입니다. 

1) 색인 및 배포  
2) 검색  
**3) 동적색인**  
4) 동적색인과 검색  


## 2. 테스트 구성

구성은 검색테스트와 동일합니다. 테스트 구성은 다음과 같습니다.

[서버 사양]

|인스턴스명|노드명|CPU|MEMORY|
|:---:|:---:|:---:|:---:|
|M5.8xlarge|호출용 노드|3.1Ghz * 32Core|128G
|M5.12xlarge|Node1(data)|3.1Ghz * 48Core|196G
|M5.12xlarge|Node2(data)|3.1Ghz * 48Core|196G

[동적색인 테스트 구성도]

![/images/2020-07-09-Elasticsearch-DynamicIndex/di-test-diagram.png](/images/2020-07-09-Elasticsearch-DynamicIndex/di-test-diagram.png)

색인 테스트와는 다르게 이번엔 FASTCAT, ElasitcSearch 모두 호출용 노드의 동적색인 요청기를 통해 동적색인을 진행합니다.
동적색인 호출기는 색인용 ndjson 문서를 읽고 하나씩 Node1의 검색엔진으로 동적색인 API를 호출합니다. 호출은 모두 비동기이며
테스트에 사용할 문서는 기존 색인된 V1 인덱스 색인 파일입니다. V1인덱스에 색인한 파일을 사용하므로 V1을 제외한 나머지 9개의 인덱스에 동적색인 요청을 하고 9개의 인덱스 모두 동적색인이 마치는 시간까지를 측정했습니다.
하나의 호출기가 V2~V10에 순차적으로 동적색인을 요청합니다.

## 3. 동적색인 테스트 결과

### 3-1. FASTCAT

 - FASTCAT의 경우 동적색인 API로 색인요청을 받더라도 해당 요청의 내용을 파일형태로 저장합니다. 그리고 내부적인 스케쥴 주기 (1초)마다 해당 파일들에 담긴 색인요청 정보로 색인을 진행합니다. 결과적으로 동적색인 요청으로 쌓인 문서들을 `bulk index`로 색인하는 형태가 됩니다. 아래는 비동기로 한건씩 호출한 요청들이 파일로 쌓이고 전부 처리될 때 까지의 결과입니다.

#### 색인 현황

|인덱스명|상품 수|색인 소요 시간|
|:------:|:---:|:---:|
|V2|4,377,838|1:22:25|
|V3|4,377,837|1:22:55|
|V4|4,377,840|1:22:42|
|V5|4,377,839|1:22:40|
|V6|4,377,838|1:22:58|
|V7|4,377,838|1:22:57|
|V8|4,377,839|1:21:56|
|V9|4,377,839|1:22:15|
|V10|4,377,839|1:22:13|

#### CPU 사용률

![/images/2020-07-09-Elasticsearch-DynamicIndex/di-test-fastcat-cpu.png](/images/2020-07-09-Elasticsearch-DynamicIndex/di-test-fastcat-cpu.png)


FASTCAT의 각 인덱스당 437만건의 색인을 처리했으며 약 1시간 20분 이후 쌓여있던 모든 동적색인 요청이 끝났습니다. CPU사용률은 Node1은 약 75%, Node2 약 50%입니다. Node1에서 받은 동적색인 요청을 Node2로도 보냈기 때문에 CPU를 더 사용하는 것 같습니다.

### 3-2. ElasticSearch

- 앞서 설명드리지 않았지만 색인기, 동적색인 호출기에서 ElasticSearch에 대한 REST API는 `High Level REST Client` 내의 API들을 사용합니다. 처음에는 동적색인의 경우 `indexAsync`로 비동기 요청하였지만 FASTCAT과는 다르게 ElasticSearch의 경우 `bulk index`로 처리되는 게 아니다보니 정상적인 테스트를 진행할 수가 없었습니다. 따라서 ElasticSearch 역시 특정 갯수만큼 상품을 묶어 `bulk index` 로 호출하는 형태로 테스트를 진행했습니다.

아래는 500개씩 묶어 동적색인을 요청 했을 때의 결과입니다.

|인덱스명|상품 수|색인 소요 시간|
|:------:|:---:|:---:|
|V2|4,377,840|0:55:14|
|V3|4,377,840|0:55:14|
|V4|4,377,839|0:55:14|
|V5|4,377,839|0:55:14|
|V6|4,377,839|0:55:14|
|V7|4,377,839|0:55:14|
|V8|4,377,839|0:55:14|
|V9|4,377,839|0:55:14|
|V10|4,377,839|0:55:14|

#### CPU 사용률

![/images/2020-07-09-Elasticsearch-DynamicIndex/di-test-es-cpu.png](/images/2020-07-09-Elasticsearch-DynamicIndex/di-test-es-cpu.png)


마찬가지로 인덱스당 437건의 색인을 처리했고 모두 호출하여 처리하는데 55분정도가 소요되었습니다. CPU는 Node1은 9% Node2는 8%를 사용했습니다. FASTCAT과 비해 약 30%정도 빨라졌고 CPU 사용률 역시 약 80%정도 감소했습니다.


## 4. 결론
이번 테스트에서는 검색엔진이 각각 처리하는 방식이 달라 기준을 어떻게 맞춰서 테스트 해야할지가 고민되었습니다.
결과적으로 두 검색엔진이 `bulk index`로 처리되는 형태로 진행했으며 ElasticSearch의 성능이 FASTCAT보다 소요시간은 더 짧고 CPU 사용률도 크게 줄었음을 확인했습니다.

다음 블로그는 검색과 동적색인이 동시에 발생 했을 때의 테스트입니다.


## 참고 자료
- https://www.elastic.co/guide/en/elasticsearch/client/java-rest/master/java-rest-high-document-index.html
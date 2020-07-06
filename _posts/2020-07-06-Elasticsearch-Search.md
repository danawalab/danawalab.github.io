---
layout: post
title:  "FASTCAT-ELASTICSEARCH 테스트 - 검색"
description: "ELASTICSEARCH과 FASTCAT의 검색 성능에 대한 테스트."
date:   2020.07.06.
writer: "하선호"
categories: Elastic
---

## 1. 개요
현재 다나와는 검색엔진을 FASTCAT에서 ELASTICSEARCH로 변경할 준비를 하고 있습니다. 
하지만 무턱대고 변경할 수는 없고 변경하였을 때 어떤 효과가 있을지 확인이 필요할 것 같습니다. 따라서
 변경하기 앞서 두 검색엔진의 성능에 대한 테스트가 필요하였고 이를 4단계로 나눠 테스트를 진행했습니다.

1) 색인 및 배포  
**2) 검색**  
3) 동적색인  
4) 동적색인과 검색  

이렇게 4단계로 나눠 테스트를 진행하였고 이번 블로그에서는 색인과 배포에 대한 두 검색엔진의 성능을 비교한 결과를 알아보겠습니다.


## 2. 색인 테스트

먼저 비교를 해야하기에 운영중에 있는 검색엔진 구성과 유사한 환경을 테스트의 기준으로 잡았습니다.

테스트 구성은 다음과 같습니다.

[색인 테스트 구성도]
![/images/2020-07-06-Elasticsearch-Index/index-test-diagram.png](/images/2020-07-06-Elasticsearch-Index/index-test-diagram.png)

먼저 색인노드는 AWS EC2의 M5.8xlarge로 CPU는 32코어 메모리는 128GB입니다. 각각의 검색엔진 HEAP 설정은 기존 사용하는 FASTCAT과 동일하게 32GB로 설정하였고
테스트에 사용할 문서는 인덱스 당 3900만씩 가질 수 있도록 미리 분배를 진행해놨습니다.
FASTCAT 같은 경우 색인기가 포함되어 있어서 내부적으로 색인용 파일을 읽어 `Bulk index`를 진행하는데요 ELASTICSEARCH도 따로 색인기를 구성하였습니다. 색인기의 역할은 FASTCAT 과 동일하게 색인용 문서를 읽어 ELASTICSEARCH로 `Bulk Index API`를 호출합니다.

테스트는 각각 5개의 인데스를 동시에 색인하도록 진행하였습니다.

## 3. 색인 테스트 결과

### 3-1. FASTCAT

 - FASTCAT은 bulk 색인시 사용하는 쓰레드를 4개로 설정하였습니다. 이는 내부적으로 4개의 샤드를 갖는 것과 같고 색인 시 소요되는 시간을 줄일 수 있습니다.

#### 색인 현황
|인덱스명|상품 수|색인 소요 시간|
|:------:|:---:|:---:|
|V1|39,401,524|1:34:46|
|V2|40,153,466|1:36:41|
|V3|40,274,432|1:36:29|
|V4|39,396,005|1:35:15|
|V5|39,818,307|1:35:46|

#### CPU 사용률
![/images/2020-07-06-Elasticsearch-Index/index-fastcat-cpu.png](/images/2020-07-06-Elasticsearch-Index/index-fastcat-cpu.png)


FASTCAT의 경우 5개 동시 색인시 CPU는 약 67%를 사용했으며 약 1시간 35분 후 색인이 완료되었습니다.

### 3-2. ELASTICSEARCH

- ELASTICSEARCH도 인덱스 셋팅 설정에 4개의 샤드를 갖도록 설정했고 색인기에서 쓰레드 설정을 5로 설정하였습니다. `bulk size`는 500입니다.

|인덱스명|상품 수|색인 소요 시간|
|:------:|:---:|:---:|
|V1|39,400,573|0:43:42|
|V2|40,152,886|0:44:57|
|V3|40,273,555|0:44:57|
|V4|39,395,383|0:44:30|
|V5|39,817,347|0:44:38|

#### CPU 사용률
![/images/2020-07-06-Elasticsearch-Index/index-es-cpu.png](/images/2020-07-06-Elasticsearch-Index/index-es-cpu.png)

ELASTICSEARCH 경우  CPU는 약 88%를 사용했고 44분 후 색인이 완료되었습니다.
FASTCAT과 비교해볼 때 20% 정도 CPU를 더 사용하였고 그 대신 색인 소요시간이 약 53% 감소하였습니다.

### 3-3 추가 테스트

기본적인 비교는 끝났고 ELASTICSEARCH의 색인을 더 빠르게 할 수 있는지에 대한 추가적인 테스트도 진행해봤습니다.
앞서 테스트에서 나왔단 것 처럼 CPU사용률은 이미 한계를 찍고 있는 것 같아 `bulk size` 수치에 따른 성능비교를 해봤습니다


#### 결과

- 추가적으로 bulk 3000, 10000개의 대한 테스트를 진행하였고 CPU사용률은 모두 91% 였습니다. `Bulk Size` 수치 조정 후 기존 테스트보다 약 4분정도 빨라졌지만 멀티인덱스, 멀티쓰레드에 대한 CPU 사용률이 워낙 높다보니 기대한 것 만큼의 소요시간 감소는 없었던 것 같습니다.

|인덱스명|상품 수|bulk 500|bulk : 3000 | bulk : 10000| 
|:------:|:---:|:---:|:---:|:---:|
|V1|39,400,573|43분|40분|39분|
|V2|40,152,886|44분|41분|40분
|V3|40,273,555|44분|41분|41분
|V4|39,395,383|44분|40분|40분
|V5|39,817,347|44분|41분|40분



## 4. 배포 테스트
다나와에서는 색인용 노드와 검색데이터 노드를 따로 두고 있습니다. 색인용 노드는 `Bulk index`만 수행하는 노드이며 색인을 마치면 데이터를 검색데이터 노드로 전송합니다. 앞서 색인테스트 때 보셨겠지만 `bulk index`은 CPU자원을 많이 사용합니다. 그렇기에 색인과 데이터노드를 같은 노드로 구성하게 된다면 검색과 색인 양쪽에 영향을 미칠 수 있기에 두 노드를 분리해서 구성하고 있습니다. 테스트 구성은 다음과 같습니다.


[배포 테스트 구성도]

![/images/2020-07-06-Elasticsearch-Index/index-test-diagram2.png](/images/2020-07-06-Elasticsearch-Index/index-test-diagram2.png)

- 먼저 기존 인덱스 노드외에 2대의 데이터 노드를 구성합니다. 색인 노드에 있는 FASTCAT, ELASTICSEARCH에는 각 10개의 인덱스가 색인되어 있으며 데이터를 동시에 노드1, 노드2로 배포합니다. 테스트에 앞서서 배포의 개념에 대해 별도로 설명하겠습니다.

### 배포
FASTCAT의 경우 배포란 개념으로 부르고있지만 ELASTICSEARCH의 기능으로 말하자면 `recovery`입니다. `recovery api`를 통해 다른 노드로 인덱스의 샤드를 재배치하는 방식으로 기존 FASTCAT의 배포를 대체할 예정입니다. 그럴려면 색인시 구성된 클러스터로 샤드들이 자동으로 배치되는 일은 없어야 합니다. 즉 인덱스 노드에서만 색인이 되야하는데요 이를 위해서 인덱스 셋팅에 할당 필터 설정을 합니다.

```
index.routing.allocation.include._name={index_node}
```

위 설정으로 색인시 샤드들은 인덱스 노드에만 색인되며 배포시엔 아래 설정처럼 셋팅을 변경하면 각각 프라이머리, 레플리카 샤드가 데이터 노드들로 할당됩니다.
```
ex)
PUT /v*/_settings
{
  "index.routing.allocation.include._name":null,
  "index.routing.allocation.exclude._name":{index_node},
  "index.number_of_replicas":"1"
}
```

샤드 할당,조정등에 대한 기본적인 동시 작업 갯수는 2개로 설정되어 있습니다. 또한 노드당 인,아웃 바운드 트래픽량은 초당 40mb입니다. 테스트에 사용하기 적합한 수치는 아니므로 `_cluster/_settings API`로 변경하겠습니다 `transient`옵션은 일시적인 옵션이며 클러스터가 재기동되면 해당 옵션은 사라집니다. 샤드가4개, 레플리카가1개 인덱스가 총 10개이므로 80개의 동시작업이 발생하게 됩니다. 아래 설정처럼 변경하여 테스트를 진행했습니다.

```
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.node_concurrent_recoveries":100,
    "indices.recovery.max_bytes_per_sec":"1024mb"
  }
}
```

설정에 대한 더 자세한 내용은 아래 링크를 통해 확인해주세요

- 인덱스 리커버리 셋팅 : [Index recovery settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/recovery.html#recovery)
- 인덱스 샤드 허용 필터 : [Index-level shard allocation filtering](https://www.elastic.co/guide/en/elasticsearch/reference/current/shard-allocation-filtering.html)
- 클러스터 단위의 할당, 라우팅 설정 : [cluster.routing.allocation.node_concurrent_recoveries](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html)
- 클러스터 설정 : [Cluster update settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)


## 3. 배포 테스트 결과

- 테스트는 FASTCAT, ELASTICSEARCH 순차적으로 진행했고 FASTCAT은 관리도구, ELASTICSEARCH는 API 호출 방식으로 테스트하였습니다.


### 3-1. FASTCAT

|인덱스명|상품 수|배포 소요 시간|
|:------:|:---:|:---:|
|V1|39,400,573|1:09:30|
|V2|40,152,886|1:12:25|
|V3|40,273,555|1:14:20|
|V4|39,395,383|1:13:35|
|V5|39,818,307|1:14:13|
|V6|39,702,330|1:14:05|
|V7|39,507,008|1:13:57|
|V8|40,157,407|1:13:59|
|V9|40,197,045|1:13:51|
|V10|39,341,785|1:13:41|

#### 트래픽

아웃바운트 트래픽량은 5분간 10GB 이를 BPS로 계산하면 (10Gb/300) * 8 = 0.24Gbps

- 색인 노드

![/images/2020-07-06-Elasticsearch-Index/release-fastcat-index-network.png](/images/2020-07-06-Elasticsearch-Index/release-fastcat-index-network.png)


- 데이터노드1

![/images/2020-07-06-Elasticsearch-Index/release-fastcat-data1-network.png](/images/2020-07-06-Elasticsearch-Index/release-fastcat-data1-network.png)

- 데이터노드2

![/images/2020-07-06-Elasticsearch-Index/release-fastcat-data2-network.png](/images/2020-07-06-Elasticsearch-Index/release-fastcat-data2-network.png)



### 3-2. ELASTICSEARCH

|인덱스명|상품 수|배포 소요 시간|
|:------:|:---:|:---:|
|V1|39,400,489|0:15:08|
|V2|40,152,773|0:16:09|
|V3|40,273,478|0:15:06|
|V4|39,395,267|0:14:08|
|V5|39,817,280|0:16:09|
|V6|39,701,596|0:16:02|
|V7|38,920,021|0:16:05|
|V8|40,156,695|0:15:08|
|V9|40,196,047|0:16:07|
|V10|39,341,048|0:16:01|


#### 트래픽 

아웃바운트 트래픽량은 5분간 10GB 이를 BPS로 계산하면 (31Gb/300) * 8 = 0.82Gbps

- 색인 노드

![/images/2020-07-06-Elasticsearch-Index/release-es-index-network.png](/images/2020-07-06-Elasticsearch-Index/release-es-index-network.png)


- 데이터노드1

![/images/2020-07-06-Elasticsearch-Index/release-es-data1-network.png](/images/2020-07-06-Elasticsearch-Index/release-es-data1-network.png)

- 데이터노드2

![/images/2020-07-06-Elasticsearch-Index/release-es-data2-network.png](/images/2020-07-06-Elasticsearch-Index/release-es-data2-network.png)



## 3. 결론
동작방식에 차이가 있어 테스트 수행방법이 달랐지만 결과적으로 색인, 배포에 있어서는 ELASTICSEARCH가 FASTCAT보다는 더 성능이 좋았습니다. 다나와는 하루 한번의 `Bulk index` 수행이 필요하고 `Bulk index` 동안은 동적으로 호출되는 insert, update 색인에 대해서는 쌓아놓기 때문에 색인, 배포 과정이 길어질수록 쌓아놓은 동적색인을 처리 할때의 자원사용률과 성능저하가 있었는데 ELASTICSEARCH로 교체할 경우 해당 문제가 크게 해소 될 것이라 기대됩니다. 또한 서버의 사양을 산정하는데 도움이 된 테스트였습니다.

다음 블로그는 검색 테스트에 대한 내용입니다.


## 참고 자료
- https://www.elastic.co/guide/en/elasticsearch/reference/current/recovery.html#recovery
- https://www.elastic.co/guide/en/elasticsearch/reference/current/shard-allocation-filtering.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html

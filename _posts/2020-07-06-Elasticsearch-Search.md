---
layout: post
title:  "FASTCAT-ElasticSearch 테스트 - 검색"
description: "ElasticSearch과 FASTCAT의 검색 성능에 대한 테스트."
date:   2020.07.06.
writer: "하선호"
categories: Elastic
---

## 1. 개요
색인,배포 테스트에 이어서 이번에는 검색 성능에 대한 테스트를 진행했습니다.

1) 색인 및 배포  
**2) 검색**  
3) 동적색인  
4) 동적색인과 검색  

## 2. 검색 테스트
검색 테스트의 경우 호출용 노드에 Jmetter를 설치하고 쓰레드 수를 조절하며 5분동안의 검색요청을 통해 측정하였습니다.
다나와 메인검색에 쓰이는 상품검색 쿼리를 사용했으며 앞서 테스트로 배포된 구성을 사용하였습니다. 즉 검색시 10개의 인덱스에서 쿼리 결과를 가져옵니다.

테스트 구성은 다음과 같습니다.

[검색 테스트 구성도]

![/images/2020-07-06-Elasticsearch-Search/search-test-diagram.png](/images/2020-07-06-Elasticsearch-Search/search-test-diagram.png)

2대의 검색노드이며 한대는 검색 요청을 받는 역할도 같이합니다. M5.12xlarge로 CPU는 48코어 메모리는 196GB 사양이며 검색엔진 모두 32Gb Heap 설정을 하였습니다. 
호출용 노드는 M5.4xlarge로 CPU 16코어, 메모리는 65Gb 입니다. 테스트는 순차적으로 진행합니다.

## 3. 검색 테스트 결과

- 성능측정은 Jmeter 결과와 Ec2 인스턴스 지표의 CPU 사용률을 확인하였습니다.

### 3-1. FASTCAT

|쓰레드|호출수|평균 속도(ms)|최대 속도(ms)|TPS|CPU 사용률(%)|
|:------:|:---:|:---:|:---:|:---:|:---:|
|2|10,410|57.42|1194|34.69|Node1 : 7.6% , Node2 : 6%|
|4|22,303|53.65|1247|74.21|Node1 : 15.8% , Node2 : 12.8%|
|8|33,167|72.19|1733|110.52|Node1 : 47.6% , Node2 : 38%|
|16|43,965|108.95|2247|146.49|Node1 : 90.8% , Node2 : 80.8%|
|32|43,709|219.3|2481|145.42|Node1 : 65.7% , Node2 : 60%|

### 3-2. ElasticSearch

|쓰레드|호출수|평균 속도(ms)|최대 속도(ms)|TPS|CPU 사용률(%)|
|:------:|:---:|:---:|:---:|:---:|:---:|
|2|8,063|78.28|1169|26.88|Node1 : 11.6% , Node2 : 11.6%|
|4|14,887|80.47|1251|49.6|Node1 : 31.6% , Node2 : 30%|
|8|23,609|101.52|2584|78.6|Node1 : 46.6% , Node2 : 40.8%|
|16|28,083|170.68|3423|93.29|Node1 : 74.3% , Node2 : 69.8%|
|32|28,498|336.5|6251|94.81|Node1 : 73.8% , Node2 : 68.4%|


### 3-3 FASTCAT-ElasticSearch 비교

- 검색 평균 소요시간이 ElasticSearch가 FASTCAT 검색에 비해 46% 정도 증가했습니다. 그에 따라 TPS는 약 30%가 감소했으며 CPU의 경우는 쓰레드 16일 때를 제외하고 ElasticSearch가 조금 더 높았습니다. 쓰레드 16때는 CPU를 정상적으로 사용하지 못한 것 같습니다.


## 4. 검색 추가 테스트

- ElasticSearch의 검색 성능이 더 낮게 나왔고 어느 부분에서 검색 성능이 지연되는지를 확인하고자 검색테스트 역시 추가적인 테스트를 진행했습니다. 상품검색 쿼리를 5개의 단계로 나누어 추가된 부분에 대한 쿼리 속도를 파악하는 테스트이며 테스트 목록은 아래와 같고 쓰레드는 8개 고정으로 테스트를 진행했습니다.  
  
1) 기본 쿼리  
2) 정렬 추가  
3) 필터 추가  
4) 멀티매치 추가  
5) 스크립트 스코어 추가  

### 4-1 기본 쿼리

- 기본적인 bool 쿼리며 처음에 만들었던 상품 쿼리에서 대부분을 제거한 상태입니다.

쿼리

```
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "TOTALINDEX": "${keyword}"
          }
        }
      ]
    }
  }
}
```

결과
```
호출수 : 192926
TPS : 643
평균 : 12ms
```

### 4-2 정렬 추가

- 기본 쿼리에 스코어, 날짜순 정렬를 추가했습니다

쿼리

```
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "TOTALINDEX": "${keyword}"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "_score": {
        "order": "desc"
      },
      "REGISTERDATE": {
        "order": "desc"
      }
    }
  ]
}
```

결과

- 기존보다 TPS가 소폭 낮아졌지만 평균으로 봤을 때 큰 차이는 없습니다

```
호출수 : 177229
TPS : 590
평균 : 13ms
```

### 4-3 필터 추가

- 쿼리내에 3개의 필터 영역이 추가됬습니다.

쿼리

```
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "TOTALINDEX": "${keyword}"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "DISPYN": "Y"
          }
        },
        {
          "term": {
            "PRICECOMPARISONSTOPYN": "N"
          }
        },
        {
          "term": {
            "CATEGORYDISPYN": "Y"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "_score": {
        "order": "desc"
      },
      "REGISTERDATE": {
        "order": "desc"
      }
    }
  ]
}
```

결과

- 평균속도가 약 4배 증가했으며 그에 따라 TPS도 크게 낮아졌습니다. 필터영역이 검색성능에 영향을 준 것 같습니다.

```
호출수 : 51856
TPS : 172
평균 : 46ms
```

### 4-4 멀티매치

- match 쿼리를 multi_match로 변경하여 여러 필드에서 검색어를 찾고 각 필드별로 차별된 가중치를 적용합니다.

쿼리

```
{
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "${keyword}",
            "fields": [
              "TOTALINDEX",
              "MODELWEIGHT^300000",
              "MAKERKEYWORD^200000",
              "BRANDKEYWORD^200000",
              "CATEGORYWEIGHT^100000"
            ],
            "operator": "OR",
            "tie_breaker": 0
          }
        }
      ],
      "filter": [
        {
          "term": {
            "DISPYN": "Y"
          }
        },
        {
          "term": {
            "PRICECOMPARISONSTOPYN": "N"
          }
        },
        {
          "term": {
            "CATEGORYDISPYN": "Y"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "_score": {
        "order": "desc"
      },
      "REGISTERDATE": {
        "order": "desc"
      }
    }
  ]
}
```

결과

- 필터 적용과 큰 차이가 없었습니다.

```
호출수 : 51133
TPS: 170
평균 : 46ms
```

### 4-5 function_score 추가

- 특정 필드에 대한 추가적인 가중치를 합하기 위해 쿼리를 `function_score` 영역 안에 배치하고 `script_score` 로 `POPULARITYSCORE` 필드의 점수를 계산 할 수 있게 변경했습니다.

쿼리

```
{
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "must": [
            {
              "multi_match": {
                "query": "${keyword}",
                "fields": [
                  "TOTALINDEX",
                  "MODELWEIGHT^300000",
                  "MAKERKEYWORD^200000",
                  "BRANDKEYWORD^200000",
                  "CATEGORYWEIGHT^100000"
                ],
                "operator": "OR",
                "tie_breaker": 0
              }
            }
          ],
          "filter": [
            {
              "term": {
                "DISPYN": "Y"
              }
            },
            {
              "term": {
                "PRICECOMPARISONSTOPYN": "N"
              }
            },
            {
              "term": {
                "CATEGORYDISPYN": "Y"
              }
            }
          ]
        }
      },
      "boost_mode": "sum",
      "functions": [
        {
          "script_score": {
            "script": {
              "source": "doc['POPULARITYSCORE'].value"
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "_score": {
        "order": "desc"
      },
      "REGISTERDATE": {
        "order": "desc"
      }
    }
  ]
}
```

결과

- 필터 적용한 쿼리 기준으로 2배정도 평균 속도가 증가했습니다. `function_score` 역시 검색 성능에 영향을 준 것 같습니다.

```
호출수 : 23552
TPS: 78
평균 : 101ms
```

### 4-6 추가 테스트 결과

- 5번의 테스트중 필터와 `function_score` 적용시 평균 소요시간이 증가했습니다. 
- 쓰레드 8기준으로 FASTCAT의 성능은 평균 72ms, TPS 110.52였습니다.


|테스트 종류|호출수|TPS|평균 소요시간(ms)| 
|:------:|:---:|:---:|:---:|
|기본|192,926|643|12|
|정렬 추가|177,229|590|13|
|필터 추가|51,856|172|46|
|멀티매치 추가|51,133|170|46|
|function_score 추가|23,552|78|101|


## 5. 결론
색인,배포 테스트 때 ElasticSearch의 성능이 더 좋았던 것과 달리 검색에서는 반대로 ElasticSearch의 검색성능이 더 낮게 나왔습니다. 스코어의 계산을 뺄 수 없는 구조라 쿼리 튜닝 혹은 다른 방법으로 이를 개선 할 수 있는지 학인해야할 것 같고 이후 동적색인과 검색을 같이 수행 했을 때도 FASTCAT에 비해 어떤 성능적인 이슈가 있는지 확인이 필요합니다.

다음 블로그는 동적색인 테스트에 대한 내용입니다.


## 참고 자료
- https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html
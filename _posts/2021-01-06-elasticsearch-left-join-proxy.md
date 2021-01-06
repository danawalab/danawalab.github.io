---
layout: post
title:  "엘라스틱서치 Left 조인 (확장 API 방식)"
description: "지난번 엘라스틱서치 Left Join Plugin 소개 포스팅을 통해 엘라스틱서치의 플러그인을 소개해드렸습니다. 하지만 엘라스틱서치 버전 7.9.0 부터 현재  최신 버전인 7.10.1까지 엘라스틱서치 플러그인 이슈가 발생하였습니다. 그렇기에 플러그인 방식은 제한된 사용이 되며 엘라스틱서치에 종속적이기 때문에 다른 방식으로 Left 조인 기능을 구현해야 했습니다.  이번에 소개하는 방식은 엘라스틱서치 앞단에 es-tention-api 서버를 두고 _left 앤드포인트를 제외한 모든 앤드포인트를 엘라스틱서치로 포워드합니다. _left 앤드포인트로 접근시 QuerySDL을 해석하여 Parent, Child 각각 검색을 요청 후 응답하는 방식으로 개발해보았습니다." 
date:   2021.01.06. 13:00:00
writer: "김준우"  
categories: Elastic
---
## 소개

지난번 [엘라스틱서치 Left Join Plugin 소개](https://danawalab.github.io/elastic/2020/08/10/ElasticSearch-Left-join-plugin.html) 포스팅을 통해 엘라스틱서치의 플러그인을 소개해드렸습니다. 하지만 엘라스틱서치 버전 7.9.0 부터 현재  최신 버전인 7.10.1까지 [엘라스틱서치 플러그인 이슈](https://danawalab.github.io/elastic/2020/12/30/elasticsearch-plugins-issue.html)가 발생하였습니다. 그렇기에 플러그인 방식은 제한된 사용이 되며 엘라스틱서치에 종속적이기 때문에 다른 방식으로 Left 조인 기능을 구현해야 했습니다.  이번에 소개하는 방식은 엘라스틱서치 앞단에 es-tention-api 서버를 두고 _left 앤드포인트를 제외한 모든 앤드포인트를 엘라스틱서치로 포워드합니다. _left 앤드포인트로 접근시 QuerySDL을 해석하여 Parent, Child 각각 검색을 요청 후 응답하는 방식으로 개발해보았습니다.

### 시스템 구성

사용자앱 또는 키바나에서 엔드포인드를 es-extention-api 서버로 요청을 보내고, es-extention-api 서버는 엘라스틱서치 노드로 검색 요청을 보내도록 합니다. es-extention-api 서버에서 _left 앤드포인트가 아닌 경우 엘라스틱서치로 전달합니다. _left로 요청하면 ES QueryDSL를 파싱하여 Parent와 Child 쿼리를 추출하여 각각 엘라스틱으로 검색요청을 합니다. 그리고 결과값을 조합하여 사용자에게 전달하게 됩니다.

![/images/2021-01-06-elasticsearch-left-join-proxy/Untitled.png](/images/2021-01-06-elasticsearch-left-join-proxy/Untitled.png)

## 서버 실행방법

### 1. es-extention-api를 다운받습니다.

[https://github.com/danawalab/es-extention-api/releases](https://github.com/danawalab/es-extention-api/releases) 접속하여 최신 버전의 릴리즈를 다운로드 합니다.

### 2. es-extention-api 퍼미션 추가

리눅스에서 실행하기위해서는 실행권한을 부여해야합니다.

```jsx
$ chmod +x application
```

### 3. ex-extention-api 실행합니다.

파라미터를 입력하여 서버를 실행합니다.

```jsx
./application port=9000 es.urls=http://elasticsearch1:9200,http://elasticsearch2:9200 es.user=elastic es.password=password go.env=production
```

### 4. 확인합니다.

아래 명령어를 실행하면 정상적으로 엘라스틱서치 결과를 받는지 확인합니다.

```jsx
$ curl http://localhost:9000
```

### 실행 파라미터

### 파라미터

|옵션|기본값|설명|
|---|---|---|
|address|0.0.0.0|Listen Address|
|port|9000|Listen Port|
|es.urls|http://elasticsearch:9200|엘라스틱서치 URL ,(콤마) 구분하여 입력|
|es.user|""|엘라스틱서치 사용자명|
|es.password|""|엘라스틱서치 비밀번호|
|go.env|"development"|개발/운영 모드|

## 조인 쿼리 방법

_left 조인 querydsl은 _search와 동일하며, 최상위에 join 필드가 추가됩니다. 최상위 query 필드는 parent 검색에 사용됩니다. 그리고 join 필드를 정의하면 parent 결과값의 innerHits 영역에 child 검색 결과를 결합하여 리턴하게 됩니다.

### 샘플 QueryDSL 구조

```jsx
GET /parent-index/_left
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "product_name": {
              "value": "노트북"
            }
          }
        }
      ]
    }
  },
  "size": 20,
  "join": [{
    "index": "child-index",
    "parent": "ref",
    "child": "ref",
    "query": {
      "match": {
        "category": "노트북"
      }
    }
  }]
}
```

### join 필수 값

join을 정의할땐 아래 필드가 필수로 있어야합니다. 그리고 join 필드는 list 또는 Object 타입 상관하지 않습니다. 샘플 처럼 list 타입으로 요청하면 여러 child의 결과를 parent innerHits에 추가하는건 동일합니다. 

|필드|설명|
|---|---|
|index|child 인덱스명|
|parent|parent와 child 매핑할 parent 필드명|
|child|parent와 child 매핑할 child 필드명|
|query|child에서 추가 검색쿼리|


### 조인 검색 예제

아래 예제 확인에 접속하여 parent, child index 생성 및 데이터 추가 후 Left 조인 검색을 테스트 해볼 수 있습니다.

[예제 확인](https://github.com/danawalab/es-extention-api/blob/master/example.txt)

### 조인 검색 예제 결과

상위 hits는 parent 검색결과이고, 맵핑된 키에 따라 innerHit안에 _child 값으로 child 결과가 추가된걸 확인 할 수 있습니다. child 가 여러개 일 경우 child hit결과에  _parent 값이 요청보낸 join 순서가 표시됩니다. 확인용도로만 사용하세요.

```jsx
{
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 0.64072424,
    "hits": [
      {
        "_score": 0.64072424,
        "_index": "parent-index",
        "_type": "_doc",
        "_id": "fwGl0HYBlXHSsN6lSSXV",
        "_seq_no": null,
        "_primary_term": null,
        "_source": {
          "ref": "REF_00001",
          "pk": "PK_00001",
          "product_name": "삼성 노트북"
        },
        "inner_hits": {
          "_child": {
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 1.6931472,
              "hits": [
                {
                  "_score": 1.6931472,
                  "_index": "child-index",
                  "_type": "_doc",
                  "_id": "ngGl0HYBlXHSsN6lYCVm",
                  "_parent": "0",
                  "_seq_no": null,
                  "_primary_term": null,
                  "_source": {
                    "ref": "REF_00001",
                    "category": "노트북"
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_score": 0.64072424,
        "_index": "parent-index",
        "_type": "_doc",
        "_id": "hgGl0HYBlXHSsN6lTiVZ",
        "_seq_no": null,
        "_primary_term": null,
        "_source": {
          "ref": "REF_00001",
          "pk": "PK_00002",
          "product_name": "LG 노트북"
        },
        "inner_hits": {
          "_child": {
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 1.6931472,
              "hits": [
                {
                  "_score": 1.6931472,
                  "_index": "child-index",
                  "_type": "_doc",
                  "_id": "ngGl0HYBlXHSsN6lYCVm",
                  "_parent": "0",
                  "_seq_no": null,
                  "_primary_term": null,
                  "_source": {
                    "ref": "REF_00001",
                    "category": "노트북"
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_score": 0.64072424,
        "_index": "parent-index",
        "_type": "_doc",
        "_id": "jAGl0HYBlXHSsN6lUiUx",
        "_seq_no": null,
        "_primary_term": null,
        "_source": {
          "ref": "REF_00001",
          "pk": "PK_00003",
          "product_name": "Asus 노트북"
        },
        "inner_hits": {
          "_child": {
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 1.6931472,
              "hits": [
                {
                  "_score": 1.6931472,
                  "_index": "child-index",
                  "_type": "_doc",
                  "_id": "ngGl0HYBlXHSsN6lYCVm",
                  "_parent": "0",
                  "_seq_no": null,
                  "_primary_term": null,
                  "_source": {
                    "ref": "REF_00001",
                    "category": "노트북"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  },
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  }
}
```

## 정리

엘라스틱서치의 Left 조인 기능을 플러그인 방식에서 확장 API 서버 방식으로 변경해보았습니다. 기존에 사용하던 플러그인 방식과 다르게 확장성이 높아진거 같습니다. 엘라스틱서치에서 미지원하던 검색을 자유롭게 개발할 수 있고, 엘라스틱서치에 종속적이지 않아 버전과 무관하게 사용할 수 있는 장점이 있는거 같습니다. 엘라스틱서치에 Left 조인이 필요한 상황에서 도움이 되었으면 좋겠습니다.

## 링크

- [https://github.com/danawalab/es-extention-api](https://github.com/danawalab/es-extention-api)
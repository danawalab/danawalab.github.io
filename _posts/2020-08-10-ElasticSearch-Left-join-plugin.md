---
layout: post
title:  "엘라스틱서치 Left Join Plugin 소개"
description: "이번 포스팅 내용은 엘라스틱서치에서 제공하는 join을 사용하지 않고, 자체적으로 개발한 Left Join 플러그인을 소개하도록 하겠습니다.  RDB에서 join은 아래 이미지 처럼 A, B 테이블이 있을때 조인을 통해 원하는 결과로 조합할 수 있습니다. 엘라스틱서치에서는 아래 이미지에서 나오지 않는 조인 기능을 제공하고있습니다. A를 검색했을때 B의 결과를 가져오거나, B를 검색하면 A의 결과를 가져오는 기능만 제공하고 있습니다. 그래서 A의 결과와 B의 결과를 같이 검색을 하는 방법은 없기 때문에 플러그인을 개발하였습니다."
date:   2020.08.10.
writer: "김준우"
categories: Elastic
---
## 소개

이번 포스팅 내용은 엘라스틱서치에서 제공하는 join을 사용하지 않고, 자체적으로 개발한 Left Join 플러그인을 소개하도록 하겠습니다.  RDB에서 join은 아래 이미지 처럼 A, B 테이블이 있을때 조인을 통해 원하는 결과로 조합할 수 있습니다. 엘라스틱서치에서는 아래 이미지에서 나오지 않는 조인 기능을 제공하고있습니다. A를 검색했을때 B의 결과를 가져오거나, B를 검색하면 A의 결과를 가져오는 기능만 제공하고 있습니다. 그래서 A의 결과와 B의 결과를 같이 검색을 하는 방법은 없기 때문에 플러그인을 개발하였습니다.

![/images/2020-08-10-ElasticSearch-Left-join-plugin/Untitled.png](/images/2020-08-10-ElasticSearch-Left-join-plugin/Untitled.png)

## Left 조인 플러그인 소개

엘라스틱서치의 검색쿼리구조로 사용 가능하며, 플러그인에서 사용하는 필드를 추가하여 검색 결과의 Hits 안에 innerHit로 child index 데이터를 조합하여 검색하는 방식입니다.

### Left 조인 플러그인 사용법

엘라스틱서치 버전: 7.6.2

자바 버전: 11

### 설치방법

```jsx
$ elasticsearch-plugin install https://github.com/danawalab/left-join-plugin/releases/download/v1.0/join-plugin.zip
```

### 검색 쿼리 작성방법

_search 쿼리와 동일하지만 join 필드가 추가된걸 확인할 수 있습니다. object, array 형식으로 작성가능합니다.

```jsx
GET /<parent 인덱스명>/_left
{
  "query": { [parent 검색 쿼리] },
  "join": {  
    "index": <child 인덱스명>,
    "parent": <parent field 명>,
    "child": <child field 명>,
    "query": { [child 검색 쿼리] }
  }
}

```

Ex 1)  간단하게 parent의 colum1에 danawa 키워드로 검색하고, parent 결과의 fk 필드의 값을 child pk로 검색하는 쿼리입니다.

```jsx
GET /parent/_left
{
  "query": {
    "match": {
      "column1": "danawa"
    }
  },
 "join": {
    "index": "child",
    "parent": "fk",
    "child": "pk"
  }
}
```

OUTPUT

```jsx
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.2039728,
    "hits" : [
      {
        "_index" : "parent",
        "_type" : "_doc",
        "_id" : "Del91nMBpjLcSSoHoZkt",
        "_score" : 1.2039728,
        "_source" : {
          "column1" : "danawa",
          "fk" : "child_danawa"
        },
        "inner_hits" : {
          "_child" : {
            "hits" : {
              "total" : {
                "value" : 1,
                "relation" : "eq"
              },
              "max_score" : 1.0,
              "hits" : [
                {
                  "_index" : "child",
                  "_type" : "_doc",
                  "_id" : "DOl91nMBpjLcSSoHnpl6",
                  "_score" : 1.0,
                  "_source" : {
                    "pk" : "child_danawa",
                    "column3" : "Product Search",
                    "column4" : "Sample Keyword"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

Ex 2) parent의 모든 문서를 조회하고, child 의 pk가 key1인 문서를 innerHit로 조합합니다.

```jsx
GET /parent/_left
{
  "query": {
    "match_all": {}
  }
  ,"join": [
    {
      "index": "child",
      "parent": "fk",
      "child": "pk",
      "query": {
        "match": {
          "pk": "key1"
        }
      }
    }
  ]
}
```

OUTPUT

```jsx
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "parent",
        "_type" : "_doc",
        "_id" : "AYE3xnMB7hQL8Ao9pTb9",
        "_score" : 1.0,
        "_source" : {
          "column1" : "a",
          "column2" : "a",
          "fk" : "key1"
        },
    .... 중략     
      },
      {
        "_index" : "parent",
        "_type" : "_doc",
        "_id" : "Del91nMBpjLcSSoHoZkt",
        "_score" : 1.0,
        "_source" : {
          "column1" : "danawa",
          "fk" : "child_danawa"
        },
        "inner_hits" : {
          "_child" : {
            "hits" : {
              "total" : {
                "value" : 0,
                "relation" : "eq"
              },
              "max_score" : 0.0,
              "hits" : [ ]
            }
          }
        }
      }
    ]
  }
}
```

### 동작 방식

1. parent 검색 쿼리를 호출합니다.
2. child 조인에 해당하는 필드의 값을 추출합니다.
3. parent 검색 쿼리의 조건값을 child 검색 쿼리로 조합하여 검색합니다.
4. child 데이터를 parent innerHit로 병합합니다.
5. 조합된 결과를 검색 요청의 결과로 응답합니다.

![/images/2020-08-10-ElasticSearch-Left-join-plugin/Untitled%201.png](/images/2020-08-10-ElasticSearch-Left-join-plugin/Untitled%201.png)

## 정리

엘라스틱서치의 Left Join 플러그인을 구현해보고, Left Join 검색을 진행 해보았습니다. 플러그인을 통해 얻게 되는 편의점으로는 엘라스틱서치에서 제공하는 join을 사용하게 되면 parent, child 인덱스가 동일한 샤드에 존재해야는 불편함이 있었습니다. 하지만 개발된 플러그인에서는 단순히 parent 조회하여 특정 필드의 값을 가지고 child에 검색을 진행하기 때문에 parent, child의 연관 또는 맵핑이  없어도 되었습니다. 단점으로는 많은 비용을 소요하는 작업이라고 볼 수 있습니다. parent 인덱스에 검색을 하고, child 인덱스에 전체 문서를 조회해야하는 작업을 진행햐아합니다. 많은 양의 문서를 가지고 있는 인덱스가 아니라면 충분히 사용해볼만한 플러그인이라고 생각합니다.

Left Join Plugin GitHub

[https://github.com/danawalab/join-plugin](https://github.com/danawalab/join-plugin)
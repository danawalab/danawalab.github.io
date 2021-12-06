---
layout: post
title:  "엘라스틱 서치의 update_by_query의 기능"
description: "엘라스틱 서치의 update_by_query의 기능"
date:   2021.12.06. 10:00:00
writer: "선지호"
categories: Elastic
---
## 소개

Elasticsearch는 이미 색인된 문서에 대해서는 사전에 관련 단어를 업데이트 하더라도 적용이 되지 않았습니다.

따라서, 이를 해결하기 위해서는 아래의 방법들을 사용해야 합니다.

- 1) 새 인덱스를 만들어 색인
- 2) reindex
- 3) update_by_query

update_by_query를 사용하여 사전 업데이트를 진행하는 방법과, update_by_query를 사용하는 방법을 상세히 설명해보겠습니다.

이번에 포스팅할 내용은 다음과 같습니다.

- 쿼리로 매칭된 문서 업데이트
- 이미 색인된 문서의 분석 내용 업데이트  

### 공식문서

[Elasticsearch update-by-query](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html, "update-by-query")

## 쿼리로 매칭된 문서 업데이트

update-by-query는 쿼리로 매칭된 문서를 업데이트 할 수 있습니다.

elasticsearch에서 제공하는 한개의 문서를 업데이트 하는 기능 만으로는 충분하지 않을 때 사용할 수 있습니다.

아래와 같이 사용할 수 있습니다.

```JSX
POST accounts-a/_update_by_query
{
  // 업데이트를 할 내용을 명시하는 필드 입니다.
  // 보통 ctx._source로 접근하여 필드들을 업데이트 합니다.
  "script": {   
    "source": "ctx._source.count++",
    "lang": "painless"
  },
  // 찾을 문서들을 쿼리로 검색합니다.
  "query": {
    "match_all": {
      
    }
  }
}
```

이번에 사용할 데이터는 엘라스틱서치에서 제공하는 샘플데이터 중 accounts 를 이용해보겠습니다.

인덱스명은 accounts-a로 생성했습니다.

[Elasticsearch sample data](https://www.elastic.co/guide/kr/kibana/current/tutorial-load-dataset.html, "sample data")

인덱스에 넣은 매핑과 셋팅은 아래 이미지와 같습니다.

### 매핑
![/images/2021-12-06-Elasticsearch-update-by-query/1.jpg](/images/2021-12-06-Elasticsearch-update-by-query/1.jpg)

### 셋팅
![/images/2021-12-06-Elasticsearch-update-by-query/2.jpg](/images/2021-12-06-Elasticsearch-update-by-query/2.jpg)

현재 나이가 30세 미만인 사람을 검색해보겠습니다.

```jsx
POST accounts-a/_search
{
  "query": {
    "range": {
      "age": {
        "lt": 30
      }
    }
  }
}
```

![/images/2021-12-06-Elasticsearch-update-by-query/1.jpg](/images/2021-12-06-Elasticsearch-update-by-query/3.jpg)

총 451건이 나옵니다.

이번에는 update-by-query를 사용해 30세 미만인 사람들의 나이를 전부 2살씩 올리겠습니다.

```jsx

POST accounts-a/_update_by_query
{
  "script": {
    "source": "ctx._source.age = ctx._source.age + 2;",
    "lang": "painless"
  }, 
  "query": {
    "range": {
      "age": {
        "lt": 30
      }
    }
  }
}
```

실행 결과로 이전에 검색한 갯수인 451건이 업데이트가 되었습니다.

![/images/2021-12-06-Elasticsearch-update-by-query/4.jpg](/images/2021-12-06-Elasticsearch-update-by-query/4.jpg)

한번 내용을 확인해 보겠습니다. 

![/images/2021-12-06-Elasticsearch-update-by-query/5.jpg](/images/2021-12-06-Elasticsearch-update-by-query/5.jpg)

이번에는 365건이 조회가 됩니다.

성공적으로 업데이트가 완료된 것을 확인 할 수 있었습니다.


## 이미 색인된 문서의 분석 내용 업데이트 

이번에는 색인된 문서의 내용을 업데이트 하는 법을 알아보겠습니다.

먼저, 모든 인덱스 문서에 데이터를 넣어보겠습니다

```jsx
POST accounts-a/_update_by_query
{
  "script": {
    "source": "ctx._source.made = \"다나다나와\"",
    "lang": "painless"
  }, 
  "query": {
    "match_all": {}
  }
}
```
![/images/2021-12-06-Elasticsearch-update-by-query/6.jpg](/images/2021-12-06-Elasticsearch-update-by-query/6.jpg)

문서에 어떻게 분석이 되어 들어가있는지 확인해보겠습니다.

```jsx
POST accounts-a/_termvectors/KK1kjX0BA-W2OvAgr90z
{
  "fields": ["made"]
}
```

![/images/2021-12-06-Elasticsearch-update-by-query/7.jpg](/images/2021-12-06-Elasticsearch-update-by-query/7.jpg)

'다나' / '다나와' 두개의 텀으로 분석이 되었습니다.

이번에는 사전에 '다나다나와'를 추가해서 한번 확인해보겠습니다.
(사전에 추가할 때에는 디서치 관리도구를 사용했습니다.)

![/images/2021-12-06-Elasticsearch-update-by-query/8.jpg](/images/2021-12-06-Elasticsearch-update-by-query/8.jpg)

![/images/2021-12-06-Elasticsearch-update-by-query/9.jpg](/images/2021-12-06-Elasticsearch-update-by-query/9.jpg)

```jsx
POST accounts-a/_analyze
{
  "analyzer": "custom_analyzer",
  "text":"다나다나와"
}
```

사전을 적용하니 의도 했던 대로 한개의 텀으로 나옵니다.

![/images/2021-12-06-Elasticsearch-update-by-query/10.jpg](/images/2021-12-06-Elasticsearch-update-by-query/10.jpg)

하지만 다시 아래의 쿼리로 분석된 내용을 확인 해보아도 업데이트 되어 분석이 되지 않습니다.

```jsx
POST accounts-a/_termvectors/KK1kjX0BA-W2OvAgr90z
{
  "fields": ["made"]
}
```

![/images/2021-12-06-Elasticsearch-update-by-query/11.jpg](/images/2021-12-06-Elasticsearch-update-by-query/11.jpg)

update-by-query로 분석된 내용을 다시 한번 업데이트한 후 다시 텀 분석을 진행해 보겠습니다.

```jsx
POST accounts-a/_update_by_query
{
  "query": {
    "match_all": {}
  }
}

POST accounts-a/_termvectors/KK1kjX0BA-W2OvAgr90z
{
  "fields": ["made"]
}
```
![/images/2021-12-06-Elasticsearch-update-by-query/12.jpg](/images/2021-12-06-Elasticsearch-update-by-query/12.jpg)

업데이트 된 텀 분석 내용으로 바뀐것을 확인 할 수 있습니다.


## 정리

엘라스틱서치에서 update-by-query를 사용하여 업데이트 하는 방법 및 텀 분석 또한 업데이트 되는 내용을 확인했습니다.

앞으로 사전의 분석내용을 변경 할 경우 update-by-query를 사용하여 새로 색인을 하지 않는 방법으로 진행하면 리소스 측면에서 많은 절약을 할 수 있을 것 같습니다.

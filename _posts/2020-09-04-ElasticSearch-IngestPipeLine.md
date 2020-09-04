---
layout: post
title:  "Ingest PipeLine 설정을 통한 데이터 전처리"
description: "PipeLine 내에 여러 processor를 적용하여 색인 전 데이터를 가공해보기"
date:   2020.09.04.
writer: "하선호"
categories: Elastic
---

## 1. 개요
ElasticSearch로 색인하다보면 필요에 따라 데이터를 가공해야할 필요가 있습니다. 예를 들어 구분값으로 나눠진 데이터를 집계해야 할 때, HTML 태그가 포함된 필드를 색인 할 때 등 그래서 이러한 데이터를 별도의 프로세스를 통해 가공하여 색인했었는데요. 인덱스 별로 가공해야 할 필드가 다르고 필드명 또한 달라서 매번 인덱스에 해당하는 프로세스를 만들고, 배포하는 작업이 매우 불편할 것 같았습니다. 그런 상태에서 `Pipeline` 의 `processor`에 대한 조언을 받아 알아보게 되었습니다. `Pipeline`은 아래 링크를 통해 확인 하실 수 있습니다.
- https://www.elastic.co/guide/en/elasticsearch/reference/current/pipeline.html


## 2. PipeLine

우선 `PipeLine` 설정하기 위해서는 `Ingest Node`가 활성화 되어있어야 합니다. `PipeLine` 설정은 다음와 같습니다.

```
PUT _ingest/pipline/{pipline name}
{
  "description" : "...",
  "processors" : [ ... ]
}
```
`description` 은 해당 `pipeLine`에 대한 설명 필드이며 `processors`에는 `pipeLine`에서 수행할 
`processors`에 대한 정의를 작성하는 필드입니다. 정의한 순서대로 수행되며 여러개의 `processors`를 등록할 수 있습니다.

## 3. Processor

`processors` 는 `PROCESSOR_NAME`는 커스텀한 네임이 아닌 `processors` 별 지정된 이름과 함께 지정된 옵션을 정의해주시면 됩니다.
```
{
  "PROCESSOR_NAME" : {
    ... processor configuration options ...
  }
}
```
### 3-1. Split processor

첫번째로 사용해본 `processor`는 `Split processor`입니다. 
정의된 필드의 데이터를 구분자로 분해하여 배열형태로 만들어줍니다.
```
PUT _ingest/pipeline/split_field
{
  "description": "Decompose the field by separator",
  "processors": [
    {
      "split": {
        "field": "productList",
        "separator": ","
      }
    }
  ]
}
```
`_simulate` API를 통해서 생성할 `processor`를 확인해 볼 수도 있습니다. 위에 설명드린대로 `pipeline` 필드를 작성하고 `doc` - `_source` 필드에 값을 입력하면 됩니다.

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "Decompose the field by separator",
    "processors": [
      {
        "split": {
          "field": "productList",
          "separator": ","
        }
      }
    ]
  },
  "docs": [
    {
      "_source": {
        "productList": "1,2,3,4,5"
      }
    }
  ]
}
```
위 처럼 `_simulate`를 통해 아래처럼 입력된 값이 구분자를 통해 배열로 출력되는걸 확인할 수 있습니다.

```
{
  "docs" : [
    {
      "doc" : {
        "_index" : "_index",
        "_type" : "_doc",
        "_id" : "_id",
        "_source" : {
          "productList" : [
            "1",
            "2",
            "3",
            "4",
            "5"
          ]
        },
        "_ingest" : {
          "timestamp" : "2020-09-04T02:23:40.241192Z"
        }
      }
    }
  ]
}
```
### 3-1. HTML strip processor
두번째로 사용해볼 `processor`는 `HTML strip processor`입니다. 필드의 HTML Tag를 제거해줍니다.


```
PUT _ingest/pipeline/html_strip
{
  "description": "Remove Html Tag", 
  "processors": [
    {
      "html_strip": {
        "field": "contents"
      }
    }
  ]
}
```

`_simulate`를 통해 확인해보면 `<em>:</em>` 태그가 제거되었습니다.

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "Remove Html Tag",
    "processors": [
      {
        "html_strip": {
          "field": "contents"
        }
      }
    ]
  },
  "docs": [
    {
      "_source": {
        "contents": "<em>다나와 ElasticSearch</em>"
      }
    }
  ]
}
```
## 4. Index 적용
정의한 `PipeLine` 을 적용하여 색인 할 때는 `index API` 뒤에 `pipeline={pipeline_name}` 을 붙여 호출하면 됩니다.

아래 예제는 `PipeLine` 정의 후 색인까지의 예제입니다.

```
## 인덱스 생성
PUT sample_index
{
  "mappings": {
    "properties": {
      "boardContents": {
        "type": "text"
      },
      "productSeqList": {
        "type": "keyword"
      }
    }
  }
}


## PipeLine 정의
PUT _ingest/pipeline/sample_pipeLine
{
  "description": "sample", 
  "processors": [
    {
      "html_strip": {
        "field": "boardContents"
      }
    },
    {
      "trim": {
        "field": "productSeqList"
      }
    },
    {
      "split": {
        "field": "productSeqList",
        "separator": ","
      }
    }
  ]
}

##색인
POST sample_index/_doc?pipeline=sample_pipeLine
{
  "boardContents" : "<em>sample 제목</em>",
  "productSeqList" : "001,002,003"
}
```

결과

```
    {
        "_index" : "sample_index",
        "_type" : "_doc",
        "_id" : "RjRaV3QBZ9BimQhnrQr8",
        "_score" : 1.0,
        "_source" : {
          "boardContents" : "sample 제목",
          "productSeqList" : [
            "001",
            "002",
            "003"
          ]
        }
      }
```

## 5. 결론
인덱스마다 별도의 데이터 전처리하는 과정을 따로 만드는게 번거롭고 관리에 있어서 불편하다 생각했는데 이번에 `PipeLine`을 알게되면서 더 용이한 방법으로 색인전 전처리를 할 수 있었습니다. 설명한 2개의 `processor`뿐 아니라 `csv`, `grok` 등 다양한 `processor`가 있으며 이를 통해 더 색인시 다양한 데이터 처리를 할수 있습니다

## 참고
- [https://www.elastic.co/guide/en/elasticsearch/reference/current/pipeline.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/pipeline.html)
- [https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest-processors.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest-processors.html)
- [https://www.elastic.co/guide/en/elasticsearch/reference/current/split-processor.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/split-processor.html)
- [https://www.elastic.co/guide/en/elasticsearch/reference/current/htmlstrip-processor.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/htmlstrip-processor.html)
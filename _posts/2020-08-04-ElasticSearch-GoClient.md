---
layout: post
title:  "다나와 검색API 개발을 위한 ElasticSearch - GoClient 선택하기"
description: "다나와 검색API를 ElasticSearch로 적용하기 위해 적합한 Client를 찾는다."
date:   2020.08.04.
writer: "하선호"
categories: Elastic
---

## 1. 개요
검색엔진이 ElasticSearch로 변경되면 기존에 FASTCAT을 연동한 검색API에서도 ElasticSearch으로 적용을 해야합니다. ElasticSearch 공식홈을 확인해보면 다양한 언어에 대한 클라이언트를 제공하고 있는데요
다나와 검색 API는 Go언어로 작성되었고 Go 언어에 대한 클라이언트도 제공하고 있어서 사용하면 될 것 같습니다.

[제공하는 언어들]

![/images/2020-08-04-ElasticSearch-GoClient/elasticsearch-client.png](/images/2020-08-04-ElasticSearch-GoClient/elasticsearch-client.png)

## 2. Go API - ElasticSearch Client

Github 페이지의 README문서를 통해 GoMod에 추가할 의존성정보와 Go file내에서 어떻게 ElasticSearch와 연동하는지 확인할 수 있었습니다.
```
// go.mod
github.com/elastic/go-elasticsearch/v7 7.x

// main.go
import (
  elasticsearch7 "github.com/elastic/go-elasticsearch/v7"
)
// ...
cfg := elasticsearch.Config{
  Addresses: []string{
    "http://localhost:9200",
    "http://localhost:9201",
  },
  // ...
}
es, err := elasticsearch.NewClient(cfg)
```

검색API인 만큼 ES의 Search API에 대한 처리 부분을 확인해보니 아래처럼 쿼리를 생성해야합니다.

```
  // 3. Search for the indexed documents
  //
  // Build the request body.
  var buf bytes.Buffer
  query := map[string]interface{}{
    "query": map[string]interface{}{
      "match": map[string]interface{}{
        "title": "test",
      },
    },
  }
```

map 안에 구문을 계속 넣는 방식으로 쿼리식을 생성하는 방식이었습니다. 이럴 경우 많은 검색 파라메터에 대해 쿼리식을 생성하기 복잡하고 유지보수 하기도 번거로울 것 같습니다.
공식적으로 제공하는 JAVA의HighLevel Client에서는 Query DSL에 대한 QueryBuilder와 각 쿼리 구문을 메소드로 구현해줬는데 말이죠..

그래서 다른 client가 있는지 찾아봤습니다. 공식적으로 제공하는 것 외에 사용자들이 만들어 제공하는 client가 있어서 확인해봤습니다.


[Community Contributed Clients - Golang]

![/images/2020-08-04-ElasticSearch-GoClient/community-elasticsearch-client.png](/images/2020-08-04-ElasticSearch-GoClient/community-elasticsearch-client.png)

그 중 사용해볼 것은 2번째 링크입니다. 사실 나머지 2개는 커밋이력이 몇년 전으로 관리되고 있지 않았습니다...

[https://github.com/olivere/elastic](https://github.com/olivere/elastic)

```
// Search with a term query
	termQuery := elastic.NewTermQuery("user", "olivere")
	searchResult, err := client.Search().
		Index("twitter").   // search in index "twitter"
		Query(termQuery).   // specify the query
		Sort("user", true). // sort by "user" field, ascending
		From(0).Size(10).   // take documents 0-9
		Pretty(true).       // pretty print request and response JSON
		Do(ctx)             // execute
	if err != nil {
		// Handle error
		panic(err)
	}
```

검색 예제를 확인해보니 Query DSL과 구문에 대한 func을 제공하여 더 쉽게 쿼리식을 생성할 수 있을 것 같습니다. 위 client를 사용하여 샘플을 작성해봤습니다.

먼저 go mod에 client 정보를 추가 하고 go file에 import에 추가합니다.

```
//go.mod
github.com/olivere/elastic/v7

//go file
import (
    "github.com/olivere/elastic"
)
```

이후 es 연결 객체를 생성합니다.

```
client, err = esclient.NewClient(
    esclient.SetURL("localhost:9200"), 
    esclient.SetSniff(false),
    esclient.SetHealthcheckInterval(10*time.Second),
    esclient.SetMaxRetries(5))
if err != nil {
    // Handle error
}
return
```
아래의 쿼리를 생성해봤습니다. BoolQuery 기반으로 must, must_not, should, filter 영역이 모두 포함된 쿼리입니다.
```
"query": {
    "bool": {
      "must": [
        {
         "match": {
           "CATEGORYCODE3": 40972
         }
        }
      ],
      "must_not": [
        {
          "match": {
            "EXPOSUREYN": "N"
          }
        }
      ],
      "should": [
        {
          "terms": {
            "MAKERCODE": [
              "91413",
              "27615"
            ]
          }
        }
      ],
      "filter": [
        {
          "term": {
            "DISCONTINUED": "N"
          }
        },
         {
          "term": {
            "DISPYN": "Y"
          }
        },
        {
          "term": {
            "PRICECOMPARISONSTOPYN": "N"
          }
        }
      ]
    }

```

각각 must, should 등에 들어갈 match, term 쿼리를 생성하고 BoolQuery에 넣어주면 됩니다. 
파라메터 유무에 따라 필요한 쿼리들을 생성하면 되니 구현 및 관리가 편할 것 같단 생각이 듭니다.

```
func makeQuery() {

    //boolQuery
    query := elastic.NewBoolQuery()
 
    //must 영역
    mustMatchQuery := elastic.NewMatchQuery("CATEGORYCODE3", 40972)

    //must not 영역
    mustNotmatchQeury := elastic.NewMatchQuery("EXPOSUREYN", "N")
 
    //should 영역
    shouldTermsQuery := elastic.NewTermsQuery("MAKERCODE", "91413", "27615")
 
    //filter 영역
    filter1 := elastic.NewTermQuery("DISCONTINUED", "N")
    filter2 := elastic.NewTermQuery("DISPYN", "Y")
    filter3 := elastic.NewTermQuery("PRICECOMPARISONSTOPYN", "N")
 
    query.Must(mustMatchQuery)
    query.MustNot(mustNotmatchQeury)
    query.Should(shouldTermsQuery)
    query.Filter(filter1, filter2, filter3)
 
    //생성된 쿼리 스트링 확인
    src, err := query.Source()
 
    q, err := json.MarshalIndent(src, "", "  ")
 
    fmt.Println(string(q))
     
}

```

쿼리 스트링을 확인해보니 순서나 must,must_not 안의 항목 갯수로 인해 배열로 묶이지 않은 것 외에 위와 동일한 쿼리가 생성된 것을 확인했습니다.

```
{
  "bool": {
    "filter": [
      {
        "term": {
          "DISCONTINUED": "N"
        }
      },
      {
        "term": {
          "DISPYN": "Y"
        }
      },
      {
        "term": {
          "PRICECOMPARISONSTOPYN": "N"
        }
      }
    ],
    "must": {
      "match": {
        "CATEGORYCODE3": {
          "query": 40972
        }
      }
    },
    "must_not": {
      "match": {
        "EXPOSUREYN": {
          "query": "N"
        }
      }
    },
    "should": {
      "terms": {
        "MAKERCODE": [
          "91413",
          "27615"
        ]
      }
    }
  }
}
```


## 3. 결론
Go 기반의 RestAPI에 ElasticSearch를 적용하기 위해 클라이언트를 찾고 적용해봤습니다. 비교한 건 딱 2개의 클라이언트였지만 공식으로 제공되는 Go 언어용 Client에서 아쉬웠던 부분들을 해결 할 수 있었습니다. 공식적인 Client는 아니기에 문제가 발생했을 때의 지원은 기대하기 어렵지만 현재도 활발하게 활동중인 프로젝트라 사용에 문제는 없을 것 같으며 공식 Client 역시 꾸준하게 업데이트 되고 있으므로 추후 더 많은 기능들을 제공받을 수 있을 것 같습니다.
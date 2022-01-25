---
layout: post
title:  "제안검색 조사"
description: "다나와는 현재 제안검색을 고려하고 있습니다. 오타로 인해 0건의 결과를 보여주지 않고, 사용자 의도를 파악하여 결과를 보여줌으로써 편의성을 높이기 위함입니다. 구글, 네이버에서는 어떻게 제안검색을 제공하는지와 제안검색 API 종류를 알아보도록 하겠습니다." 
date:   2020.09.09.
writer: "김준우"  
categories: Common 
---
## 소개

다나와는 현재 제안검색을 고려하고 있습니다. 오타로 인해 0건의 결과를 보여주지 않고, 사용자 의도를 파악하여 결과를 보여줌으로써 편의성을 높이기 위함입니다. 구글, 네이버에서는 어떻게 제안검색을 제공하는지와 제안검색 서비스 종류를 알아보도록 하겠습니다.

### 타사 제안검색

네이버, 구글에 각각 오타 키워드로 검색하여 원하는 키워드 결과로 출력되는지 확인해보겠습니다.


|의도한 키워드 | 오타 키워드 |
| --- | --- |
| 지오다노 | 지오다느 |
| 바비 인형 세트 | qkql dlsgud tpxm |


   
### 1. 검색 키워드: 지오다느 (지오다노)

네이버는 의도한 대로 지오다노의 결과가 출력되었습니다.

![/images/2020-09-09-Common-suggest-search/Untitled.png](/images/2020-09-09-Common-suggest-search/Untitled.png)

구글도 지오다노의 결과가 출력되었습니다.

![/images/2020-09-09-Common-suggest-search/Untitled%201.png](/images/2020-09-09-Common-suggest-search/Untitled%201.png)

### 검색 키워드: qkql dlsgud tpxm (바비 인형 세트)

네이버는 바비 인형 세트로 제안을 하였습니다.

![/images/2020-09-09-Common-suggest-search/Untitled%202.png](/images/2020-09-09-Common-suggest-search/Untitled%202.png)

구글도 의도한 바비 인형 세트로 결과가 나왔습니다.

![/images/2020-09-09-Common-suggest-search/Untitled%203.png](/images/2020-09-09-Common-suggest-search/Untitled%203.png)

### 테스트 정리

간단하게 제안검색을 알아보았습니다. 네이버와 구글에서는 오타가 발생하면 사용자의 의도로 결과를 보여주었습니다. 네이버는 2018년도에 AIQSpell 검색어 교정 시스템을 적용하여 한국어에 특화된 제안검색을 제공하고있습니다. 예를들어 "목포에서 군산까지 가는 법" 검색을 오타로 "목포에세 군산깢 가는버" 검색을 해도 의도한 결과를 확인 할 수 있습니다. 구글은 네이버와 비교해보았을때 결과에 조금더 정확한 모습을 보았습니다. 예를들어 "헤지스"를 원하지만 오타로 "해지스"로 검색을 하면 구글은 "헤지스"로 제안검색을 하는 모습을 확인 할 수 있었습니다.

## 제안 검색 서비스 종류

제안검색을 제공하는 서비스를 찾아보았습니다. 각각의 장점, 단점을 정리해보았습니다.

### 네이버 오타변환 API

링크: [https://developers.naver.com/docs/search/errata/](https://developers.naver.com/docs/search/errata/)

- 장점:
    - 한국어 특화된 오타변환을 제공합니다.
    - 비용없이 2만5천건의 요청을 제공합니다.
    - 네이버와 동일한 제안검색을 제공할 수 있습니다.
- 단점:
    - 상업적으로 사용불가합니다.
    - 2만5천건 이싱 필요시 제휴해야합니다.
    - 네이버 결과에 종속적입니다.

### 애저 Bing Spell Check API

링크: [https://azure.microsoft.com/ko-kr/services/cognitive-services/spell-check/](https://azure.microsoft.com/ko-kr/services/cognitive-services/spell-check/)

- 장점:
    - 모든 언어에 대해서 교정키워드를 제공합니다.
    - 분리단어, 브랜드 등 다양한 기능을 제공합니다.
- 단점:
    - TPS에 따른 비용이 발생합니다.

### Google Suggestqueries API

링크: [http://suggestqueries.google.com/complete/search?output=toolbar&q=](http://suggestqueries.google.com/complete/search?output=toolbar&q=%EB%91%90%EB%A3%A8%ED%82%B9**)다나가

- 장점:
    - 오타 키워드를 교정된 연관된 키워드 목록을 제공합니다.
- 단점:
    - 자동완성 키워드에 적합합니다.
    - 여러 키워드 중 제안검색 키워드 선별이 안됩니다.

### SerpAPI

[https://serpapi.com/spell-check](https://serpapi.com/spell-check)

구글을 크롤링하여 결과를 전달하는 서비스 중 하나 입니다.

- 장점:
    - json으로 결과를 받아 필요한 데이터만 사용가능합니다.
- 단점:
    - 구글은 크롤링을 허용하지 않았습니다.
    - 요청의 응답속도가 느립니다.

## 정리

제안검색에 알아보고 오타에 대해 변환하는 서비스를 조사해보았습니다. 현재 다나와에서는 PC와 모바일을 합쳐 평균 140만건의 검색이 발생합니다. 제안검색을 통해 사용자 의도한 결과를 보여주게 되면 불필요한 검색량을 줄일수도 있고, 사용자가 의도한 결과를 보여줌으로 편의성과 만족도는 향상될걸로 생각이됩니다.
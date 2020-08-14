---
layout: post
title:  "디서치 관리도구 사용법  #7 - 템플릿"
description: "디서치 관리도구 템플릿 메뉴에 대해 소개하고 사용하는 방법에 대해 설명 하도록 하겠습니다." 
date:   2020.08.14. 14:00:00
writer: "선지호"  
categories: Elastic 
---

## 소개

이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 템플릿 메뉴에 대해 소개하는 시간을 가지도록 하겠습니다.

## 템플릿 메뉴란?

엘라스틱서치에 관련된 인덱스 템플릿을 제공해 주는 메뉴 입니다.

이 메뉴에서 템플릿을 생성하면, 템플릿이 엘라스틱서치 클러스터에 저장됩니다.

이전 포스팅에서 다루었던 컬렉션을 생성하게 되면 자동으로 컬렉션 인덱스에 관련된 템플릿이 생성됩니다.

## 템플릿 메뉴 사용법 

### 처음 화면

템플릿 메뉴를 클릭 했을 때의 화면입니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/1.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/1.png)

### 템플릿 생성 

컬렉션을 생성해 보겠습니다.

매핑 설정은 이렇게 넣었습니다.

```jsx
{
    "item": {
        "type": "keyword"
    },
    "hit": {
        "type": "double"
    },
    "YN": {
        "type": "text"
    },
    "keyword": {
        "type": "keyword"
    }
}
```
셋팅 설정은 이렇게 넣었습니다.

```jsx
{
    "number_of_shards": "1",
    "number_of_replicas": "0",
    "refresh_interval": "-1"
}
```
![/images/2020-08-14-DSearch-Management-Tool-Usage-template/2.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/2.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/3.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/3.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/4.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/4.png)

### 템플릿 조회

템플릿이 생성 되었다면, 해당 템플릿을 클릭하여 템플릿을 조회 할 수 있습니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/5.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/5.png)

이곳에서는 템플릿에 대한 매핑 및 셋팅 정보를 볼 수 있습니다.
방금전에 생성한 템플릿은 아래와 같이 나옵니다.

#### 매핑 정보
![/images/2020-08-14-DSearch-Management-Tool-Usage-template/6.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/6.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/7.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/7.png)

#### 셋팅 정보
![/images/2020-08-14-DSearch-Management-Tool-Usage-template/8.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/8.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/9.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/9.png)

### 템플릿 수정

템플릿을 생성 하였으나 수정이 필요할 때, 템플릿 수정버튼을 클릭해 주시면 아래와 같은 화면으로 나오게 됩니다.

기본적으로 폼은 json 을 확인하는 용도로 쓰이며, 주로 json을 입력하여 사용하게 됩니다.

json 으로 입력된 수정 사항이 곧바로 적용됩니다.

#### 매핑

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/10.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/10.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/10-2.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/10-2.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/10-1.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/10-1.png)

#### 셋팅

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/11.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/11.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/11-2.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/11-2.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/11-1.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/11-1.png)

#### 템플릿 삭제

사용하지 않는 템플릿을 삭제하려면 템플릿 수정메뉴에 들어가 템플릿 삭제를 클릭해주시면 됩니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-template/12.png](/images/2020-08-14-DSearch-Management-Tool-Usage-template/12.png)

## 정리

이번에는 다나와에서 개발한 디서치 관리도구-템플릿 메뉴에 대해 설명하고, 간단하게 사용하는 포스팅을 했습니다.

디서치를 한번도 안 써본 개발자의 입장이 되어 최대한 자세하게 작성해 보았습니다.

이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
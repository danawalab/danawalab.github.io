---
layout: post
title:  "디서치 관리도구 사용법  #4 - 랭킹튜닝"
description: "디서치 관리도구 랭킹튜닝 대해 소개하고 사용하는 방법에 대해 설명 하도록 하겠습니다." 
date:   2020.08.13. 16:00:00
writer: "선지호"  
categories: Elastic 
---

## 소개

이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 랭킹튜닝 메뉴에 대해 소개하는 시간을 가지도록 하겠습니다.

## 랭킹튜닝 메뉴란?

인덱스에 쿼리를 전송해 가져온 정보를 한눈에 보기 쉽도록 표로 정리한 메뉴입니다.
인덱스 하나하나 쿼리를 전송해 볼 수 있고, 또는 여러 인덱스를 입력해서 쿼리를 전송해 볼 수 있습니다.

## 랭킹튜닝 메뉴 사용법 

### 처음 화면

처음 메뉴를 눌렀을 때 나오는 화면입니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/1.png](/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/1.png)


### 인덱스 입력

인덱스는 총 두가지 방식으로 입력 할 수 있습니다.

첫번째 방식은 조회되어서 나오는 인덱스를 클릭하여 입력하는 방식입니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/2.png](/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/2.png)

두번째 방식은 직접 사용자가 여러개의 인덱스를 조합하여 넣는 방식입니다.
이 방식은 각 인덱스마다 ','로 구분되어야 하며, 만약 정확하게 입력하지 않았다면 올바른 결과가 나오지 않습니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/3.png](/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/3.png)


### 쿼리 입력

쿼리가 정상적으로 입력된다면 아래와 같이 나옵니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/4.png](/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/4.png)

그러나 쿼리가 정상적으로 입력되지 않았다면, 오류메시지를 보여줍니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/5.png](/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/5.png)

### 점수 펼치기 / 접기

쿼리로 나온 결과에 점수(explain)가 있다면, 점수 펼치기 및 접기를 통해 간편하게 모든 점수들을 펼쳐보고 접을 수 있습니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/6.png](/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/6.png)

![/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/7.png](/images/2020-08-13-DSearch-Management-Tool-Usage-ranking-tuning/7.png)


## 정리

이번에는 다나와에서 개발한 디서치 관리도구-랭킹튜닝 메뉴에 대해 설명하고, 사용하는 방법에 대하여 포스팅 해보았습니다.

디서치를 한번도 안 써본 개발자의 입장이 되어 작성해 보았습니다.

이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
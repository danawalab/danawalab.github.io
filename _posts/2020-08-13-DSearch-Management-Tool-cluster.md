---
layout: post
title:  "디서치 관리도구 사용법  #5 - 클러스터"
description: "디서치 관리도구 클러스터 메뉴에 대해 소개하고 사용하는 방법에 대해 설명 하도록 하겠습니다." 
date:   2020.08.13.
writer: "선지호"  
categories: Elastic 
---

## 소개

이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 클러스터 메뉴에 대해 소개하는 시간을 가지도록 하겠습니다.

## 클러스터 메뉴란?

디서치에 등록된 엘라스틱서치 클러스터를 한눈에 볼 수 있게 시각화 한 메뉴입니다.

각종 인덱스 및 샤드, 레플리카에 대한 정보 또한 볼 수 있습니다.

## 클러스터 메뉴 사용법 

### 처음 화면

처음 메뉴를 눌렀을 때 나오는 화면입니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/1.png](/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/1.png)

### 총 사용량

이 곳은 현재 이 엘라스틱서치 클러스터가 몇개의 노드로 구성되어 있고, 인덱스는 몇개이며, 샤드 및 문서, 그리고 용량에 대해 알려주는 탭 입니다.

밑의 이미지에서는 10개의 노드로 현재 48개의 인덱스, 문서수는 약 3천만건, 용량은 17.5G 입니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/2.png](/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/2.png)

### 샤드 배치 

상단에 X축으로는 인덱스, 좌측에 Y축으로 엘라스틱서치 노드로 이루어진 표 입니다.

X축과 Y축이 일치하는 부분에는 프라이머리샤드 및 레플리카샤드가 들어갈 수 있는데, 각각 선과, 실선으로 테두리가 되어 있습니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/3.png](/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/3.png)



인덱스가 많기 때문에, 인덱스 필터로 원하는 인덱스 명을 입력하여 찾을 수 있습니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/4.png](/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/4.png)



특수 인덱스 체크박스를 클릭하면, 특수 인덱스 별로 볼 수 있습니다.

아래의 사진은 특수 인덱스 중 인덱스명에 kibana를 가지고 있는 인덱스를 찾은 화면입니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/5.png](/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/5.png)



닫힌 인덱스 체크박스를 클릭하면, 닫힌 인덱스 또한 찾아 볼 수 있습니다.

아래의 사진은 닫힌 인덱스가 없어 찾지 못한 화면과 닫힌 인덱스가 있는 화면 입니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/6.png](/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/6.png)

![/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/7.png](/images/2020-08-13-DSearch-Management-Tool-Usage-cluster/7.png)


## 정리

이번에는 다나와에서 개발한 디서치 관리도구-클러스터 메뉴에 대해 설명하고, 소개하는 포스팅을 했습니다.

디서치를 한번도 안 써본 개발자의 입장이 되어 최대한 자세하게 작성해 보았습니다.

이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
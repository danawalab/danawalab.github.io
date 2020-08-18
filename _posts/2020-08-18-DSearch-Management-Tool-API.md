---
layout: post
title:  "디서치 관리도구 사용법 #12 - API"
description: "안녕하세요. 이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 API메뉴에 대해 소개하도록 하겠습니다. API 메뉴는 엘라스틱의 _cat API의 요청을 UI로 쉽게 요청하고, 보기 편한 형태로 결과를 확인하는 메뉴입니다.  " 
date:   2020.08.18. 17:00:00
writer: "김준우"  
categories: Elastic 
---
## 소개

안녕하세요. 이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 API메뉴에 대해 소개하도록 하겠습니다. API 메뉴는 엘라스틱의 _cat API의 요청을 UI로 쉽게 요청하고, 보기 편한 형태로 결과를 확인하는 메뉴입니다.  

## API 메뉴 사용법

### 초기화면

초기화면은 master API를 호출한 모습으로 보여지게 됩니다. 이미지에서는 마스터 노드가 1개로 표시되고있고, 테이블 UI로 정리된 모습으로 결과를 확인 가능합니다.

![/images/2020-08-18-DSearch-Management-Tool-API/Untitled.png](/images/2020-08-18-DSearch-Management-Tool-API/Untitled.png)

### API  목록

엘라스틱서치에서 지원하는 allocation, shards, master, nodes, tasks 등 모든 _cat API를 지원하고있습니다.

![/images/2020-08-18-DSearch-Management-Tool-API/Untitled%201.png](/images/2020-08-18-DSearch-Management-Tool-API/Untitled%201.png)

## 정리

이번에는 다나와에서 개발한 디서치 관리도구-API 메뉴에 대해 설명하고, 사용하는 방법에 대하여 포스팅 해보았습니다. 엘라스틱서치에서 지원하는 _cat API를 쉽게 요청할 수 있고, 결과를 테이블 형태로 확인이 가능합니다.

디서치를 한번도 안 써본 개발자의 입장이 되어 작성해 보았습니다.

이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
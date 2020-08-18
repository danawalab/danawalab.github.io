---
layout: post
title:  "디서치 관리도구 사용법 #10 - 래퍼런스UI"
description: "안녕하세요. 이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 레퍼런스UI 메뉴에 대해 소개하도록 하겠습니다. " 
date:   2020.08.18. 15:00:00
writer: "김준우"  
categories: Elastic 
---
## 소개

안녕하세요. 이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 레퍼런스UI 메뉴에 대해 소개하도록 하겠습니다. 

## 래퍼런스UI 메뉴란?

레퍼런스UI는 검색 쿼리를 미리 구성하여, 키워드만 입력하면 구성한 쿼리가 전부 검색이 하는 기능입니다. 레퍼런스UI 메뉴에 엘라스틱서치에 검색할 쿼리 관리합니다. 키워드입력은 대시보드 상단에 검색을 하게 되면 결과가 보여지게 됩니다. 

## 레퍼런스UI 메뉴 사용법

### 초기화면

래퍼런스UI 메뉴 초기화면입니다. 상단의 검색 입력란에 보여질 자동완성 URL을 입력공간이 있고, 아래에 래퍼런스UI 영역이 있습니다.

![/images/2020-08-18-DSearch-Management-Tool-ReferenceUI/Untitled.png](/images/2020-08-18-DSearch-Management-Tool-ReferenceUI/Untitled.png)

### 자동완성 URL

자동완성 ULR을 입력한뒤 등록버튼을 누르면 자동완성을 사용할 수 있습니다. 자동완성 URL은 엘라스틱이 지원하는 QueryString 기반으로 작성하여도 되지만, 직접 서버를 개발하여 url을 입력하여도 됩니다. 결과를 array 형식입니다. 아래 이미지 처럼 p= 로 끝나게 되면 키워드를 입력할때마다 p=<키워드>로 자동입력되어 호출하게 됩니다.

![/images/2020-08-18-DSearch-Management-Tool-ReferenceUI/Untitled%201.png](/images/2020-08-18-DSearch-Management-Tool-ReferenceUI/Untitled%201.png)

### 영역추가

영역추가 버튼을 누르게 되면 엘라스틱에 검색할 쿼리를 입력할 수 있는 입력란이 생성이 됩니다. 영역을 추가 할때마다 검색결과가 탭별로 영역이 보여지게 됩니다.    

영역이름: 영역의 이름을 입력합니다.   

인덱스: _search 시 사용될 인덱스명을 입력합니다.   

검색쿼리: _search의 requestBody영역의 QueryDsl을 입력합니다. `*키워드는 ${keyword} 변수를 사용합니다.`   

제목, 클릭URL, 썸네일, 정보: 검색 결과의 문서필드를 입력합니다.  

어그리게이션: 검색 결과의 어그리게이션 필드를 입력합니다.   

![/images/2020-08-18-DSearch-Management-Tool-ReferenceUI/Untitled%202.png](/images/2020-08-18-DSearch-Management-Tool-ReferenceUI/Untitled%202.png)

### 검색 결과

상단의 마스크팩이라는 키워드를 입력한 후 엔터를 누르게 되면 결과 페이지로 이동하면서 래퍼런스 영역이  탭으로 결과를 보여지게 됩니다. 설정버튼을 누르면 래퍼런스UI 화면으로 이동하게 됩니다. 

![/images/2020-08-18-DSearch-Management-Tool-ReferenceUI/Untitled%203.png](/images/2020-08-18-DSearch-Management-Tool-ReferenceUI/Untitled%203.png)

## 정리

래퍼런스UI를 통해 미리 검색쿼리작성하여 검색을 즉시 확인 해볼 수 있는 좋은 기능입니다.

이번에는 다나와에서 개발한 디서치 관리도구-래퍼런스UI 메뉴에 대해 설명하고, 사용하는 포스팅을 했습니다.

디서치를 한번도 안 써본 개발자의 입장이 되어 최대한 자세하게 작성해 보았습니다.

이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
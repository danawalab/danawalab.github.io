---
layout: post
title:  "디서치 관리도구 사용법  #2 - 사전"
description: "디서치 관리도구 사전에 대해 소개하고 사용하는 방법에 대해 설명 하도록 하겠습니다." 
date:   2020.08.13. 13:00:00
writer: "선지호"  
categories: Elastic 
---

## 소개

이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 사전 메뉴에 대해 소개하는 시간을 가지도록 하겠습니다.

## 사전 메뉴란?

디서치에서는 기본적으로 사용자 사전을 지원합니다. 
지원하는 사용자 사전을 조금더 간편하게 볼수 있게, 또는 간편하게 사용 및 검색 할 수 있게 도와주는 메뉴입니다.

## 사전 메뉴 사용법 

### 처음 화면 

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/1.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/1.png)

### 검색 

검색을 하게 되면 아래와 같이 값이 나오게 됩니다.

단어의 형태소 분석결과 : 값
각 사전 이름 : 검색된 결과 

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/2.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/2.png)

### 개요 탭 

개요 탭에서는 각 사전에 대한 작업 단어 갯수 및 적용된 단어 갯수, 토큰 타입 등등을 한눈에 볼 수 있습니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/3.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/3.png)

사전에 적용하기 위해서는 왼쪽의 체크버튼을 클릭하고 사전적용 버튼을 클릭합니다 (체크버튼이 체크되지 않는다면 버튼이 활성화 되지 않습니다)

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/4.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/4.png)

수정사항이 있다면 아래와 같이 적용 시간과 적용 단어 갯수가 변경되게 됩니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/5.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/5.png)

### 각 사전 탭

#### 사전 탭 UI
![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/6.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/6.png)

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/6-1.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/6-1.png)

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/6-1.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/6-2.png)

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/6-1.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/6-3.png)


#### 단어매칭 

클릭 전
![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/7.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/7.png)


단어 매칭 기준이 되는 단어가 똑같을 때 찾아집니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/8.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/8.png)

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/9.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/9.png)

#### 다운로드 

다운로드를 클릭하게 되면 사전을 텍스트 파일로 다운 받을 수 있게 됩니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/11.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/11.png)

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/12.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/12.png)

#### 수정 클릭 시 변화 

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/10.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/10.png)

이렇게 직접 수정 가능합니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/13.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/13.png)

#### 추가 버튼 클릭시 

데이터를 새로 등록할 수 있습니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/14.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/14.png)

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/14-1.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/14-1.png)

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/14-2.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/14-2.png)

#### 삭제 버튼 클릭시 (왼쪽의 체크박스 체크 필요)

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/15.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/15.png)

삭제 된 이후 

![/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/16.png](/images/2020-08-13-DSearch-Management-Tool-Usage-dictionary/16.png)

## 정리

이번에는 다나와에서 개발한 디서치 관리도구-사전 메뉴에 대해 설명하고, 사용하는 방법에 대하여 포스팅 해보았습니다.
디서치를 한번도 안 써본 개발자의 입장이 되어 작성해 보았습니다.
이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
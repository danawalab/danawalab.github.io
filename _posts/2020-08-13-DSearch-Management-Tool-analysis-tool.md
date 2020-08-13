---
layout: post
title:  "디서치 관리도구 사용법  #3 - 분석도구"
description: "디서치 관리도구 분석도구 메뉴에 대해 소개하고 사용하는 방법에 대해 설명 하도록 하겠습니다." 
date:   2020.08.13. 
writer: "선지호"  
categories: Elastic 
---

## 소개

이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 분석도구 메뉴에 대해 소개하는 시간을 가지도록 하겠습니다.

## 분석도구 메뉴란?

인덱스에서 사용하는 분석기를 이용하여 입력한 내용에 대해 테스트를 할 수 있는 메뉴입니다.
각각 간략과 상세로 나누어져 있으며, 상세 탭은 디서치에서 제공하는 플러그인을 설치해야 사용이 가능합니다.

## 분석도구 메뉴 사용법 

1. 처음 화면 

처음 메뉴를 눌렀을 때 나오는 화면입니다.

만약 인덱스가 없다면 아래와 같이 나옵니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/1.png](/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/1.png)

만약 인덱스가 있다면 아래와 같이 나오게 됩니다.

![/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/2.png](/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/2.png)

2. 간략 탭

간략탭에서는 간단하게 분석기가 어떤식으로 입력에 대해 다루고 있는지 알 수 있는 탭입니다.

- 분석기 선택

![/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/3.png](/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/3.png)

- 분석 버튼 클릭

![/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/4.png](/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/4.png)

3. 상세 탭

상세탭에서는 디서치 플러그인 분석기가 어떤식으로 입력에 대해 다루고 있는지 알 수 있는 탭입니다.
모델명 규칙, 단위명규칙, 형태소 분리 결과, 동의어 확장, 복합명사, 최종결과 로 총 6개의 단락이 있으며, 이를 통해 사용자에게 조금 더 편리한 분석기능을 제공합니다.

- 분석기 선택

![/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/5.png](/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/5.png)

- 분석 버튼 클릭

![/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/6.png](/images/2020-08-13-DSearch-Management-Tool-Usage-analysis-tool/6.png)

## 정리

이번에는 다나와에서 개발한 디서치 관리도구-분석도구 메뉴에 대해 설명하고, 사용하는 방법에 대하여 포스팅 해보았습니다.
디서치를 한번도 안 써본 개발자의 입장이 되어 작성해 보았습니다.
이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
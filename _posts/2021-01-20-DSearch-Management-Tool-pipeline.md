---
layout: post
title:  "디서치 관리도구 사용법  #14 - 파이프라인"
description: "안녕하세요. 이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 파이프라인 메뉴에 대해 소개하는 시간을 가지도록 하겠습니다." 
date:   2021.01.20.
writer: "선지호"  
categories: Elastic 
---

## 소개

안녕하세요. 이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 파이프라인 메뉴에 대해 소개하는 시간을 가지도록 하겠습니다.

## 파이프라인 메뉴란?

엘라스틱서치에 관련된 파이프라인을 생성, 삭제, 수정 및 테스트를 할 수 있게 제공해 주는 메뉴 입니다.

이 메뉴에서 파이프라인을 생성하면, 파이프라인이 엘라스틱서치 클러스터에 저장됩니다.

이전 포스팅에서 다루었던 컬렉션의 수집소스에서 파이프라인을 설정할 수 있기 때문에, 색인을 진행할 때 이 메뉴에서 생성한 파이프라인을 지정해 줄 수 있습니다.

## 파이프라인 메뉴 사용법 

### 처음 화면

파이프라인 메뉴를 클릭 했을 때의 화면입니다.

이미 생성되어 있거나, 생성한 파이프라인을 볼 수 있습니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/main.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/main.png)

### 파이프라인 생성 

첫 클릭시의 화면 입니다.
기본적으로 자주 사용되는 파이프라인의 프로세서들이 템플릿으로 입력되어 있습니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/create1.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/create1.png)

여기서 저는 html 태그를 제거하는 파이프라인을 만들어 보겠습니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/create2.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/create2.png)

위와 같이 입력하고 생성 버튼을 누르면, 아래의 이미지와 같이 정상적으로 생성되었음을 볼 수 있습니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/create3.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/create3.png)

### 파이프라인 테스트 (1)

만든 파이프라인을 한번 테스트 해보겠습니다.

먼저, 오른쪽에 있는 파이프라인-테스트 탭을 클릭하여 메뉴를 바꾸어 줍니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-1.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-1.png)

그리고 테스트할 파이프라인을 선택하여 바꾸어줍니다

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-2.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-2.png)

이후 기본적으로 셋팅되어 있는 테스트를 할 내용을 수정해 줍니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-3.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-3.png)

마지막으로 하단에 있는 테스트 버튼을 눌러 테스트를 진행합니다. 의도했던대로 html 태그가 제거가 된 내용이 나온 것을 확인 할 수 있습니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-4.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-4.png)

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-5.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/test1-5.png)

### 파이프라인 수정

이번에는 생성한 파이프라인을 수정해 보겠습니다.

수정 버튼을 눌러 파이프라인의 프로세서를 수정해봅니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/modify1.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/modify1.png)

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/modify2.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/modify2.png)

이번에는 field 안의 내용을 전부 소문자로 만든 후 html 태그를 제거해 보겠습니다.

### 파이프라인 테스트 (2)

다시 파이프라인-테스트 탭을 클릭해서 접근해 줍니다.

아까전에 테스트를 진행 했던대로, 테스트 할 파이프라인을 선택하고, 아까와 같은 테스트 할 내용을 입력해 줍니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/test2-1.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/test2-1.png)

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/test2-2.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/test2-2.png)

그리고 아래 테스트 버튼을 눌러 테스트를 진행하면 이번에는 저희가 의도했던대로 html 태그는 없어지면서 내용이 소문자로 나올수 있게 변환 했습니다.

이제 자세히를 클릭해서 다시 테스트를 진행해 봅니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/test2-3.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/test2-3.png)

각 프로세스별 단계에 따른 결과를 볼 수 있습니다.

### 파이프라인 삭제

파이프라인을 삭제할 때는 다시 파이프라인 탭으로 되돌아와 해당 파이프라인의 삭제를 눌러줍니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/delete1.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/delete1.png)

정상적으로 삭제되었음을 확인 할 수 있습니다.

![/images/2021-01-20-Dsearch-Management-Tool-pipeline/delete2.png](/images/2021-01-20-Dsearch-Management-Tool-pipeline/delete2.png)

## 정리

이번에는 다나와에서 개발한 디서치 관리도구-파이프라인 메뉴에 대해 설명하고, 사용하는 포스팅을 했습니다.

디서치를 한번도 안 써본 개발자의 입장이 되어 최대한 자세하게 작성해 보았습니다.

이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.

감사합니다.
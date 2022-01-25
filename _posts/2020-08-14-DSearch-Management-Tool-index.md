---
layout: post
title:  "디서치 관리도구 사용법  #8 - 인덱스"
description: "디서치 관리도구 인덱스 메뉴에 대해 소개하고 사용하는 방법에 대해 설명 하도록 하겠습니다." 
date:   2020.08.14.
writer: "선지호"  
categories: Elastic 
---

## 소개

이번 포스팅 내용은 다나와에서 개발한 디서치 관리도구(엘라스틱서치 관리도구)에서 인덱스 메뉴에 대해 소개하고 사용하는 방법에 대해 설명하는 시간을 가지도록 하겠습니다.

## 인덱스 메뉴란?

엘라스틱서치에 생성된 인덱스에 관련된 기능을 제공해 주는 메뉴 입니다.

이 메뉴에서 인덱스를 생성할 수는 없지만, 인덱스 안의 내용을 조회하거나 수정하거나 인덱스를 삭제하거나 하는 기능 등등 여러 기능이 있습니다.


## 인덱스 메뉴 사용법 



### 처음 화면

인덱스 메뉴를 클릭 했을 때의 화면입니다. 인덱스의 상태(열린 인덱스 / 닫힌 인덱스)를 이곳에서 간단히 확인할 수 있습니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/1.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/1.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/2.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/2.png)

### 인덱스 조회

인덱스를 조회하면, 5개의 탭과 관리 버튼을 보실 수 있습니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/3.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/3.png)




#### 관리 버튼

관리버튼은 6개의 기능으로 이루어져 있는데, 각각 인덱스 열기/닫기, 강제 머징, 리프레쉬, 플러시, 동결/동결해제, 삭제 기능으로 되어있습니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/4.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/4.png)

- 인덱스 열기/닫기 

엘라스틱서치에서는 인덱스를 열고 닫을 수 있습니다. 더이상 쓰이지 않는 인덱스는 서버의 성능을 고려하여 인덱스를 닫거나 삭제를 해주는 것이 좋은데, 이를 위해 디서치에서는 이 기능을 제공하고 있습니다. 닫힌 인덱스는 인덱스의 내용을 읽거나 쓰는 것이 불가능해 집니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/5.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/5.png)

- 강제 머징

엘라스틱서치에서는 디스크에 세그먼트 형태로 데이터를 저장합니다.
이 세그먼트는 새로운 도큐먼트 혹은 레코드를 인덱싱하거나 도큐먼트 삭제시에 세그먼트를 만드는데, 이 세그먼트의 갯수가 증가한다면 서버에 악영향을 끼칠수 있기때문에 엘라스틱서치에서는 강제 머징 기능을 제공합니다.
디서치에서도 이 기능을 간편히 사용할 수 있게 기능을 제공하고 있습니다.

- 리프레쉬

인덱스를 강제로 새로고침 합니다. 강제로 새로고침 하지 않는다면, 최근 인덱싱된 도큐먼트는 일정시간 후에 검색이 되게 됩니다.
이를 피하기 위해 엘라스틱서치에서는 새로고침 하는 기능이 있습니다.
디서치에서도 이 기능을 간편히 사용할 수 있게 기능을 제공하고 있습니다.

- 플러시

엘라스틱서치에서는 성능상의 이유로 특정 데이터를 메모리와 트랜잭션 로그에 저장합니다. 
노드 종료시 스테일 데이터 방지 및 모든 데이터를 안전한 상태로 유지하기 위해 엘라스틱서치에서는 강제 플러시 기능을 제공하고 있습니다. 디서치에서도 이 기능을 간편히 사용할 수 있게 기능을 제공하고 있습니다.

- 동결/동결해제

엘라스틱서치에서는 인덱스를 읽기 전용으로 설정할 수 있습니다. 
그것이 바로 인덱스 동결/동결해제 기능인데, 디서치에서 이 기능을 간편히 사용할 수 있게 기능을 제공하고 있습니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/6.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/6.png)

- 삭제

현재 인덱스를 삭제하는 기능입니다.








#### 개요

현재 인덱스의 상태 및 인덱스에 관련된 내용들 (사용중인 스토리지 용량, 삭제 문서 갯수, 현재 문서 갯수 등등)을 한눈에 볼 수 있게 한 탭입니다.




#### 매핑

현재 인덱스의 매핑 정보를 볼 수 있습니다. 
폼과 Json 라디오 버튼의 클릭으로 으로 볼 수 있는 형식을 바꿀 수 있습니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/7.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/7.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/8.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/8.png)




#### 셋팅

현재 인덱스의 셋팅 정보를 볼 수 있습니다. 
폼과 Json 라디오 버튼의 클릭으로 으로 볼 수 있는 형식을 바꿀 수 있습니다.
동적변경 버튼을 통해 셋팅 정보를 json 형식으로 재 입력해주시면 동적으로 현재 인덱스의 셋팅 정보를 변경시킬수 있습니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/9.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/9.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/10.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/10.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/11.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/11.png)




#### 통계

현재 인덱스의 통계 데이터를 볼 수 있습니다.
위에서 설명했던 인덱스 열기/닫기, 플러시 등 이 기능들을 몇번 사용했는지 여부와 문서의 갯수 같은 인덱스에 관련된 모든 통계정보를 제공합니다.

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/12.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/12.png)




#### 데이터

현재 인덱스에 저장된 데이터를 간단히 생성하고 삭제하고 읽고 변경하고를 가능하게 해주는 탭 입니다.

기본 / 분석된 색인어  두가지 형태로 제공하고 있으며, 데이터에 대한 생성, 변경 및 삭제는 기본에서만 제공됩니다.




#### 데이터 - 기본

이곳에서는 데이터 생성 / 삭제 / 변경 등의 기능을 제공하고 있습니다.

- 데이터 생성

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/13.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/13.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/14.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/14.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/15.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/15.png)

- 데이터 삭제

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/16.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/16.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/17.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/17.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/18.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/18.png)

- 데이터 변경

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/19.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/19.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/20.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/20.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/21.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/21.png)

- 한줄에 보여지는 문서 수 변경

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/22.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/22.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/23.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/23.png)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/24.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/24.png)

- 검색 

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/25.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/25.png)




#### 데이터 - 분석된 색인어

이 곳은 데이터에 대해 분석을 한번 더 진행하여 어떤식으로 데이터가 쪼개지는지를 볼 수 있습니다.

아이디 - 필드명
값 - 원본 데이터
분석결과 - 분석된 결과

(*가 되어있는 아이디 밑 필드들은 전부 같은 문서 입니다.)

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/26.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/26.png)

- 검색 방법

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/27.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/27.png)

- 한 페이지에 보여지는 문서 수 변경

![/images/2020-08-14-DSearch-Management-Tool-Usage-index/28.png](/images/2020-08-14-DSearch-Management-Tool-Usage-index/28.png)





## 정리

이번에는 다나와에서 개발한 디서치 관리도구-인덱스 메뉴에 대해 설명하고, 사용하는 포스팅을 했습니다.

디서치를 한번도 안 써본 개발자의 입장이 되어 최대한 자세하게 작성해 보았습니다.

이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
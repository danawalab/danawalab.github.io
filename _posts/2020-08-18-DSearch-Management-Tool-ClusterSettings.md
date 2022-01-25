---
layout: post
title:  "디서치 관리도구 사용법 #13 - 클러스터설정"
description: "안녕하세요, 이번 포스팅에서는 클러스터설정 메뉴에 대해 소개하도록 하겠습니다. 클러스터설정에서는 엘라스틱의 클러스터 모든 설정을 확인할 수 있습니다. 클러스터 설정에는 기본설정, 영구설정, 임시설정이 있습니다. 각 설정별로 분류하여 쉽게 확인이 가능 하도록 하였습니다." 
date:   2020.08.18.
writer: "김준우"  
categories: Elastic 
---
## 소개

안녕하세요, 이번 포스팅에서는 클러스터설정 메뉴에 대해 소개하도록 하겠습니다. 클러스터설정에서는 엘라스틱의 클러스터 모든 설정을 확인할 수 있습니다. 클러스터 설정에는 기본설정, 영구설정, 임시설정이 있습니다. 각 설정별로 분류하여 쉽게 확인이 가능 하도록 하였습니다.

## 클러스터설정 메뉴 사용법

### 초기화면

초기화면에서는 기본값이 보여지게 됩니다. action, bootstrap, cache, ccr 등 설정들을 그룹화하여 보여주고 있습니다.

![/images/2020-08-18-DSearch-Management-Tool-ClusterSettings/Untitled.png](/images/2020-08-18-DSearch-Management-Tool-ClusterSettings/Untitled.png)

### 영구설정

엘라스틱서치의 클러스터 영구설정은 기본적으로 아무런 값이 없기 때문에 max_bytes_per_sec 설정을 50mb 로 설정해보았습니다.

![/images/2020-08-18-DSearch-Management-Tool-ClusterSettings/Untitled%201.png](/images/2020-08-18-DSearch-Management-Tool-ClusterSettings/Untitled%201.png)

### 임시설정

엘라스틱서치의 임시설정은 아무런 값이 없기 때문에 indices.recovery.max_bytes_per_sec 설정을 20mb로 추가 하여 캡쳐하였습니다. 

![/images/2020-08-18-DSearch-Management-Tool-ClusterSettings/Untitled%202.png](/images/2020-08-18-DSearch-Management-Tool-ClusterSettings/Untitled%202.png)

## 정리

이번에는 다나와에서 개발한 디서치 관리도구-클러스터설정 메뉴에 대해 설명하고, 사용하는 방법에 대하여 포스팅 해보았습니다. 클러스터설정은 수정,삭제 기능은 없지만 키값을 그룹별로 나누어두어 보기 쉽게 하였습니다.

디서치를 한번도 안 써본 개발자의 입장이 되어 작성해 보았습니다.

이 포스팅이 나중에 사용하시는 분들께 많은 도움이 되었으면 좋겠습니다.
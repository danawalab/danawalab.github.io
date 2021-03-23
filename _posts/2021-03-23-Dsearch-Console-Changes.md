---
layout: post
title:  "디서치 관리도구 변경사항 - 20210323"
description: DSearch 콘솔과 서버에 기능이 추가 되었습니다.
date:   2021.03.23.
writer: "김준우"
categories: Dsearch
---

## 개요
DSearch 콘솔과 서버에 기능이 추가 되었습니다.

- 클러스터 화면에서 사전 소스 정보를 선택이 가능해졌습니다. 개발, 운영 등 멀티 엘라스틱서치 클러스터를 관리시 사전 데이터를 매번 동기화하기 번거러움을 해소 할 수 있습니다. 같은 DSearch 서버에 등록된 엘라스틱서치면 사전을 제한없이 공유할 수 있습니다.



## 변경사항

### 1. 원격 사전 

사전 소스 클러스터 선택)

![/images/2021-03-23-Dsearch-Console-Changes/2021-03-23_17h55_50.png](/images/2021-03-23-Dsearch-Console-Changes/2021-03-23_17h55_50.png)

사전 소스 확인)

![/images/2021-03-23-Dsearch-Console-Changes/2021-03-23_17h46_48.png](/images/2021-03-23-Dsearch-Console-Changes/2021-03-23_17h46_48.png)

사전에 표시된 사전 소스 정보)

![/images/2021-03-23-Dsearch-Console-Changes/2021-03-23_17h49_05.png](/images/2021-03-23-Dsearch-Console-Changes/2021-03-23_17h49_05.png)


클러스터 메뉴에서 사전 소스 클러스터를 선택하면 사전 정보가 자동으로 변경이 되고, 사전 적용 버튼을 클릭 시 원격지의 사전 데이터를 조회하여 분석기에서 사용됩니다. 사전 데이터를 수정하게 되면 원격지의 데이터가 변경됩니다.

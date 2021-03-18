---
layout: post
title:  "디서치 관리도구 변경사항 - 1"
description: 디서치 관리도구 변경사항에 대해 포스팅합니다
date:   2021.03.18.
writer: "선지호"
categories: Dsearch
---

## 개요
- 디서치 관리도구 변경사항에 대해 알려드립니다.

1. ES 클러스터별 관리도구 페이지 - 팝업창 제거
2. 분석 #사전 - 사전 순서 변경 기능 추가
3. 분석 #사전 - 사전 추가 시 엔터 타이핑으로 추가 및 해당 창에서 여러번 추가 가능하게 변경
4. 분석 #사전 - 최근 등록한 사전 순서로 노출
5. 분석 #사전 - 사전 추가 시 자동으로 추가된 키워드 입력 동작 제거
6. 분석 #사전 - 사전 수정 시 적용 버튼과 삭제버튼 여백 추가
7. 분석 #분석도구 - 상세 및 쿼리용도 체크 디폴트 처리
8. 분석 #분석도구 - 분석 내용 엔터 타이핑 시 분석 


## 변경사항

### 1. 클러스터별 관리도구 페이지 - 팝업창 제거
![/images/2021-03-18-Dsearch-Console-Changes/1.png](/images/2021-03-18-Dsearch-Console-Changes/1.png)

기존에 노드 부분에 있던 팝업창을 띄울 수 있게 버튼이 있었는데, 이를 제거했습니다.

### 2. 사전 관리도구 순서 변경 기능 추가
![/images/2021-03-18-Dsearch-Console-Changes/2-1.png](/images/2021-03-18-Dsearch-Console-Changes/2-1.png)
![/images/2021-03-18-Dsearch-Console-Changes/2-2.png](/images/2021-03-18-Dsearch-Console-Changes/2-2.png)
![/images/2021-03-18-Dsearch-Console-Changes/2-3.png](/images/2021-03-18-Dsearch-Console-Changes/2-3.png)
![/images/2021-03-18-Dsearch-Console-Changes/2-4.png](/images/2021-03-18-Dsearch-Console-Changes/2-4.png)
![/images/2021-03-18-Dsearch-Console-Changes/2-5.png](/images/2021-03-18-Dsearch-Console-Changes/2-5.png)

사전 탭을 눌렀을 때 나오는 사전 리스트의 순서를 변경 가능 하게 했습니다.

### 3. 사전 추가 시 엔터 타이핑으로 추가 및 해당 창에서 여러번 추가 가능하게 변경
![/images/2021-03-18-Dsearch-Console-Changes/3-1.png](/images/2021-03-18-Dsearch-Console-Changes/3-1.png)
![/images/2021-03-18-Dsearch-Console-Changes/3-2.png](/images/2021-03-18-Dsearch-Console-Changes/3-2.png)
![/images/2021-03-18-Dsearch-Console-Changes/3-3.png](/images/2021-03-18-Dsearch-Console-Changes/3-3.png)

사전에 대한 내용을 추가할때 기존에는 추가 했을 때 추가창이 사라졌지만, 현재는 변경하여 남아 있을 수 있게 바꾸었습니다.

### 4. 최근 등록한 사전 순서로 노출
![/images/2021-03-18-Dsearch-Console-Changes/4.png](/images/2021-03-18-Dsearch-Console-Changes/4.png)

기존에는 정렬순서가 먼저 등록한 순서 였지만, 최근에 등록한 사전 순서로 노출 시킬수 있도록 변경하였습니다.

### 5. 사전 추가 시 자동으로 추가된 키워드 입력 동작 제거
![/images/2021-03-18-Dsearch-Console-Changes/5.png](/images/2021-03-18-Dsearch-Console-Changes/5.png)

기존에는 사전에 데이터를 추가 하게되면 추가 창이 사라지고,
사전에 정상적으로 등록되었다는 것을 나타내기 위해 자동으로 추가된 키워드를 입력 후 조회 해서 보여주었습니다.
하지만 이 동작을 삭제하고 전체 사전에 대해 나타나게 변경 했습니다.

### 6. 사전 수정 시 적용 버튼과 삭제버튼 여백 추가
![/images/2021-03-18-Dsearch-Console-Changes/6-1.png](/images/2021-03-18-Dsearch-Console-Changes/6-1.png)
![/images/2021-03-18-Dsearch-Console-Changes/6-2.png](/images/2021-03-18-Dsearch-Console-Changes/6-2.png)

사전 데이터 수정 시 적용버튼과 삭제버튼이 가까이 있어, 두 버튼 사이에 여백을 추가 했습니다.

### 7. 분석도구 상세 및 쿼리용도 체크 디폴트
![/images/2021-03-18-Dsearch-Console-Changes/7.png](/images/2021-03-18-Dsearch-Console-Changes/7.png)

분석도구에 대해 상세분석과 쿼리용도가 더 많이 사용되어 상세분석 및 쿼리용도 체크를 디폴트로 변경했습니다.

### 8. 분석도구 내용에 여러 줄 입력 제거 및 엔터 타이핑 시 분석처리 
![/images/2021-03-18-Dsearch-Console-Changes/8.png](/images/2021-03-18-Dsearch-Console-Changes/8.png)

기존에는 분석도구 내용을 입력한 뒤에 버튼을 클릭해야만 분석이 되었다면, 이제는 엔터를 눌렀을때도 분석이 되게 변경했습니다.
또한 기존에는 엔터로 개행문자까지 포함을 시켜 분석을 했다면 개행문자 입력을 입력을 제거 하였습니다.

---
layout: post
title:  "디서치 관리도구 변경사항 - 1"
description: 디서치 관리도구 변경사항에 대해 포스팅합니다
date:   2021.03.18.
writer: "선지호"
categories: Dsearch
---

## 개요
디서치 관리도구 변경사항에 대해 알려드립니다.

- 클러스터 관리 - 팝업창 버튼 제거
- 분석 #사전 - 엔터키 입력으로 사전 추가
- 분석 #사전 - 아이콘 드래그로 사전 순서 변경 기능 추가
- 분석 #사전 - 최근 등록된 사전 데이터 순서로 정렬 변경
- 분석 #사전 - 사전 추가 시 추가된 키워드 조회 기능 제거
- 분석 #사전 - 사전 수정 창 버튼 간 공백 추가
- 분석 #분석도구 - 디폴트로 상세 및 쿼리용도 체크 셋팅
- 분석 #분석도구 - 분석도구에서 엔터 타이핑 시 분석 가능
- 분석 #사전 - 사전 데이터 추가/삭제 버튼 공백 추가
- 분석 #사전 - 사전 등록 창 '취소' -> '닫기'로 명칭 변경
- 분석 #사전 - 사전 데이터 삭제 버튼 클릭 시 선택이 되어 있지 않다면 무시
- 분석 #사전 - 사전 데이터 추가 시 팝업창에서 등록 텍스트 추가


## 변경사항

### 1. 팝업창 버튼 제거
![/images/2021-03-18-Dsearch-Console-Changes/1-2.png](/images/2021-03-18-Dsearch-Console-Changes/1-2.png)

![/images/2021-03-18-Dsearch-Console-Changes/1.png](/images/2021-03-18-Dsearch-Console-Changes/1.png)

기존에 노드 부분에 있던 팝업창을 띄울 수 있게 버튼이 있었는데, 이 버튼이 사라졌습니다.

### 2. 아이콘 드래그로 사전 순서 변경 기능 추가 
![/images/2021-03-18-Dsearch-Console-Changes/2-1.png](/images/2021-03-18-Dsearch-Console-Changes/2-1.png)

![/images/2021-03-18-Dsearch-Console-Changes/2-2.png](/images/2021-03-18-Dsearch-Console-Changes/2-2.png)

![/images/2021-03-18-Dsearch-Console-Changes/2-3.png](/images/2021-03-18-Dsearch-Console-Changes/2-3.png)

![/images/2021-03-18-Dsearch-Console-Changes/2-4.png](/images/2021-03-18-Dsearch-Console-Changes/2-4.png)

![/images/2021-03-18-Dsearch-Console-Changes/2-5.png](/images/2021-03-18-Dsearch-Console-Changes/2-5.png)

기존에는 사전 탭의 사전리스트 순서를 변경할 수 없었습니다.

이제는 사전 탭을 눌렀을 때 나오는 사전 리스트의 순서를 변경 하실 수 있습니다.

### 3. 엔터키 입력으로 사전 추가
![/images/2021-03-18-Dsearch-Console-Changes/3-1.png](/images/2021-03-18-Dsearch-Console-Changes/3-1.png) => ![/images/2021-03-18-Dsearch-Console-Changes/3-2.png](/images/2021-03-18-Dsearch-Console-Changes/3-2.png)

![/images/2021-03-18-Dsearch-Console-Changes/3-3.png](/images/2021-03-18-Dsearch-Console-Changes/3-3.png)

기존에는 사전에 대한 내용을 추가할때 추가창이 사라졌고, 엔터로 사전 내용 추가가 불가능 했습니다.

하지만 이제는 추가 창이 사라지지 않고, 엔터 입력으로 사전 내용을 추가 하실 수 있습니다

### 4. 최근 등록된 사전 데이터 순서로 정렬 변경
![/images/2021-03-18-Dsearch-Console-Changes/4.png](/images/2021-03-18-Dsearch-Console-Changes/4.png)

최근 등록된 사전 데이터 순서로 변경됩니다.

### 5. 사전 추가 시 추가된 키워드 조회 기능 제거
![/images/2021-03-18-Dsearch-Console-Changes/5.png](/images/2021-03-18-Dsearch-Console-Changes/5.png)

기존에는 사전에 데이터를 추가 하게되면 추가 창이 사라지고, 사전에 정상적으로 등록되었다는 것을 나타내기 위해 자동으로 추가된 키워드를 입력 후 조회 해서 보여주었습니다.

하지만 이제는 사전 추가시 추가 창이 사라지지 않고 추가된 키워드를 조회해서 보여주지 않습니다.

### 6. 사전 수정 창 버튼 간 공백 추가
![/images/2021-03-18-Dsearch-Console-Changes/6-1.png](/images/2021-03-18-Dsearch-Console-Changes/6-1.png)

![/images/2021-03-18-Dsearch-Console-Changes/6-2.png](/images/2021-03-18-Dsearch-Console-Changes/6-2.png)

사전 데이터 수정 시 적용버튼과 삭제버튼, 두 버튼 사이에 공백이 추가 됩니다.

### 7. 디폴트로 상세 및 쿼리용도 체크 셋팅
![/images/2021-03-18-Dsearch-Console-Changes/7.png](/images/2021-03-18-Dsearch-Console-Changes/7.png)

분석도구에 대해 상세분석과 쿼리용도가 더 많이 사용되어 상세분석 및 쿼리용도 체크가 디폴트 설정이 되었습니다.

### 8. 분석도구에서 엔터 타이핑 시 분석 가능
![/images/2021-03-18-Dsearch-Console-Changes/8.png](/images/2021-03-18-Dsearch-Console-Changes/8.png)

기존에는 분석도구 내용을 입력한 뒤에 버튼을 클릭해야만 분석이 되었고, 분석 내용을 입력할 때 엔터 키 입력이 가능했습니다.

하지만 변경 이후에는 분석 내용을 작성한 뒤 엔터키 입력으로 분석이 가능합니다.

### 9. 사전 데이터 추가/삭제 버튼 공백 추가
![/images/2021-03-18-Dsearch-Console-Changes/9-1.png](/images/2021-03-18-Dsearch-Console-Changes/9-1.png)

![/images/2021-03-18-Dsearch-Console-Changes/9-2.png](/images/2021-03-18-Dsearch-Console-Changes/9-2.png)

사전 데이터 추가/삭제 버튼 사이에 공백이 생깁니다.

### 10. 사전 등록 창 '취소' -> '닫기'로 명칭 변경
![/images/2021-03-18-Dsearch-Console-Changes/10-1.png](/images/2021-03-18-Dsearch-Console-Changes/10-1.png)

![/images/2021-03-18-Dsearch-Console-Changes/10-2.png](/images/2021-03-18-Dsearch-Console-Changes/10-2.png)

기존에는 사전 데이터 등록창에 추가/취소 버튼으로 되어있었습니다.

변경된 이후에는 추가/닫기 버튼으로 명칭을 변경하여 텍스트가 표시됩니다.

### 11. 사전 데이터 삭제 버튼 클릭 시 선택이 되어 있지 않다면 무시
![/images/2021-03-18-Dsearch-Console-Changes/11.png](/images/2021-03-18-Dsearch-Console-Changes/11.png)

기존에는 사전 데이터 삭제 버튼 클릭시 아무런 선택이 없었더라도 삭제 팝업창이 오픈 되었습니다.

변경된 이후에는 사전 데이터 삭제 선택이 없다면 삭제 팝업창을 오픈하지 않습니다.

### 12. 사전 데이터 추가 시 팝업창에서 등록 텍스트 추가
![/images/2021-03-18-Dsearch-Console-Changes/12.png](/images/2021-03-18-Dsearch-Console-Changes/12.png)

사전 데이터 추가 시 등록 되었다는 텍스트가 표시 됩니다.


# 다나와 기술블로그 


## 카테고리 등록방법

1. _category 디렉터리 하위에 md 파일 생성합니다.

2. 생성한 md 파일 내용 입력합니다.
```

---
layout: category # 기본적으로 사용될 레이아웃입니다.
title: java      # 카테고리 구분및 제목입니다.
---

```

3. github으로 commit 하면 자동 등록됩니다. post가 하위로 있을때만 카테고리가 보여집니다.


## 포스트 등록방법

1. _posts 디렉터리 하위에 YYYY-MM-DD-title.md 구문으로 md 파일생성합니다.

2. 상단에 아래 내용을 필수로 입력 후 다음 라인부터 내용을 작성하시면 됩니다.
```
---
layout: post
title:  "제목"  # 리스트에 보여질 제목
description: "설명" # 리스트에 보여질 설명
image: "https://cdn.pixabay.com/photo/2015/11/02/18/34/banner-1018818_960_720.jpg" # [optional] 배너 이미지를 바꾸고 싶을때 입력하면됩니다. 없을시 기본 배너 표시됩니다.
date:   2020-01-23 10:53:24 +0900 # 등록일자를 입력하세요.
writer: "작성자"  # 작성자명을 입력하세요.
categories: java # 생성한 카테고리 title을 입력하세요.
---


내용....


```

github에 commit 후 잠시 기다리면 적용된 모습을 확인할 수 있습니다.




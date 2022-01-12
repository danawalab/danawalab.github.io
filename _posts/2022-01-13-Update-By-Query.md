---
layout: post
title:  "Elasticsearch Update_By_Query 적용기"
description: Elasticsearch에서 제공하는 Update_By_Query의 개념과 적용 방법을 살펴보고
적용시 주의해야할 사항과 성능에 대해 알아봅니다.
date:   2022-01-13
writer: "반윤성"
categories: Elastic
---

![/images/2022-01-11-Update-by-query/image_1.jpg](/images/2022-01-11-Update-by-query/image_1.jpg)

## 소개 : Update_By_Query ?
Elasticsearch(이하 es)에서 기존에 색인된 내용을 변경하고자 할 때, 'Update_By_Query' 기능을 사용할 수 있습니다. 이 기능은 단순히 업데이트를 수행하는것 뿐만 아니라 쿼리를 통한 질의 후에 해당하는 조건에 맞는 필드를 탐색하여 업데이트를 진행합니다.

취급하는 데이터의 양이 많다보니 일괄적으로 값을 변경해주는 기능이 필요했고, 기존 데이터 업데이트시 bulk api를 사용하고 있었지만 id를 통해 업데이트되는 방식이었습니다. 그래서 es에서 제공하는 Update_By_Query기능을 사용하게 되었습니다.
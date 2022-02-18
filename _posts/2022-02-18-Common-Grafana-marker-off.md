---
layout: post
title:  "그라파나 게이지의 Threshold 마커"
description: "그라파나에서 많이 사용하는 게이지의 Threshold 마커특성과 헷갈릴 경우 대처방법을 알아봅니다."
date:   2022.02.18.
writer: "송상욱"
categories: Common
---

## 그라파나 게이지의 Threshold



다나와에서는 그라파나를 사용하여 시스템을 모니터링하고 있습니다. 그중에서도 검색엔진의 동적색인 소요시간을 모니터링하여 상품 데이터의 변화를 빠르게 추적하고 있습니다. 그리고 동적색인의 소요시간 데이터는 Gauge를 사용해서 보여주고 있습니다.

그런데, 어떤 때에는 아래처럼 게이지 테두리가 전체적으로 녹색일 때가 있고 비슷한 수치인데도 테두리가 빨간색으로 보일때가 있습니다. 수치는 비슷한데 말이죠. 

![Untitled](/images/2022-02-18-Common-Grafana-marker-off/threshold-red.png)
![Untitled](/images/2022-02-18-Common-Grafana-marker-off/threshold-green.png)


그 이유는 게이지의 특성때문입니다. 테두리는 Threshold의 마킹을 의미합니다. 그리고 최근의 최대값을 기반으로 그려지기 때문에, 최대값이 크면 클수록 빨간 테두리는 커질수 있습니다.

그렇기 때문에, 현재수치는 작아도 테두리는 빨간색으로 보인다면 자칫 시스템에 문제가 있는 것으로 오해할수 있습니다. 이 최대값 수치가 얼마나 유지되는지는 문서상으로 찾지는 못했는데요, 경험상 1시간이상은 유지되었습니다.

실제 수치는 안쪽에 있는 도넛모양의 Bar에 그려집니다.

만약 이 테두리 때문에 값이 헷갈리는 경우는 테두리를 안보이게 할수가 있습니다. 패널 속성을 열어서 Show threshol markers(마커보기)를 끄면 됩니다.

![Untitled](/images/2022-02-18-Common-Grafana-marker-off/setting.png)


이제 아래와 같이 테두리가 사라지고 실제 수치만 볼수 있게 되었습니다.

![Untitled](/images/2022-02-18-Common-Grafana-marker-off/done.png)


게이지를 볼때 색깔이 달라지는게 궁금해서 구글링을 했지만 내용을 찾을 수 없어서 직접 원인을 찾게되었습니다. 그라파나를 사용하면서 비슷한 고민을 가진 분들에게 도움이 되었으면 좋겠습니다. 


참고자료:

- https://grafana.com/docs/grafana/latest/visualizations/gauge-panel/
  

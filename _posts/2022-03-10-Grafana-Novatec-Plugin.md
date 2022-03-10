---
layout: post
title: "Grafana Novatec Service Dependency Graph Panel 사용법"
description: "그라파나 노바택 서비스 디펜던시 그래프 패널 사용법에 대해 알아볼 예정입니다."
date: 2022.03.10.
writer: "장민규"
categories: Common
---

## 소개

그라파나 패널 플러그인 Novatec Service Dependency Graph Panel이라 있습니다.  
위 패널의 사용법을 알려드리고자 합니다.  
Novatec Service Dependency Graph Panel를 앞으로 노바택 패널이라 부르겠습니다.

### 환경

- 그라파나 7.1.5
- 노바택 패널 4.0.3

**노바택 패널 4.0.0 버전 이상은 그라파나 7.1.0 버전 이상부터 지원합니다.**

패널 설치는 두 가지 방법이 있습니다.

- grafana cli를 통해 설치
- 플러그인을 직접 받아 grafana 불륨에 직접 추가하는 방법입니다.

두 방식 중 하나를 선택해 노바택 패널을 설치하고 그라파나를 재부팅 하고 나면  
Visualization을 보시면 아래 사진과 같이 Service Dependency Graph Panel가 나오는 것을 볼 수 있습니다.  
![노바택패널1](/images/2022-03-10-Grafana-Novatec-Plugin/1.PNG)

먼저 패널을 그리기 위해 request 매트릭 데이터를 수집하는 쿼리를 작성해 보겠습니다.  
**여기서 제일 중요한 게 있습니다, 아래 사진을 보시면 Format 옆을 보면 Table로 설정되어 있습니다.  
초기 설정은 Time series로 설정돼있는데 Table로 설정을 변경하지 않으면 노바택 패널이 매트릭 데이터를 제대로 못 읽어
패널을 그리지 못합니다.**  
![노바택패널3](/images/2022-03-10-Grafana-Novatec-Plugin/6.PNG)

그러면 아래의 사진과 같이 매트릭 데이터를 불러오게 됩니다.
아래 사진에 job과 url 그리고 Value #B가 나온 것을 볼 수 있습니다.  
![노바택패널3](/images/2022-03-10-Grafana-Novatec-Plugin/7.PNG)

패널을 그리기 위해 노바택 패널에 Connection Mapping을 확인해 보면 6개의 칸을 볼 수 있습니다.  
저는 job에서 url로 requset가 가는 것을 그리기 위해 Component Column에 job을 Target Component Column에 url을 매핑해 줬습니다.  
![노바택패널4](/images/2022-03-10-Grafana-Novatec-Plugin/2.PNG)

그러면 아래와 같이 그려지는 것을 볼 수 있습니다.  
![노바택패널5](/images/2022-03-10-Grafana-Novatec-Plugin/10.PNG)

다음에 관계도를 그려주기 위해 노바택 패널에 Data Mapping에
Request Rate Column에 Value #B를 매핑해 주겠습니다.  
![노바택패널6](/images/2022-03-10-Grafana-Novatec-Plugin/3.PNG)

아래 사진과 같이 관계도가 그려지는 것을 볼 수 있으며
오른쪽에 5개 버튼이 있는데 첫 번째는 request가 요청되는 것을 시각적으로 보여줍니다.  
**(실제 request를 라이브로 보여주는게 아닌 매트릭 데이터 값을 기반으로 시각적인 효과만 있습니다.)**  
두 번째는 관계도를 자동으로 배열해 줍니다.
세 번째는 패널을 한눈에 들어오게 확대해주고 네 번째 다섯 번째는 확대 축소 입니다.  
![노바택패널7](/images/2022-03-10-Grafana-Novatec-Plugin/8.PNG)  
첫 번째 버튼을 작동했을 경우  
![노바택패널8](/images/2022-03-10-Grafana-Novatec-Plugin/9.PNG)

위 사진처럼 원 안과 테두리에 초록색과 빨간색을 볼 수 있는데  
Data Mapping에 Response Time Baseline (Upper) 한계치를 설정하고 아래 사진과 같이 Show Baselines 켜주시면  
Response Time Health를 확인 할 수 있습니다.  
![노바택패널9](/images/2022-03-10-Grafana-Novatec-Plugin/4.PNG)

## 마무리

노바택 패널은 모니터링을 더욱더 시각적으로 보여주는 장점이 있습니다.  
그러나 수집된 메트릭 데이터가 많고 관계도가 복잡해지면 패널을 그리는데 초기에 시간이 좀 걸립니다.  
그리고 request가 요청을 실시간으로 보여주는 게 아니라 수집된 매트릭 데이터 기반으로 시각적으로 보여주는 게 아쉬운 점입니다.

또한 grafana 8버전 이후부터 Node graph panel이라고 노바택 패널과 비슷하게
서비스 관계도를 그려주고 더 많은 기능을 제공하는 패널이 베타 버전으로 출시했습니다.  
Node graph panel은 아직 베타 버전이라 aws에 x-ray를 통해서 사용이 가능해 당장 사용하기에는 제한적이라 생각합니다.

참고자료:

- https://github.com/NovatecConsulting/novatec-service-dependency-graph-panel
- https://grafana.com/grafana/plugins/novatec-sdg-panel/

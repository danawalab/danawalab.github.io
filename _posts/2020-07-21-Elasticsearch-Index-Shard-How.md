---
layout: post
title:  "엘라스틱서치 인덱스와 샤드 분할"
description: "엘라스틱서치의 인덱스와 샤드를 어떻게 나누었는지 그 경험을 나누겠습니다"
date:   2020.07.21.
writer: "송상욱"
categories: Elastic
---
![/images/2020-07-21-Elasticsearch-Index-Shard-How/fence-1280.png](/images/2020-07-21-Elasticsearch-Index-Shard-How/fence-1280.png)

## 시작하며

엘라스틱서치로 대용량 인덱스를 구성하다보면 몇가지 고민이 생깁니다. 하나의 인덱스를 구성하여 샤드를 수십개로 나눌것인지 아니면, 인덱스 자체를 여러개로 쪼갤지를 선택해야 합니다. 물론 선택의 기준은 성능이겠지요. 여기서는 다나와에서 엘라스틱서치의 인덱스와 샤드를 어떻게 나누었는지 그 경험을 나누겠습니다.



## 여러개의 샤드 vs 인덱스 쪼개기

다나와 서비스의 상품갯수는 급격하게 늘어나고 있습니다. 몇달전 4~5억건이었던 상품이 7억건에 육박하고 있는데요. 현재 사용하고 있는 패스트캣 검색엔진에서는 대량의 문서들을 여러개의 인덱스로 나누어 색인하고 있습니다. 검색할때는 나눈 인덱스들을 하나로 합쳐서 검색하고 있는데요. 엘라스틱서치에도 여러 인덱스를 하나로 합쳐서 한번에 검색하는 기능이 있어서 지금처럼 인덱스를 쪼개서 사용하는 것도 가능한 시나리오입니다. 참고로 얘기하면, 이렇게 데이터를 범위나 카테고리로 나누어 관리하는 기법을 파티셔닝이라고 합니다.

먼저 인덱스를 하나로 가져가든 여러개로 나누든, 샤드 하나의 크기는 비슷하게 설정해야 할텐데요, 적절한 샤드의 크기를 먼저 정해야 할 것 같습니다. 몇차례 구글링을 한결과 엘라스틱서치의 공식블로그에서 도움이 될만한 글를 찾을수 있었습니다.

> TIP: 작은 샤드는 작은 세그먼트를 만들며 부하를 증가시킵니다. 평균 샤드 크기를 최소한 수 GB와 수십 GB 사이를 유지하세요. 시간 기반 데이터를 사용한 과거 사례를 보면, 20GB ~ 40GB 정도의 사이즈가 적당합니다.

샤드크기는 수GB에서 수십GB 사이가 적당하며, 경험상 시계열 데이터의 경우 20GB~40GB가 적당하다고 합니다. 그리고 아래에서 또다른 팁을 발견할수 있었습니다.

> TIP: 하나의 노드에 저장할 수 있는 샤드의 개수는 가용한 힙의 크기와 비례하지만, Elasticsearch에서 그 크기를 제한하고 있지는 않습니다. 경험상 하나의 노드에 설정한 힙 1GB 당 20개 정도가 적당합니다. 따라서 30GB 힙을 가진 노드는 최대 600개 정도의 샤드를 가지는 것이 가능하지만, 이 보다는 적게 유지하는 것이 더 좋습니다.

힙메모리 30GB로 엘라스틱서치를 구동시 샤드는 최대 600개 정도라고 합니다. 이미 32GB 를 힙에 할당할 생각을 하고 있으므로, 활성화된 인덱스가 50개라고 한다면, 인덱스당 `600샤드 / 50 인덱스 = 12` 로 인덱스당 최대 12개의 샤드를 설정할수 있을 겁니다. 일단 샤드갯수의 최대치는 넉넉한것 같으므로, 많아서 문제가 될것같지는 않습니다. 

하나의 샤드를 20GB~40GB로 설정한다면, 7억건의 상품은 몇개의 샤드로 나눠지게 될까요?  다나와 상품기준으로는 15개의 샤드로 나눠지게 됩니다. 물론 엘라스틱 서치 블로그에서는 시계열 데이터에 대한 팁이라서 상품과는 문서의 특성이 다릅니다. 대부분은 웹서버나 DB 로그데이터 일텐데요, 로그스태시의 파싱모듈을 확인해보면 한 로그내의 필드수가 10개 내외로 매우 적습니다. 반면에 상품의 필드는 수십개는 기본입니다. 이러한 문서특성의 차이로 검색과 색인시 1개 문서에 들어가는 리소스와 시간이 더 투여됩니다. 그러므로, 상품문서는 하나의 샤드를 20GB가 아닌 조금 더 작게 나누면 여러 서버에 더 잘게 분산이 되어 전체적인 응답시간은 더 빨라질겁니다. 우리는 샤드당 5GB로 설정하고 테스트 했는데, 빠른 응답속도를 얻을 수 있었습니다.

그럼, 인덱스를 나누는것과 샤드를 나누는것 어떤것이 적합할까요? 어치피 샤드는 여러 서버에 분산되어 병렬 및 병행 (참고: [https://soy.me/2015/01/03/concurrent/](https://soy.me/2015/01/03/concurrent/))으로 검색이 되므로, 인덱스가 같던 다르던 상관은 없습니다. 검색의 기본 단위는 샤드이기 때문이죠. 따라서 인덱스를 나누는 것은 운영의 편의성을 고려할때 선택하는 방법입니다. 장애없는 운영의 측면에서 생각해보면, 큰 덩어리 하나를 다루는 것은 부담스러워도, 작은 덩어리 여러개를 다루는 것이 번거롭긴해도 그만큼 장애 가능성을 낮추는 방법이 됩니다.

## 전체색인을 빠르게 하려면

전체색인을 할 경우 인덱스 하나가 7억건이라면 색인이 모두 끝날때까지 약 3-4시간이 소요될것이고, 검색에 노출되기 까지 이시간을 고스란히 기다려야 합니다. 더 문제가 되는것은 전체색인 도중에도 상품의 가격은 계속해서 변하게 되는데 이 변경분을 전체색인후에 일괄적용을 해야하며, 대기시간이 길수록 일괄적용시간도 늘어나게 됩니다. 그러므로, 최대한 전체색인을 빠르게 해야하는데, 이를 위해서는 문서를 입력하는 REST 클라이언트를 멀티스레드로 여러개 생성하여 동시입력량을 늘려야 합니다. 이때 Bulk Insert 를 쓰는것은 기본이구요. 하지만, 엘라스틱서치의 Write IO도 고려해야 합니다. 동시입력량을 늘리면, 서버의 CPU와 Disk IO중 둘중 하나는 최대치에 이를것이고, 더이상 색인속도는 늘어나지 않게 됩니다. 우리가 경험한 최대 색인속도는 500KB 상품문서 기준으로 16000건/초 였습니다. 이때 CPU는 32코어로 90%를 사용했으니 활용도는 매우 높다고 할수 있습니다. (참고: [https://danawalab.github.io/elastic/2020/07/06/Elasticsearch-Index.html](https://danawalab.github.io/elastic/2020/07/06/Elasticsearch-Index.html)) 결국은 더 빠른 색인을 위해서는 하나의 인덱스를 여러개로 나눠야 합니다. 통으로 4시간이 걸리는 문서를 10개의 인덱스로 나누면 색인시간이 서버를 공유하므로 정확히 1/10이 될수는 없지만, 그대로 각각 40분정도로 병행완료가 됩니다. 상품DB의 특성상 우리는 카테고리군별로 인덱스를 나누고 있습니다. 카테고리별로 인덱스를 나눌때의 장점은 특정카테고리만 검색할때 해당 인덱스만 검색하면 되므로, 검색부하가 현저히 감소하게 됩니다. 우리는 인덱스도 나누고 샤드도 2-3개로 나누어 전체적인 검색응답시간을 최대한 단축하는 방향으로 설계했습니다.

## 레플리카 갯수는 몇개로?

고가용성을 추구할때 빼놓을 수 없는 것이 레플리카인데요, 레플리카는 레플리카 샤드를 줄여서 얘기하는 것으로, 프라이머리 샤드에 종속됩니다. 우리말로는 복제본 이라고도 하죠. 복제본은 분산 데이터 시스템에서는 동일한 역할을 담당하는데요, 하드웨어 장애를 극복하고 검색과 같은 읽기처리량을 향상시키는 것입니다. 그런데 읽기성능이 좋아진다는 것은 쓰기성능이 낮아진다는 얘기도 됩니다. 왜냐하면, 복제본을 여러개 만들기 위해서는 그만큼 문서색인시에 쓰기작업도 복제본 갯수만큼 발생하기 때문이죠. 하나의 서버만 본다면 1이지만, 전체클러스터 입장에서는 복제본이 열개면 10의 쓰기가 일어나는 것입니다. 문서가 색인될때 구지 모든 서버의 IO발생하게 만들 필요는 없을겁니다. 또한 복제본의 갯수는 동시에 장애가 발생할 노드를 몇개 까지 허용하는지에 달려있습니다. 복제본이 1개라면 하나의 노드가 죽어도 검색서비스는 유지됩니다. 하지만, 이제 복제본은 0개가 되므로, SA가 빨리 노드를 복구하지 않으면, 다른 또하나의 노드가 죽을때 검색서비스에는 장애가 생깁니다. 만약 회사내에 장애 복구시스템이 잘 갖춰져 있다고 하면, 복제본은 1개로도 가능합니다. 복제본이 2라면 서버가 연속 2개 죽어도 상관이 없으므로, 마음을 느긋하게 가질수 있습니다. 하지만 전체 클러스터에서 서버가 2개 빠지고도 부하를 충분히 견딜수 있게 서버를 배치해 놓아야 합니다. 우리는 카테고리별 인덱스의 읽기성능을 고려해서 부하가 높은 인덱스는 복제본 2를 나머지는 1을 설정할 계획입니다.

## 결론

nginx로그와 같은 로그성 문서는 색인을 하고 나면 수정이 필요없는 정적인 컨텐츠인 반면에, 상품문서는 색인이 끝나도 계속 갱신이 되어야 하는 살아있는 컨텐츠입니다. 그렇기 때문에, 동적색인이 원활하고 장애도 대비할수 있으며, 검색성능도 높은 설계가 필요합니다. 엘라스틱서치를 사용하여 검색시스템을 설계할때 운영자가 할수 있는 최선의 방법은 문서와 샤드를 잘 관리하는 것입니다. 따라서 견고한 검색시스템을 위해 인덱스와 샤드를 어떻게 설정해야 하는지에 대해 살펴봤습니다. 

## 참고

[https://www.elastic.co/kr/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster](https://www.elastic.co/kr/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster)

[https://medium.com/kariyertech/elasticsearch-cluster-sizing-and-performance-tuning-42c7dd54de3](https://medium.com/kariyertech/elasticsearch-cluster-sizing-and-performance-tuning-42c7dd54de3c)

[https://www.elastic.co/guide/en/elasticsearch/reference/current/scalability.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/scalability.html)
---
layout: post
title:  "1G 메모리를 사용하여 1초만에 2TB 텍스트파일을 검색하기"
description: "디스크 기반 정렬을 통해 대용량 텍스트 파일의 내용을 검색하여 API로 제공하는 방법"
date:   2022.11.10.
writer: "반윤성"
categories: Common
---

![/images/2022-11-10-big-file-searcher/books.avif](/images/2022-11-10-big-file-searcher/books.avif)
<div>
  Photo by nadi borodina on unsplash
</div>

## 대용량 파일을 모두 뒤져서 검색할 때 문제점

업무를 하다보면 대용량 파일(Gb, Tb 단위)을 다룰 일이 종종 있습니다. 

보통 자신이 주로 사용하는 ``Notepad++``나 ``Vim``과 같은 문서 편집 도구를 이용해 큰 파일들을 불러와서 내용을 탐색하곤 합니다.

작은 파일이라면 순식간에 읽어서 원하는 내용을 검색할 수 있지만 큰 파일을 빈번하게 읽어야할 일이 생긴다면 그때마다 파일을 열어 탐색하기까지 기나긴 인고의 시간 (🙉) 을 보내야 할 지 모릅니다.


![/images/2022-11-10-big-file-searcher/error.png](/images/2022-11-10-big-file-searcher/error.png)
<div>
  아무래도 Notepad++가 감당하기엔 무리.
</div>
<br>

## 어떻게 하면 쉽고 빠르게 파일을 열어 검색할 수 있을까?

#### 1. 색인의 개념

우리가 사전에서 원하는 내용을 찾는다면, 보통 맨 뒷 페이지에 있는 ``색인``부터 살펴봅니다. 기호와 활자로 정렬된 일련의 목록에서 특정 위치를 기록하고 있기에 위치를 기준으로 검색합니다. 이렇기에 맨 첫장부터 원하는 단어가 나올 때까지 검색하는 비용을 아낄 수 있습니다.

기본적으로 컴퓨터에서 처음 큰 파일을 열때는 첫 줄부터 마지막 줄까지 읽어내립니다. 하지만 내가 찾는 키워드의 위치를 알고 있다면, 해당 포지션부터 읽을 수 있습니다. 단 색인📗이 존재해야겠지만요.

#### 2. 색인 파일 만들기

색인이 필요하다는 사실을 알았으니 그 방법을 살펴보겠습니다. 우선 큰 파일을 열어 찾고자 하는 단어의
위치를 기록합니다. 이 위치를 메모리에 바로 올리면 편하겠지만 그렇게 되면 비싸고 큰 메모리를 필요로 합니다.

이럴때는 디스크의 노는 공간을 조금 활용합니다. 찾고자하는 키워드와 위치가 기록된 인덱스 파일을 작성해 저장하는 것이지요. 예로들면 <상품ID>,<바이트 수>를 기록해놓았다가 검색할때 이 파일을 참조해서 원본 내용에서 필요한 만큼 읽어내는 식입니다.

다만 파일의 내용이 정렬된 데이터라는 보장이 없으므로 소팅이 필요합니다. 여기서 메모리에 단위 크기 만큼 올려 퀵 소트를 통해 부분적으로 정렬을 시도합니다. 약 10,000,000 라인의 파일이 있을때 1,000건, 2,000건, 4,000건, 8,000건, ... , 10,000,000건까지 청크 파일을 만들어 디스크 기반 정렬을 시도합니다.

```go
// 아주 간단한 퀵 소트 알고리즘, 실패시 최종 위치를 반환한다
func QuickSortList(a []model.Index) []model.Index {

	if len(a) < 2 {
		return a
	}

	left, right := 0, len(a)-1

	pivot := rand.Int() % len(a)

	a[pivot], a[right] = a[right], a[pivot]

	for i, _ := range a {
		if a[i].ProductId < a[right].ProductId {
			a[left], a[i] = a[i], a[left]
			left++
		}
	}

	a[left], a[right] = a[right], a[left]

	QuickSortList(a[:left])
	QuickSortList(a[left+1:])

	return a

}
```


이렇게 만들어진 색인 파일을 메모리에 곧바로 올려도 되지만, 메모리 크기에 따라 어떤 PC에서는 동작하고 또 다른 PC에서는 동작하지 않을 수 있습니다. 그래서 인덱스의 1/N만 가지고 있는 색인 파일을 다시 색인합니다.

최종적으로 우리가 메모리에 적재하여 검색에 사용할 내용은 인덱스 파일의 인덱스 파일이 되는 셈입니다.

![/images/2022-11-10-big-file-searcher/index_0.png](/images/2022-11-10-big-file-searcher/index_0.png)

![/images/2022-11-10-big-file-searcher/index_6.png](/images/2022-11-10-big-file-searcher/index_6.png)
<div>
  원본 상품 데이터 파일 (무려 60GB)
</div>

![/images/2022-11-10-big-file-searcher/index_1.png](/images/2022-11-10-big-file-searcher/index_4.png)
<div>
  상품 아이디 기준으로 색인한다
</div>

![/images/2022-11-10-big-file-searcher/index_1.png](/images/2022-11-10-big-file-searcher/index_5.png)
<div>
  최종적으로 메모리에 올릴 파일, 색인 파일을 한번 더 색인한다
</div>
<br>

#### 3. 검색하기

인덱스 파일에서 나온 경량화 인덱스 파일은 쉽게 말해 간격이 띄워진 색인 파일입니다. 이렇게 완전한 색인 파일이 아닌데도 검색할 수 있는 방법이 있습니다. 바로 이진 탐색(binary search)를 통해 검색을 진행하고 실패한 부분에서는 원래 인덱스 파일로 돌아가 간격만큼 읽어내는 방식입니다.

즉, 1,000개씩 띄워진 색인 파일이 있을때 실패한 지점에서 1,000개 만큼 읽어내면 간격사이의 데이터를 찾을 수 있습니다. 이를 통해 전체가 아닌 부분적으로 탐색하므로 적은 자원으로 __빠른__ 검색이 가능합니다.  ⚡

```go
// 아주 간단한 바이너리 서치 알고리즘
func BinarySearch(array []model.MiniIndex, target string) (int, bool) {
	var isExist bool

	r := -1 // not found
	start := 0
	end := len(array) - 1
	for start <= end {
		mid := (start + end) / 2
		if array[mid].ProductId == target {
			r = mid // found
			break
		} else if array[mid].ProductId < target {
			start = mid + 1
		} else if array[mid].ProductId > target {
			end = mid - 1
		}
	}

	if r == -1 {
		if end == -1 {
			r = 0
		} else {
			// 없을 경우 최종적으로 읽은 부분을 리턴한다
			r = end
		}
	} else {
		isExist = true
	}

	// 찾은 인덱스를 리턴한다
	return r, isExist
}
```


![/images/2022-11-10-big-file-searcher/search_0.png](/images/2022-11-10-big-file-searcher/search_0.png)
<br>

## 파일 서처 서비스
![/images/2022-11-10-big-file-searcher/service_1.png](/images/2022-11-10-big-file-searcher/service_1.png)

현재 서비스 중인 파일 서처의 구조는 다음과 같습니다. 수집된 상품 데이터에 대해 디렉토리 감지를 통해 해당 파일에 대해 색인하여 메모리에 적재합니다. 검색 API를 통해 요청이 들어오면, 색인된 내용을 기반으로 탐색하고 요청된 데이터를 반환합니다.

데이터는 설정을 통해 파싱하는 방식, 색인 간격, 적재 데이터 초기화 스케줄등을 설정할 수 있으며 Go 1.18을 사용해 개발되었습니다.

## 파일 서처 서비스 API 가이드

`GET /search`

해당 업체 코드와 상품ID로 원본 파일의 내용을 조회합니다.

#### Request Parameter
|parameter|description|data type|example value
|:---:|:---:|:---:|:---:|
|shopCode|업체 코드|string|TH201, ED302 , ..|
|productId|상품 아이디|string|2074042,7658077,2186943|
|startDate|날짜 From|string (YYYYMMDD)|20221110|
|endDate|날짜 To|string (YYYYMMDD)|20221111|
|renew|전체/갱신 여부|string|1 또는 2|

#### Return Value
- result : 검색 처리 결과 (success or fail)
- date : 검색 일자
- renew : 전체/갱신 여부
- data : 검색 내용
- timestamp : 해당 데이터 일자
- source : 원본 데이터 내용


```
GET /search?shopCode=EE301&productId=2073143136&startDate=20220326&&endDate=20220329

    {
        result: success,
        date: 20220926,
        renew : 1,
        data : [
            {
                timestamp: 1664171131
                source :    "2073143136^이미용|화장품|메이크업(여성용)|볼터치/하이라이터^[현대백화점] [삼성카드7%할인~08/22]아워글래스 앰비언트 블러쉬 +무이자3개월^(주) 신세계인터네셔날^http://image.thehyundai.com/static/3/1/3/14/73/2073143136_0_600.jpg^http://www.thehyundai.com/front/pda/itemPtc.thd?       
                         ReferCode=030&utm_source=danawa&utm_medium=ep_price&slitmCd=2073143136^60000^0^^0개월^^^^^0^삼성카드^55800^^60000^N^^N^Y"
            },
            {
                timestamp : 1661929118
                source :    "2073143136^이미용|화장품|메이크업(여성용)|볼터치/하이라이터^[현대백화점] [삼성카드7%할인~08/22]아워글래스 앰비언트 블러쉬 +무이자3개월^(주) 신세계인터네셔날^http://image.thehyundai.com/static/3/1/3/14/73/2073143136_0_600.jpg^http://www.thehyundai.com/front/pda/itemPtc.thd?       
                         ReferCode=030&utm_source=danawa&utm_medium=ep_price&slitmCd=2073143136^60000^0^^0개월^^^^^0^삼성카드^45800^^50000^N^^N^Y"
            }
        ]
    },
    {
        result : success,
        date : 20220925,
        renew : 1,
        data : [
            {
                timestamp : 1664171131
                source :    "2073143136^이미용|화장품|메이크업(여성용)|볼터치/하이라이터^[현대백화점] [삼성카드7%할인~08/22]아워글래스 앰비언트 블러쉬 +무이자3개월^(주) 신세계인터네셔날^http://image.thehyundai.com/static/3/1/3/14/73/2073143136_0_600.jpg^http://www.thehyundai.com/front/pda/itemPtc.thd?       
                         ReferCode=030&utm_source=danawa&utm_medium=ep_price&slitmCd=2073143136^60000^0^^0개월^^^^^0^삼성카드^55800^^60000^N^^N^Y"
            },
            {
                timestamp : 1661929118
                source :    "2073143136^이미용|화장품|메이크업(여성용)|볼터치/하이라이터^[현대백화점] [삼성카드7%할인~08/22]아워글래스 앰비언트 블러쉬 +무이자3개월^(주) 신세계인터네셔날^http://image.thehyundai.com/static/3/1/3/14/73/2073143136_0_600.jpg^http://www.thehyundai.com/front/pda/itemPtc.thd?       
                         ReferCode=030&utm_source=danawa&utm_medium=ep_price&slitmCd=2073143136^60000^0^^0개월^^^^^0^삼성카드^45800^^50000^N^^N^Y"
            }
        ]
    }
```

## 파일 서처 서비스 성능
|서버|수집된 EP 파일 크기|실 메모리 사용량|원본 대비 색인 효율|검색 속도
|:---:|:---:|:---:|:---:|:---:|
|server1|607G|296M|0.048%|0.959초
|server2|757G|431M|0.056%|1.003초
|server3|569G|419M|0.073%|0.8초
|Total|1.9TB|1,029M|0.059%(평균)|0.9초(평균)

색인화 과정을 통해 원본EP 파일의 인덱스 파일을 메모리에 올려서 사용하고 있으며, 원본 파일 대비 약 0.05%의 색인 효율을 보이고 있습니다.

약 __2T__ 데이터에 대해 검색 속도는 평균 __1초__ 이하의 검색결과를 제공하고 있습니다.

## 정리
큰 파일을 읽는것은 자원과 시간이 많이 소요되는 일이므로 가능한 파일 서처와 같은 검색 도구를 활용하여 검색하는 것이 바람직하다고 생각됩니다. 또한 개발 후 테스트를 통해 데이터 검증을 마쳤으므로 다양한 문서 포맷에 대해 색인/검색 가능 여부를 확인하는 것이 중요하다고 생각합니다.

Github link :  <https://github.com/danawalab/file-searcher/>

## 참고 자료
- https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html
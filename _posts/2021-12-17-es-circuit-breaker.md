---
layout: post
title:  "elasticsearch Circuit breaker exception 알람 구성 사례"
description: "이번 포스팅은 Elasticsearch 운영 중에 circuit breaker exception 발생시 알림을 구성하는 방법을 알아보도록 하겠습니다. 다나와는 일정 주기로 상품 인덱스를 다시 생성하는 과정, 그리고 상품의 변경사항만 문서를 업데이트하는 과정의 반복으로 인덱스를 운영하고 있습니다. 전체색인이 진행 중 서킷브레이크가 발생하였고, 전체 색인하는 클라이언트에서는  서킷브레이크에러에 처리가 누락이 존재하였고, 전체색인이 정상종료로 진행하는 바람에 운영되어 장애가 발생한 사례입니다. 대응방안중 하나로 알림구성에 포스팅합니다." 
date:   2021.12.17.
writer: "김준우"  
categories: Elastic 
---
## 개요
이번 포스팅은 Elasticsearch 운영 중에 circuit breaker exception 발생시 알림을 구성하는 방법을 알아보도록 하겠습니다. 다나와는 일정 주기로 상품 인덱스를 다시 생성하는 과정, 그리고 상품의 변경사항만 문서를 업데이트하는 과정의 반복으로 인덱스를 운영하고 있습니다. 전체색인이 진행 중 서킷브레이크가 발생하였고, 전체 색인하는 클라이언트에서는  서킷브레이크에러에 처리가 누락이 존재하였고, 전체색인이 정상종료로 진행하는 바람에 운영되어 장애가 발생한 사례입니다. 대응방안중 하나로 알림구성에 포스팅합니다.


### Circuit Breaker Exception 요약

엘라스틱서치에서는 OutOfMemory Error 발생전 CircuitBreakerException을 발생시켜 차단시키는 역할을 합니다. 

![/images/2021-12-17-es-circuit-breaker/2021-12-17_10h36_55.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_10h36_55.png)

힙메모리 서킷브레이크 발생 제한 수치
- parent limit: 95%
- fielddata limit: 60%
- request limit: 60%
- inflight limit: 60%
- accounting limit: 100%


### heap 사용량 확인 API
URI로 호출시 노드마다 limit size와 사용량에 대해 알 수 있습니다. 서킷브레이크 발생시 해당 API도 안될 가능성이 있습니다.
```
GET /_nodes/stats/breaker
```
![/images/2021-12-17-es-circuit-breaker/2021-12-17_10h57_38.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_10h57_38.png)


client에서 Circuit Breaker Exception 응답
![/images/2021-12-17-es-circuit-breaker/2021-12-17_10h49_15.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_10h49_15.png)

## 개선방안

1. 엘라스틱서치 로그에 서킷브레이크 출력
2. 파일비트로 로그 수집
3. 키바나의 logs 메뉴에서 alerts로 rocket.chat 채널로 알림


### 구성도
![/images/2021-12-17-es-circuit-breaker/2021-12-16_10h04_41.png](/images/2021-12-17-es-circuit-breaker/2021-12-16_10h04_41.png)


## 구성방법

### 서킷브레이크 로그 출력시키기.
공식문서에서는 로그가 출력된다고 명시가 되어 있습니다. 하지만 저희는 로그가 출력되지 않아 확인해봅니다.
![/images/2021-12-17-es-circuit-breaker/2021-12-17_12h31_08.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_12h31_08.png)

패키지에 서킷브레이크 관련 소스를 확인하였습니다.
![/images/2021-12-17-es-circuit-breaker/2021-12-17_12h34_05.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_12h34_05.png)
로그레벨이 디버그인걸 확인하였습니다. 그리고 Exception도 발생시킨단느걸 알았습니다.
![/images/2021-12-17-es-circuit-breaker/2021-12-17_12h34_59.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_12h34_59.png)

### 로컬에서 테스트해보기.

엘라스틱구성은 생략...



서킷브레이크 로그 레벨을 조정해보았습니다.
```
PUT /_cluster/settings
{
  "persistent": {
    "logger.org.elasticsearch.common.breaker": "DEBUG"
  }
}
```

서킷브레이크 limit 까지 요청을 보내기는 어려우므로 수치를 5%로 내렸습니다.
```
PUT /_cluster/settings
{
	"transient": {
		"indices": {
			"breaker": {
				"total": {
					"limit": "5%"
				}
			}
		}
	}
}
```


간단한 /_bulk API로 요청을 보내 서킷브레이크를 발생시켜봅니다.

![/images/2021-12-17-es-circuit-breaker/2021-12-17_12h50_43.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_12h50_43.png)

예상처럼 클라이언트에서 요청시 서킷브레이크 응답을 받았습니다.
![/images/2021-12-17-es-circuit-breaker/2021-12-17_15h05_16.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_15h05_16.png)

엘라스틱서치 로그에서 예외처리 내용도 출력되는걸 알 수 있습니다.
![/images/2021-12-17-es-circuit-breaker/2021-12-17_15h05_27.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_15h05_27.png)


### 이제 알림을 구성해봅니다.

 * 파일비트 구성 및 logs 수집 과정은 생략하겠습니다.

키바나에서 제공하는 alerts를 생성합니다.
로그 중 CircuitBreakingException이 1회 이상 출현시 action이 실행되도록 하였습니다. 팀내부적으로 Rocket.Chat을 사용하고 있어, 채널로 웹훅을 보내도록 하겠습니다.
![/images/2021-12-17-es-circuit-breaker/2021-12-17_15h22_58.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_15h22_58.png)

Rocket.Chat의 웹훅 스크립트는 아래 처럼 간단하게 메시지를 파싱 후 리턴하도록 하였습니다.

```
class Script {
  process_incoming_request({ request }) {
    let text = "서킷브레이크 에러 발생.";
    try {
        const content = JSON.parse(Object.keys(request.content||{}).join());
        text = content?.message ?? content;
    } catch(e) {
        console.log("서킷브레이크 파싱 중 에러 발생.", e)
    }
    
    return {
        content: { 
            text
         }
    };
  }
}
```

동작 확인을 위해 쿼리에서 CircuitBreakingException 대신 INFO로 바꿔서 알림을 확인해보았습니다. 정상적으로 알림을 받을 수 있었습니다.
![/images/2021-12-17-es-circuit-breaker/2021-12-17_15h29_11.png](/images/2021-12-17-es-circuit-breaker/2021-12-17_15h29_11.png)


## 정리

엘라스틱서치에서 서킷브레이크 로그를 출력하는 방법과 로그기반으로 키바나를 통해 로켓챗의 채널로 알림을 보내는 방법에 대해 알아보았습니다. 아무래도 네트워크 지연, 색인 지연이 존재하여 약 1~2초 가량 늦게 알람을 받는부분은 개선이 필요해 보입니다.


## 참고링크

- https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html
- https://martinschoeler.github.io/docs/administrator-guides/integrations/
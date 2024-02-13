---
layout: post
title: "다나와 Kotlin을 만나다!"
description: "다나와에서 Kotlin을 사용하기 시작한 이야기를 소개합니다."
date: 2024.02.01.
writer: "장민규, 윤성현, 이주영"
categories: Common
---

# 소개

안녕하세요. 다나와 검색 파트 장민규, 오피스 파트 윤성현, 이주영입니다.

다나와에는 많은 기술 스택을 보유 하고 있습니다.   
Java, PHP, Go, Python 등 다양한 언어를 사용하고 있으며, 최근에는 Kotlin을 도입하여 사용하고 있습니다.   
이번 포스팅에서는 Kotlin을 도입하게 된 이유와 어떻게 사용하고 있는지 소개하겠습니다.


## 왜 Kotlin을 사용하게 되었나요?

장민규: Kotlin을 시작하게 된 이유 단순했습니다. '요즘 코틀린이 유행이라고?' 때문인데요 22년도 말에 Kotlin을 도입하는 회사가 차츰 늘어나기 시작하였고    
저도 Kotlin에 호기심이 생겨 Kotlin을 학습하고 사용하다 Kotlin의 매력에 빠지게 되었습니다.   
일단 Java와 비슷하면서도 Kotlin만의 간결함 그리고 다른 언어들의 장점만 전부 때려 넣은듯한 언어였습니다.    
거기다 Java와 호환성 문제가 없어 혼합해서 사용해도 문제가 없고 또한 *null-safety* 는 몰론 `scope` 함수와 `runCatching`, `use` 같은 Kotlin만의 유용한 함수들이 있어서 더욱 매력적이었습니다.

한번 상품 명과 가격이 제대로 들어왔는지 검증하는 코드를 Java와 Kotlin 각각 비교해 보겠습니다.

Java
```java
void valid(String productName, Integer price) {
    if (productName == null || productName.isBlank()) { // Java11 부터 isBlank() 사용 가능
        throw new IllegalArgumentException("productName is blank");
    }
    if (price == null) {
        throw new IllegalArgumentException("price is null");
    }
}
```
Kotlin
```kotlin
fun valid(productName: String, price: Int) {
    require(productName.isNotBlank()) { "productName is blank" }
}
```
*require 함수는 Kotlin 표준 라이브러리에 있는 함수로 조건을 만족하지 않으면 IllegalArgumentException을 발생시킵니다.*

단장 위 두 코드를 보면 Kotlin 코드가 더 간결하고 가독성이 좋습니다.    
*null-safety* 그리고 각종 유용한 함수가 지원하기 때문입니다.

이러한 이유들로 Kotlin에 매력에 빠져 Kotlin을 학습하고 사용하게 되었습니다.

무엇보다 Go에 이어 Kotlin의 마스코트가 귀여워서는 비밀입니다. 😊   

![kodee](/images/2024-02-01-Meet-The-Kotlin/kodee.png)   

출처: [Kotlin 공식 홈페이지](https://blog.jetbrains.com/ko/kotlin/2023/05/the-kotlin-mascot-returns/)

성현, 주영: 처음에 민규님이 Kotlin을 도입하자고 발표하기 전까지는 Kotlin이란 언어에 대해서 잘 몰랐습니다.   
하지만 민규님의 Kotlin에 대한 설명을 듣고 Java와 비교해 보니 Kotlin이 더 간결하고 가독성이 좋다는 것을 알게 되었습니다.   
그렇게 Kotlin에 관심을 가지고 있을 때 민규님이 Kotlin 스터디를 모집하였고 저희도 참여하게 되었습니다.

Kotlin 스터디를 진행하면서 Kotlin의 장점을 더욱 느끼게 되어, 새로 개발할 프로젝트에 Kotlin을 도입하게 되었습니다.


## 다나와에서 Kotlin은 어떻게 도입되었나요?

장민규: 다나와에서 Kotlin의 시작은 저였습니다.

![kotlin1](/images/2024-02-01-Meet-The-Kotlin/kotlin-1.PNG)
![kotlin2](/images/2024-02-01-Meet-The-Kotlin/kotlin-2.PNG)

사진과 같이 ppt로 Kotlin을 소개 자료도 만들어 발표하고 위키에도 여러 장점들을 작성하였습니다.   
지금 같이 포스팅하고 있는 성현님과 주영님 그리고 Kotlin에 관심을 가진 팀원들과 함께 제 주도하에 스터디를 진행하기도 했습니다.   
퇴근 후, 주말에 개인 시간을 사용하여 기존 Java 프로젝트에 Kotlin을 추가하여 신규 기능은 Kotlin으로 개발하는 방법,    
기존 Java 코드를 Kotlin으로 마이그레이션 하여 다른 언어를 도입하는 것보다 리스크가 낮고 안정성이 높은 걸 입증하면서 Kotlin을 도입하게 되었습니다.

# 다나와에서 어떻게 사용하고 있나요?

## 검색에서 Kotlin

검색 파트에서 Kotlin은 현재 Elasticsearch 8.x 버전으로 업그레이드 작업을 준비 중    
Elasticsearch 라이브러리인 high-level-rest-client가 deprecated 이슈로 elasticsearch-java 마이그레이션 작업이 필요하기에   
기존 Java 프로젝트를 Kotlin과 함께 elasticsearch-java로 마이그레이션 작업을 진행하고 있습니다.

## 오피스에서 Kotlin

오피스 서비스 중 제휴 상품들의 가격을 실시간으로 갱신해 주는 서비스가 있었습니다.   
해당 서비스는 2010년도에 PHP로 처음 개발된 이후 개선이 이뤄지지 않았습니다.   
그 결과 가격 갱신 기능과 비즈니스 로직들이 혼합되어 지나치게 복잡도가 증가해 유지 보수가 어려웠습니다.

그래서 서비스의 복잡도를 낮추고 장애 대응에 용이하도록 새로운 서비스를 개발하기로 하였습니다.
실시간 가격 수집 기능은 수집기 파트에서 개발을 하였으며, 오피스 파트에서는 수집된 가격을 받아 가격을 갱신하는 기능을 개발하기로 했습니다.   
Kotlin의 null-safety, 간결함, 확장 함수를 사용하여 개발하면 비즈니스 로직을 보다 간결하게 개발할 수 있다고 판단하여 Kotlin으로 개발을 진행하였습니다.

## 결론

현재 다나와에서 Kotlin을 사용하는 파트는 검색, 오피스, 빅데이터로 총 3개의 파트가 존재합니다.   

Kotlin 처음 도입이라 지속적으로 어느 코드가 더 가독성이 좋고 유지 보수가 쉬운지 항상 고민하고 있습니다.      
Effective Kotlin 책에 적혀 있는 내용으로써  Kotlin의 `scope` 함수, `Elvis` 연산자 등 너무 남발해서 오히려 인식 부하가 오기 때문입니다.   
또한 어떻게 개발해야 생산성이 높아지는지 계속 연구하며 Kotlin을 사용하는 파트들끼리는 서로 Kotlin에 대한 지식을 공유하고 있습니다.
 
이 글을 읽어주신 모든 분들께 감사하며 Kotlin에 관심을 가진 개발자분들은 다나와에 많은 지원 부탁드립니다.   
감사합니다.

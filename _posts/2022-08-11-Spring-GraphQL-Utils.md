---
layout: post
title: "Spring for GraphQL에서 Interceptor와 Map"
description: "Spring for GraphQL에서 Interceptor 설정 방법이랑 Map을 반환하는 방법을 알아보도록 하겠습니다."
date: 2022.08.11.
writer: "장민규"
categories: Spring
---

안녕하세요. 이번에는 Spring for GraphQL에서 인터셉터 설정 방법과 Map을 반환하는 방법을 알아 보겠습니다.

## Spring for GraphQL Interceptor 사용한 이유
로그인한 회원은 모든 기능을 사용 가능하지만 비로그인 회원은 일부 기능만 사용이 가능하게 기능을 구현이 필요했습니다.   

또한 보안을 위해 JWT 토큰을 사용 중인데 처음에는 Filter로 토큰을 검증하는 방법을 구현하려 했지만   
GraphQL 특성상 엔드포인트가 1개여서 REST API처럼 특정 링크로 들어오는 요청에 필터를 통해 토큰 검증하는 식을 구현하기란 힘들었습니다.   

그래서 생각한 게 인터셉터 였습니다.

## Interceptor 설정 방법

```java
@Slf4j
@Configuration
public class JwtInterceptor implements WebGraphQlInterceptor {

    @Override
    public Mono<WebGraphQlResponse> intercept(WebGraphQlRequest request, Chain chain) {
        
        log.info("request = {}". request);
        log.info("request getDocument = {}". request.getDocument());
        log.info("request getHeaders = {}", request.getHeaders());
        
        return chain.next(request);
    }
}
```
Spring for GraphQL에서 인터셉터 등록하는 방법은 매우 쉽습니다.   

```@Configuration``` 어노테이션 하나면 바로 설정이 됩니다.   
다음에 로그를 찍고서 토큰을 담아서 요청을 하면   
![응답](/images/2022-08-11-Spring-GraphQL-Utils/1.PNG)
사진과 같이 응답이 오늘 걸 확인할 수 있습니다.

## 인터셉터 구현
A 유저가 B 유저에게 선물을 주는 GraphQL 쿼리가 존재하고 해당 쿼리는 로그인하여 jwt토큰으로 검증된 유저만 사용이 가능할 경우
```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class JwtInterceptor implements WebGraphQlInterceptor {
    
    ... 생략
    
    @Override
    public Mono<WebGraphQlResponse> intercept(WebGraphQlRequest request, Chain chain) {
        var giftItem = request.getDocument().contains("giftItem"); // request.getDocument()를 통해 쿼리에 해당하는 문자가 포함될 경우  
        var jwt = resolveToken(request); // request.getHeader() 를 통해 들어오는 값에서 jwt 토큰 값만 분리
        
        if (giftItem) { 
            if (isValidateToken(jwt)) { // jwt 토큰이 검증이 되면 정상적으로 반환
               return chain.next(request); 
            } else {
                throw new RuntimeException(); // 아닐 경우 Exception
            }
        }
        return chain.next(request); // giftItem 이외의 쿼리들은 반환
    }
    
    ... 생략
}
```
위와 같이 구현을 통해 일부 graphQL 쿼리를 인터셉터를 통해 제한하고 오픈하는 게 가능합니다.   

만약 mutation 쿼리는 전부 jwt 검증이 필요하다면 ```request.getDocument().contains("mutation");```   
이렇게 작성하여 mutation 쿼리는 전부 jwt 검증을 통과해야 하는 인터셉터를 구현할 수 있습니다.

## Map 반환하기

Map 반환은 [JPA로 RDB의 JSON 다루기](https://danawalab.github.io/spring/2022/08/05/Jpa-Json-Type.html) 포스팅과 연계됩니다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
@Entity
@TypeDef(name = "json", typeClass = JsonType.class)
@Table(name = "user_histories")
public class UserHistory {

    @Id
    private Long id;

    @Type(type = "json")
    @Column(name = "histories", columnDefinition = "longtext")
    private Map<String, Object> history = new HashMap<>();

    public static UserHistory of(Map<String, Object> history) {
        return new UserHistory(null, history);
    }
}
```

전에 포스팅한 글에 코드를 다시 발췌하여 설명하겠습니다.
위와 같은 Entity가 존재하고

```java
@RequiredArgsConstructor
@Service
public class UserHistoryService {
    
    private final UserHistoryRepository repository;
    
    @Transactional
    public void save(String exercise, String work, String movie) {
        var histories = Map.of(
                "exercise", exercise,
                "work", work,
                "movie", movie
        );
        
        var history = UserHistory.of(histories);
        repository.save(history);
    }
}
```
Map에는 exercis와, work, movie만 일정하게 들어간다고 가정하겠습니다.

```java
@Getter
@ToString
public class UserHistoryPayload {
    private final String exercise;
    private final String work;
    private final String movie;
    
    public UserHistoryPayload(UserHistory userHistory) {
        this.exercise = userHistory.getHistory().get("exercise").toString();
        this.work = userHistory.getHistory().get("work").toString();
        this.movie = userHistory.getHistory().get("movie").toString();
    }
}
```
```graphql
type UserHistoryPayload {
    exercise: String
    work: String
    movie: String
}
```

위와 같이 Payload를 만들고 Paylaod로 반환하면 됩니다.

저는 이걸 운용해서 exercise를 배열로 ```List<Map<String, Object>>``` 형식으로 만든 다음에 반환을 해보겠습니다.

변경된 코드
```java
@Getter
@ToString
public class UserHistoryPayload {
    private final List<Map<String, Object>> exercises;
    private final String work;
    private final String movie;
    
    public UserHistoryPayload(UserHistory userHistory) {
        this.exercises = (List<Map<String, Object>>) userHistory.getHistory().get("exercises");
        this.work = userHistory.getHistory().get("work").toString();
        this.movie = userHistory.getHistory().get("movie").toString();
    }
}
```
```graphql
type UserHistoryPayload {
    exercises: [exercise]
    work: String
    movie: String
}

type exercise {
    exerciseName: String
    exerciseTime: String
}
```
반환 시   
![map반환값](/images/2022-08-11-Spring-GraphQL-Utils/2.PNG)   

위 와 같은 값을 반환하는걸 볼 수 있습니다. 


## 정리
네 이번에는 Spring for GraphQL에서의 Interceptor 그리고 Map 반환하는 방법을 알아봤습니다.  

신규 프로젝트를 Spring for GraphQL 사용하다 보니 자료가 없어서 많이 힘들었지만 하나하나 해결해 가면서    
제가 많이 고민하고 시간 투자한 기능들을 나만 알지 말고 모두가 알았으면 해서 포스팅하게 됐습니다.   

만약 제 포스팅에서 설명하는 방법 보다 더 좋은 방법이 있다면 언제나 댓글 피드백은 환영합니다!

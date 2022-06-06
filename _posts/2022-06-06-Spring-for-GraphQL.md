---
layout: post
title: "Spring for GraphQL"
description: "Spring의 GraphQL에 대해 알아보도록 하겠습니다."
date: 2022.06.06.
writer: "장민규"
categories: Common
---

## 소개
안녕하세요 이번에는 최근에 Spring에서 공식 릴리즈한 Spring-for-GraphQL에 대해 알아보도록 하겠습니다.
![Spring-for-GraphQL](/images/2022-06-06-Spring-for-GraphQL/40.PNG)

- [graphql-java/graphql-java-spring](https://github.com/graphql-java/graphql-java-spring)
- [graphql-java-kickstart/graphql-spring-boot](https://github.com/graphql-java-kickstart/graphql-spring-boot)
- [Netflix/dgs-framework](https://github.com/Netflix/dgs-framework)

기존 Java 진영에서 Spring과 함께 GraphQL을 사용하기 위해서는
위 3개가 대표적인 Java 진영에서의 GraphQL을 사용하기 위한 라이브러리/프레임워크 이기에 3개 중 1개를 선택해 사용했다고 생각합니다.

위 3개는
[graphql-java/graphql-java](https://github.com/graphql-java/graphql-java) 를 기반으로 개발되었습니다.   
또한 이번에 알아볼 Spring-for-GraphQL은 공식적으로 [graphql-java/graphql-java](https://github.com/graphql-java/graphql-java)
의 후속 프로젝트라 소개 하고 있습니다.

### 환경

- JAVA 11
- SpringBoot 2.7.0

**Spring-for-GraphQL은 SpringBoot 2.7.0 버전 이상부터 지원합니다.**   

### 시작
Spring-for-GraphQL은 JPA 환경에서 쉽게 사용 가능하게 지원하고 있습니다.
또한 WebFlux 환경에서도 원활하게 지원하고 있습니다.

이번 포스팅에서는 WebMVC를 예제로 소개 드리겠습니다.

우선 프로젝트 루트 경로에 .graphqlconfig 파일을 만들어 주고 아래 내용을 넣어 주겠습니다.
```graphql
{
"name": "Graphql-Example",
"schemaPath": "src/main/resources/graphql/schema.graphqls",
"extensions": {}
}
```
IntelliJ에 GraphQL 플러그인을 설치하면 .graphqlconfig 파일을 쉽게 만드는게 가능합니다.

다음으로 application.yml을 사진과 같이 설정하도록 하겠습니다.
![application](/images/2022-06-06-Spring-for-GraphQL/15.PNG)
```yaml
graphql:
  graphiql: 
    enable: true
    # graphiql을 true를 설정해 주면 localhost:8080/h2-console과 같이 localhost:8080/graphiql 통해
    # graphql 쿼리를 테스트가 가능합니다.
    # 이 방법 말고도 IntelliJ에 GraphQL 플러그인을 설치해서 IntelliJ에서 직접 테스트도 가능하며 Postman으로도 가능합니다.
    printer: 
      enable: true
    # 이 설정 시 jpa에 show-sql 같이 graphql 쿼리를 출력해 줍니다.
```
![graphiql](/images/2022-06-06-Spring-for-GraphQL/2.PNG)

Food라는 Entity를 만들고 이어서 Repository와 Service 그리고 Controller를 만들어 보겠습니다.   
![Entity](/images/2022-06-06-Spring-for-GraphQL/1.PNG)
![Repository](/images/2022-06-06-Spring-for-GraphQL/13.PNG)
![Service](/images/2022-06-06-Spring-for-GraphQL/12.PNG)
```java
@Slf4j
@RequiredArgsConstructor
@Controller
public class FoodController {

    private final FoodService foodService;

    /**
     *  @MutationMapping은 @PostMapping과 같은 어노테이션으로 graphql에 Mutation에 사용됩니다. 
     *  graphql은 endpoint과 하나이므로 @MutationMapping 어노테이션만 지정해 주고 다른 설정은 필요 없습니다.
     */
    @MutationMapping 
    public Food save(@Argument String name) { // @Argument 는 @RequestBody, @RequestParam과 같은 인자값을 지정해줄 때 사용합니다.
        return foodService.save(name);
    }

    /**
     * @QueryMapping도 @GetMapping과 같은 어노테이션 입니다.
     * 말고도 @SubscriptionMapping이 있습니다.
     */
    @QueryMapping
    public Food getFood(@Argument String name) {
        return foodService.getFood(name);
    }

    @QueryMapping 
    public List<Food> getFoods() {
        return foodService.getFoods();
    }
}
```

다음으로 /resource/graphql 폴더에 schema.graphqls를 만들겠습니다.  

**.graphql이 아닌 .graphqls 파일이어야 정상적으로 작동합니다.**   

저와 같이 개발하던 동료도 개발 초기에 graphqls 가 아닌 graphql 파일로 만들다 보니 코드는 문제가 없는데 실행 시 예외가 생겨서 한참 들여다보곤 했습니다. 
```graphql
type Food {
    id: ID!
    name: String!
}

type Query {
    getFood(name: String!): Food # Controller에 @QueryMapping 메서드명과 같아야 합니다.
    getFoods: [Food]
}

type Mutation {
    save(name: String!): Food # Controller에 @MutationMapping 메서드명과 같아야 합니다.
}
```

db에 예제로 과일 이름을 넣고 graphql 쿼리를 실행해 보도록 하겠습니다.
![getFoods](/images/2022-06-06-Spring-for-GraphQL/11.PNG)   
성공적으로 쿼리를 실행하여 데이터를 조회했습니다.

### TDD
다음은 Spring-for-GraphQL에서 테스트 코드를 작성하는 방법을 알아보겠습니다.

test/resource에 graphql-test라는 폴더를 만들어주고 해당 폴더에 테스트할 쿼리를 만들어주면 됩니다.   
#### 테스트 할 쿼리들
```graphql
mutation save($name: String!) {
    save(name: $name) {
        id
        name
    }
}
```
```graphql
query getFood($name: String!) {
    getFood(name: $name) {
        id
        name
    }
}
```
```graphql
query {
    getFoods {
        name
    }
}
```
#### 테스트 코드
```java
@SpringBootTest
@AutoConfigureGraphQlTester
class FoodServiceTest {

    @Autowired
    private GraphQlTester graphQlTester;
    @Autowired
    private FoodRepository foodRepository;

    /**
     * service와 controller 에서 void로 반환하기에 반환 값이 null로 표시되기에
     * Food로 반환 값을 지정하면 테스트가 가능합니다.
     */
    @Test
    void save_쿼리_테스트() throws Exception {
        graphQlTester.documentName("save") // /resource/graphql-test에 만든 쿼리 이름을 적어주면 됩니다.
                .variable("name", "딸기") // 다음으로 인자 값을 적어줍니다. 첫음에는 인자 명, 두 번째는 인자 값
                .execute()
                .path("save.id") // 쿼리에 id를 반환하지 않을 경우 해당 값이 없어 예외가 터집니다.
                .entity(Long.class) // 반환 값에 자료형은 자바 클래스를 맞춰 주면 됩니다.
                .isEqualTo(1L)
                .path("save.name")
                .entity(String.class)
                .isEqualTo("딸기");
    }

    @Test
    void getFood_쿼리_테스트() throws Exception {
        foodRepository.save(Food.from("망고"));
        graphQlTester.documentName("getFood")
                .variable("name", "망고")
                .execute()
                .path("getFood.id")
                .entity(Long.class)
                .isEqualTo(1L)
                .path("getFood.name")
                .entity(String.class)
                .isEqualTo("망고");
    }

    @Test
    void getFoods_쿼리_테스트() throws Exception {
        foodRepository.save(Food.from("망고"));
        foodRepository.save(Food.from("딸기"));
        foodRepository.save(Food.from("키위"));
        foodRepository.save(Food.from("사과"));
        foodRepository.save(Food.from("배"));
        foodRepository.save(Food.from("귤"));

        graphQlTester.documentName("getFoods")
                .execute()
                .path("getFoods[*].name")  // 반환 값이 배열일 경우 배열 사용하듯 []로 테스트가 가능합니다.  
                .entityList(String.class) 
                .containsExactly("망고", "딸기", "키위", "사과", "배", "귤"); // 저장한 순서와 일치해야 합니다.
//              .path("getFoods[0].name") // 배열에 특정 값만 테스트할 경우
//              .entity(String.class)
//              .isEqualTo("망고");
    }
}
```
### DTO
Entity를 그대로 반환할 경우 원치 않는 데이터도 같이 반환될 경우가 있어 많이 Dto를 사용합니다.   
저는 FoodInput.java 와 FoodPayload.java를 만들어 적용해 보겠습니다.
```java
@Getter
@AllArgsConstructor(staticName = "from")
public class FoodInput {
    @NotBlank(message = "name은 필수")
    private String name;
}
```
```java
@Getter
@ToString
public class FoodPayload {

    private final String name;

    public FoodPayload(Food food) {
        this.name = food.getName();
    }
}
```
service 코드도 이에 맞춰 변경해 주고 나서 Controller 부분도 변경해 주겠습니다.
```java
   /** 
     * @Valid 와 @Validate 을 사용이 가능합니다
     */
    @MutationMapping
    public FoodPayload save(@Argument("input") @Valid FoodInput name) {
        return foodService.save(name);
    }

    @QueryMapping
    public FoodPayload getFood(@Argument FoodInput input) {
        return foodService.getFood(input.getName());
    }
```
그리고 schema.graphqls 를 이와 같이 변경해 주면 됩니다.
```graphql
type Food {
    id: ID!
    name: String!
}

input FoodInput {
    name: String!
}

type FoodPayload {
    name: String!
}

type Query {
    getFood(input: FoodInput!): FoodPayload 
    getFoods: [FoodPayload]
}

type Mutation {
    save(input: FoodInput!): FoodPayload
}
```
테스트 코드도 변경이 되어하는데
```java
    @Test
    void save_쿼리_테스트() throws Exception {
        Map<String, Object> var = new HashMap<>();
        var.put("name", "치킨");
        // var.put("calorie", 800);
        // var.put("price", 16000);
        
        graphQlTester.documentName("save") 
                .variable("input", var)
                .execute()
                .path("save.id")
                .entity(Long.class)
                .isEqualTo(1L)
                .path("save.name")
                .entity(String.class)
                .isEqualTo("치킨");
    }
```
위와 같이 작성해 주면 됩니다, Input 값이 name만 아니라 칼로리 같은 영양정보가 같이 입력되어야 한다면
map을 통해 관리해 주시면 됩니다.

FoodInput에 @NotBlank를 사용하여 name에 "" 값을 넘겨 줄 경우 사진과 같이 정상적으로 예외가 터지는 걸 확인할 수 있습니다.
![valid](/images/2022-06-06-Spring-for-GraphQL/30.PNG)

### GraphQL 파일 관리
![graphqls](/images/2022-06-06-Spring-for-GraphQL/33.PNG)
위 사진과 같이 resource/graphql 폴더에 graphqls 파일을 직관적으로 정리하기 위해서   
application.yml 파일에 코드를 추가하면 됩니다.
![application2](/images/2022-06-06-Spring-for-GraphQL/32.PNG)
```yaml
    schema:
      locations: classpath:graphql/**/
      fileExtensions: .graphqls, .gqls
```

### 마무리
기존 java 진영에서 사용하던 graphql 라이브러리/프레임워크를 사용하기 위해서 공통적으로 수많은 작업이 있었지만
Spring-for-GraphQL은 앞서 소개했듯이 graphql-java의 단순 후속 프로젝트뿐 아니라 graphql-java 개발팀이 개발을 하여서
Spring이 추구하는 방향답게 기존 MVC 개발하듯 추가적인 코드 없이 개발이 가능합니다.

저는 처음 개발하면서 Spring-for-GraphQL에서 제공하는 샘플과 공식 문서를 참고했는데,      
개발 당시 1.0.0 M3 버전이였는데 당시 @Validate 어노테이션을 지원 안했습니다.
저는 @Valid 보단 @Validate 어노테이션을 선호하는데 빈 값을 넘길 경우 그대로 Valid가 안되고 저장되는 경우도 있었고

테스트 코드에 필요한 쿼리 또한 test/resource/graphql-test 가 아닌 java/resource/graphql 한 폴더에 다 있어야 했습니다.
개발 초기에 수많은 시행착오가 많았습니다 또한 graphql을 처음 사용하는 만큼 익숙해지는 데 시간도 걸렸습니다,
그러나 지금은 개발 속도가 빨라졌고 graphql의 장점을 살려 개발하고 있습니다.

여러분들도 다음 프로젝트에 Graphql을 사용한다면
Spring-for-GraphQL을 사용해 보시는 거는 어떨까요?

---
참고:
- https://docs.spring.io/spring-graphql/docs/current/reference/html/
- https://fe-developers.kakaoent.com/2022/220113-designing-graphql-mutation/
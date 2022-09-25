---
layout: post
title: "Spring for GraphQL Custom Exception"
description: "Spring for GraphQL에서 Exception 다루는 법을 알아보겠습니다."
date: 2022.09.25.
writer: "장민규"
categories: Spring
---

안녕하세요 오늘은 Spring for GraphQL에서 Exception을 다루는 방법을 알아보겠습니다.

이번 포스팅은 Java 코드와 Kotlin 코드가 같이 포함되어 있습니다.
위에는 Java 코드 아래는 Kotlin 코드로 진행됩니다.

# 공통 Exception 그리고 GraphQLError 상속

공통 Exception을 만들고 GraphQLError과, RuntimeException을 상속해 주겠습니다.

```java
public class GraphqlException extends RuntimeException implements GraphQLError {

    public String message;

    public String getMessage() {
        return this.message;
    }

    @Override
    public List<SourceLocation> getLocations() {
        return null;
    }

    @Override
    public ErrorClassification getErrorType() {
        return null;
    }

    public GraphqlException(String message) {
        super(message);
    }

    @Override
    public Map<String, Object> getExtensions() {
        return Map.of(
                "exception", this.getClass().getName()
        );
    }
}
```

```kotlin
abstract class GraphqlException(
    
    @JvmField
    @Suppress("INAPPLICABLE_JVM_FIELD")
    override val message: String?
): GraphQLError, RuntimeException(message) {

    override fun getMessage(): String? = message

    override fun getLocations(): MutableList<SourceLocation>? = null

    override fun getErrorType(): ErrorClassification? = null

    override fun getExtensions(): MutableMap<String, Any> {
        return mutableMapOf(
            Pair("exception", this.javaClass.simpleName)
        )
    }
}
```

# ErrorType 상속

기본적으로 Spring for GraphQL에서 ErrorType 종류는 몇 가지 없기에 Exception을 다루는 만큼
ErrorType 종류도 늘려 주겠습니다.   

```java
public enum ErrorType implements ErrorClassification {
    CustomException
}
```

```kotlin
enum class ErrorType : ErrorClassification {
    CustomException
}
```
``ErrorClassification``을 상속하고 원하는 ErrorType을 enum으로 작성해주면 됩니다.

# CustomException

다음 CustomException을 만들어주겠습니다.    
앞서 만든 공통 Exception을 상속해 주겠습니다.

```java
public class CustomException extends GraphqlException {

    public String message;

    public CustomException(String message) {
        super(message);
        this.message = message;
    }

    @Override
    public String getMessage() {
        return message;
    }

    @Override
    public ErrorClassification getErrorType() {
        return ErrorType.CustomException;
    }
}
```
```kotlin
class CustomException(
    @JvmField
    @Suppress("INAPPLICABLE_JVM_FIELD")
    override val message: String? = "CustomException",
) : GraphqlException(message) {

    override fun getMessage(): String? {
        return super.getMessage()
    }

    override fun getErrorType(): ErrorClassification {
        return ErrorType.CustomException
    }
}
```

# Exception 등록

다음은 해당 Exception이 일어나면 해당하는 Exception이 나오게 설정하고 등록해 줍니다.

```java
@Configuration
public class GraphqlConfig extends DataFetcherExceptionResolverAdapter {

    @Override
    protected GraphQLError resolveToSingleError(Throwable ex, DataFetchingEnvironment env) {
        return ex instanceof CustomException ?
                (GraphQLError) (new CustomException(ex.getMessage())) : super.resolveToSingleError(ex, env);
    }
}
```
```kotlin
@Configuration
class GraphqlConfig: DataFetcherExceptionResolverAdapter() {

    override fun resolveToSingleError(ex: Throwable, env: DataFetchingEnvironment): GraphQLError? {
        return when (ex) {
            is CustomException -> CustomException(ex.message)
            else ->  super.resolveToSingleError(ex, env)
        }
    }
}
```

# 예외 

![code](/images/2022-09-25-Spring-GraphQL-Custom-Exception/1.PNG)

name이 5자가 넘으면 예외가 터지게 설정하겠습니다.


# 결과

![result](/images/2022-09-25-Spring-GraphQL-Custom-Exception/2.PNG)

다음과 같이 query를 날리면 사진과 같이 제가 직접 만들고 설정한 예외가 나오는 걸 확인할 수 있습니다.

# 정리

Spring for GraphQL을 다루면서 Exception을 커스텀 하려고 기존 REST API 하듯이 예외를 만들었는데
예외가 터지는 상황에 제가 만든 예외가 아닌 Spring for GraphQL에 Exception들 반환되어서 고생했던 기억이 있습니다.

그러다 최근에 해결 방법을 찾고 적용하는데 성공해서 이렇게 공유하기 위해 포스팅했습니다!

여러분들도 Spring에 GraphQL 개발해 보시는 건 어떠신가요?
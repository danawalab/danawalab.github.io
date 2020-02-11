---
layout: post
title:  "Apache Tomcat 쿠키 프로세서"
description: "톰캣 버전 증가에 따른 기술 표준이 변화된 내용과 그에 따른 쿠키 오류가 발생했던 원인을 알아보겠습니다. "
date:   2020.02.11.
writer: "안성일"
categories: Common
---


## 소개 및 오류 내용

여행 서비스 내 쿠키 생성 시, 다나와 서비스에 쿠키를 공하기 위해서 도메인 설정을 서브도메인('.danawa.com')으로 설정하였습니다.  
쿠기 생성 시 문제가 없었고 정상적으로 잘 작동되던 기능이 예상치 못한 오류가 발생했습니다.

최근에 기존 서비스의 Tomcat 버전을 업그레이드하면서 쿠키 생성 시 '.danawa.com'으로 도메인을 설정하면 에러가 발생된 다는 것을 알게 되었습니다. 
오류의 내용은
```
 "java.lang.IllegalArgumentException: An invalid domain [.danawa.com] was specified for this cookie"
```
쿠키 도메인으로 유효하지 않다는 내용이었습니다.


## 원인


해당 오류의 내용을 알아보면서, Tomcat과 cookieProcessor의 관계를 알게되었습니다.

Tomcat8은 LegacyCookieProcessor와 이를 대체할 수 있는 Rfc6265CookieProcessor를 제공하고 있습니다. 
따라서 사용자의 입장에 따라 알맞은 cookieProcessor의 구성 요소를 LegacyCookieProcessor로 설정할 수 있고, Rfc6265CookieProcessor로도 설정할 수 있습니다.

cookieProcessor는 className이라는 속성을 가지며, className의 값으로 'org.apache.tomcat.util.http.LegacyCookieProcessor' 및 'org.apache.tomcat.util.http.Rfc6265CookieProcessor'가 될 수 있습니다.
또한, 별다른 지정을 하지 않으면 Tomcat8은 기본 값으로 LegacyCookieProcessor를 사용하도록 되어 있습니다. 

## 레거시 쿠키 vs RFC6265 쿠키


"레거시 쿠키"는 V0, V1, 여러 구 RFC문서에 근거하는 다양한 스펙을 가질 수 있기 때문에 `레거시 쿠키는 톰캣의 LegacyCookieProcessor 가 인용한 스펙을 기준`으로 다루겠습니다. 실제로 LegacyCookieProcessor는 혼란스러운 표준 문제 때문에 RFC2109, 2616, 6265를 모두 부분 부분 인용하고 있습니다.

| 쿠키 | 버전속성 | Set-cookie 헤더 | 만료 메커니즘 | 도메인 속성 | 쿠키 name,value 제한사항 | 쿠키값 |
| --- | --- | --- | --- | --- | --- | --- |
| 레거시 | Version=0, 1 | Set-cookie, Set-cookie2 | max-age, expires 혼용 | `.`으로 시작 | name,value 모두 HTTP/1.1 token형식 | name=이후의 token형식 value 또는 `"`로 감싸진 값 |
| RFC6265 | 사용하지 않음  | Set-cookie | max-age가 있을 경우 expires 무시 | `.`으로 시작하지 않음 | name만 HTTP/1.1 token 형식 | 첫 `=`와 첫 `;`사이의 문자 |

### 쿠키 버전

- 레거시 쿠키: Version 헤더를 사용해서 cookie-version을 명시하며 넷스케이프 스펙의 쿠키는 Version=0을 사용하거나 버전 헤더를 사용하지 않고, RFC2965 쿠키는 Version=1을 사용합니다. 톰캣의 LegacyCookieProcessor는 버전 0의 쿠키를 만드는 것을 시도하고, 특별히 버전 1의 스펙을 인용할 때만 버전 1로 쿠키를 생성합니다.
- RFC6265 쿠키: 새로운 스펙을 제시하기보다 현업 스펙을 정리하는 것을 목표로 했기 때문에 $Version을 사용하지 않습니다.

### Set-cookie 헤더

- 레거시 쿠키: Set-cookie를 통해 일반 쿠키 프로세싱을 하고, Set-cookie2 스펙을 통해 RFC2965 전용 스펙을 사용하지만, 톰캣의 LegacyCookieProcessor는 Set-cookie2를 사용하고 있지 않습니다.
- RFC6265 쿠키: Set-cookie만을 사용하고 있습니다.

### 쿠키 만료 메커니즘

- 레거시 쿠키: RFC2965는 Expires 속성을 허용하고 있지 않으며, HTTP/1.1 스펙으로 계산한 max-age만을 허용하고 있습니다. 하지만 IE6,7 브라우저는 max-age를 구현하지 않았기 때문에 톰캣의 LegacyCookieProcessor는 max-age와 동일한 expires를 계산해서 둘 다 추가하고 있습니다. 즉시 만료는 max-age=0을 사용합니다.
- RFC6265 쿠키: 동일한 쿠키에도 expires와 max-age를 둘 다 사용 가능하지만, max-age가 있는 경우 expires를 완전 무시하도록 되어 있습니다. 즉시 만료는 과거의 시간을 expires 어트리뷰트에 사용합니다.

### 도메인 속성

- 레거시 쿠키: V0의 스펙은 도메인 문법에 대해 자세히 기술하고 있지 않습니다. RFC2965의 V1스펙에서는 도메인명은 `.`으로 시작한다고 명시하고 있으며, abc.com 은 자동으로 .abc.com으로 전환하여 저장하도록 되어 있습니다.
- RFC6265 쿠키: 레거시 쿠키와 정면으로 반대되는 스펙으로, 도메인명의 첫 글자가 `.`인 것을 혀용 하지 않습니다. 스펙에는 위배되지만 첫 `.`을 확인하면 해당 문자를 무시해서 프로세싱합니다.

RFC6265 스펙에 정의된 것과는 다르게, 톰캣의 Rfc6265CookieProcessor는 8.5.x 버전부터 첫 .을 무시하지 않고 예외가 발생하게 됩니다.

### 톰캣 버전별 차이

| 항목 | Tomcat LegacyCookieProcessor | Tomcat 8.0.x Rfc6265CookieProcessor | Tomcat 8.5.x Rfc6265CookieProcessor |
| --- | --- | --- | --- |
| cookie-value : cookie-value에 특수문자가 있으면 자동으로 `"`로 감싸는가? | O | X | X |
| cookie-value : `"` 로 감싸지지 않은 cookie-value에 특수문자가 있으면 인식이 가능한가? | X (첫 HTTP/1.1 토큰의 separator 이후 값 사라짐) | O | O |
| domain : domain이 `.`으로 시작해도 처리가 가능한가? | O (스펙 표준이다)  | O (스펙은 아니지만 무시하고 처리) | X (에러) |


## 해결


Tomcat 서버 사용시 context.xml에 추가

```
 <CookieProcessor className="org.apache.tomcat.util.http.LegacyCookieProcessor"/>
```

SpringBoot에서 Embedded Tomcat에 설정

```
@Bean
public EmbeddedServletContainerCustomizer tomcatCustomizer() {
    return container -> {
        if (container instanceof TomcatEmbeddedServletContainerFactory) {
            TomcatEmbeddedServletContainerFactory tomcat = (TomcatEmbeddedServletContainerFactory) container;
            tomcat.addContextCustomizers(context -> context.setCookieProcessor(new LegacyCookieProcessor()));
        }
    };
}
```

Rfc6265CookieProcessor와 LegacyCookieProcessor는 각각 장단점이 있습니다. 
Rfc6265CookieProcessor가 더 관대하다는 장점이 있지만, 현재 나온 Tomcat버전들은 대부분 LegacyCookieProcessor를 지원하기 때문에 일괄적으로 쿠키 프로세서를 통일하기가 쉽습니다.
그리고 LegacyCookieProcessor의 경우 스펙 하향평준화를 한다는 점이 마음에 걸릴 수밖에 없는 것은 단점입니다.

근본적인 해결책

- 어떤 쿠키 프로세서든 HTTP/1.1 스펙 토큰의 형식을 준수하여 LegacyCookieProcessor에서도 따옴표 없이 인식을 할 수 있게 합니다.
- 특수문자를 가능하면 사용하지 않습니다.
- 특수문자가 포함된다면 URL safe 인코딩을 합니다.



## 참고문서


- [https://feel5ny.github.io/2019/11/16/HTTP_011_02/](https://feel5ny.github.io/2019/11/16/HTTP_011_02/)
- [https://jistol.github.io/java/2017/08/30/tomcat8-invalid-domain/](https://jistol.github.io/java/2017/08/30/tomcat8-invalid-domain/)
- [https://meetup.toast.com/posts/209](https://meetup.toast.com/posts/209)
- [https://tomcat.apache.org/tomcat-8.5-doc/config/cookie-processor.html](https://tomcat.apache.org/tomcat-8.5-doc/config/cookie-processor.html)
- [http://web04.echomail.com/docs/config/cookie-processor.html](http://web04.echomail.com/docs/config/cookie-processor.html)

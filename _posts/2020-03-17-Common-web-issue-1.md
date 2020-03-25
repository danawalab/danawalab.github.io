---
layout: post
title:  "2020년 1/3분기 (1~4월) 웹 브라우저 이슈"
description: 2020년은 구글 크롬을 필두로 개인정보 보호관련 업데이트가 많이 일어날 것으로 예상됩니다. 영향력있는 소프트웨어가 정책을 제시하면 타 소프트웨어들이 표준처럼 따르는 관례가 있는데요. 이를 근거로 구글의 행보가 표준안 이 될 가능성이 높습니다. 많은 관심이 필요해 보입니다
date:   2020-04-01
writer: "김명운"
categories: Common
---

![web-issue-1-image-1](https://k.kakaocdn.net/dn/tET4b/btqCxP83NlQ/JciKUEtbAnT7llEqkTAkZK/img.png){: .image-center}

2020년은 구글 크롬을 필두로 개인정보 보호관련 업데이트가 많이 일어날 것으로 예상됩니다. 영향력있는 소프트웨어가 정책을 제시하면 타 소프트웨어들이 표준처럼 따르는 관례가 있는데요. 이를 근거로 구글의 행보가 표준안 이 될 가능성이 높습니다. 많은 관심이 필요해 보입니다.

2020년 1/3분기 웹 브라우저 이슈에 대해 살펴 보겠습니다.

<br/><br/>

# 1. 서드파티 쿠키(Third-Party Cookie) 2년내로 사용불가.

![web-issue-1-image-2](https://k.kakaocdn.net/dn/nucjr/btqCy4EKoje/K6WT7NG9p6asq6WAkg1JAk/img.png){: .image-center width="60%"}

서드파티 쿠키(Third-Party Cookie)란 사용자가 방문한 웹사이트가 아닌 다른 웹사이트에서 발행한 쿠키를 말합니다. 구글 크롬은 2년안에 서드파티 쿠키지원을 중단한다고 발표했습니다.

서드파티 쿠키는 주로 광고업계에서 이용했습니다. 사용자 브라우저에 스크립트가 담긴 쿠키를 발행하고, 사용자의 브라우저 사용 정보를 알아내기 위해 많이 사용했습니다. 예를 들어 사용자가 쿠팡에서 본 상품 정보들이 구글 광고에 뜨는것을 심심치 않게 볼 수 있는데요. 이 때 사용한 방법이 서드파티 쿠키입니다.

사용자의 기록을 추적하기 위한 행위는 개인정보보호 이슈로 판단되어 사파리, 모질라 파이어폭스, 엣지 등은 이미 쿠키 추적기능을 방지하거나, 서드파티 쿠키사용을 금지 했습니다. 크롬의 경우 후발주자라고 볼 수 있는데요. 다른 브라우저에 비해 크롬의 경우 점유율이 높아 영향도가 클것으로 예상됩니다.

#### 참고자료
- <http://it.chosun.com/site/data/html_dir/2020/02/16/2020021600579.html>
- <https://www.newspim.com/news/view/20200115000078>

<br/><br/>

# 2. 크롬 80, SameSite 쿠키 이슈

![web-issue-1-image-3](https://k.kakaocdn.net/dn/pXk1t/btqCylUflyD/qyZ4MzU0rCLnzkQfmEOqe0/img.png){: .image-center width="40%"}

구글은 2월 크롬80버전을 업데이트 하면서 쿠키의 SameSite 의 디폴트 값을 "None" 에서 "Lax"로 변경 했습니다.

쿠키의 SameSite란 쿠키의 서로 다른 도메인간(크로스 사이트) 쿠키 전송에 관한 설정입니다. 각 설정 값으로 "None", "Lax", "Strict"가 있습니다. 각각의 설정값은 다음을 의미합니다.

| SameSite 값 | 설명 | 크로스 도메인 처리 | CSRF 위험도 |
| :---: | :---: | :---: | :---: |
| None | 사용 도메인과 크로스 도메인 모두 허용 |	모두 허용 | 높음 |
| Lax | 사용 도메인과 크로스 도메인 방식 일부 허용 | HTTP GET method, `<a>`, `<link>` 방식 허용 | 낮음 |
| Strict | 사용 도메인만 사용 |	모두 차단 | 거의 없음 |

구글 크롬은 SameSite 의 디폴트 값을 "Lax" 로 설정해 CSRF 에 대비한 보안 수준을 올렸습니다. 기존처럼 "None" 속성을 사용하기 위해선 쿠키의 Secure 속성을 부여하여 HTTPS 환경에서만 사용되어야 합니다.

구글 크롬과 더불어 모질라 파이아폭스, 사파리 등도 SameSite의 기본 값이 바뀜에 따라 iframe 서비스, 인증 대행, 결제  서비스등에 의해 영향을 끼칠 것으로 예상됩니다.

#### 참고자료

- <https://developers-kr.googleblog.com/2020/01/developers-get-ready-for-new.html?fbclid=IwAR0wnJFGd6Fg9_WIbQPK3_FxSSpFLqDCr9bjicXdzy--CCLJhJgC9pJe5ss>

<br/><br/>

# 3. 크롬, HTTPS 아닌 다운로드 점진적으로 차단

![web-issue-1-image-4](https://k.kakaocdn.net/dn/lmdMA/btqCyl7Nsqo/yqWDoChXfuailDy82cC3z0/img.png){: .image-center}

구글 2월달 크롬80을 발표할 때 HTTPS가 아닌 다운로드를 점진적으로 차단한다고 발표했습니다. 이와 더불어 TLS(Transport Layer Security) 1.0 ,1.1 버전을 지원 중단 한다고 발표했습니다.

> TIP : TLS(Transport Layer Security)이란 SSL(Secure Sockets Layer)의 기술을 기반으로 만들어진 표준으로 SSL로 많이 불리지만 정식 명칭은 TLS이 맞습니다.

<br/>
구글에서 공개한 HTTP 다운로드 차단 일정은 다음과 같습니다.

| 날짜 | 크롬 버전 | 처리 내역 |
| :---: | :---: | :---: |
| 2020.03 | Chrome 81 | 모든 HTTP 다운로드시 콘솔 경고 기록 <br/> TLS 1.0, 1.1 지원 중단 | 
| 2020.04 | Chrome 82 | 실행 파일에 대한 HTTP 다운로드 경고 노출 |
| 2020.06 | Chrome 83 | 실행 파일에 대한 HTTP 다운로드  차단 <br/> 압축 파일 및 디스크 이미지에 대하 HTTP 다운로드 경고 노출 |
| 2020.08 | Chrome 84 | 압축 파일 및 디스크 이미지에 대하 HTTP 다운로드 차단 <br/> 문서 형식 파일에 대한 HTTP 다운로드 경고 노출 |
| 2020.09 | Chrome 85 | 문서 형식 파일에 대한 HTTP 다운로드 차단 <br/> 미디어 형식 파일에 대한 HTTP 다운로드 경고 노출 |
| 2020.10 | Chrome 86 | 미디어 형식 파일에 대한 HTTP 다운로드 차단 |

#### 참고자료
- <https://www.zdnet.com/article/google-to-block-some-http-file-downloads-starting-with-chrome-83>
- <https://nakedsecurity.sophos.com/2020/02/10/google-chrome-to-start-blocking-downloads-served-via-http>

<br/><br/>

# 4. User-Agent 크롬 점진적으로 제거

![web-issue-1-image-5](https://k.kakaocdn.net/dn/bumarr/btqCRzyKqVL/hNwvDQJkLlbPYDi6jlGBJ0/img.png){: .image-center width="90%"}

구글은 올해 초 크롬 브라우저 User-Agent를 점진적으로 중지할 계획이라며 크롬 85까지 지원 중단 계획을 발표했습니다. 
User-Agent엔 불필요한 정보들이 많이 있어 개인정보에 취약하다는 명분으로, 크롬 81, 83, 85 버전 업그래이드를 통해 점진적으로 중지할 것이라 발표 했습니다.

User-Agent란 사용자가 HTTP 프로토콜 안에서 동작할 때 자신의 환경을 문자열로 제공하는 기술로 흔히 해더 정보를 이용해 제공됩니다. 
많은 웹 로직들이 User-Agent사용해 사용자의 환경에 따른 분기처리나, 버전 별 분기처리를 할 수 있는데요. 그렇기에 User-Agent에 중지 이슈는 완벽한 검토와 대비가 필요합니다.

구글에서 발표한 버전별 일정과 과정은 다음과 같습니다.

| 날짜 | 크롬 버전 | 처리 내역 |
| :---: | :----: | :---: |
| 2020.03 | Chrome 81 | User-Agent 문자열을 읽는 웹페이지에 대해 크롬 콘솔에 경고를 표시할 계획이며, 이를 통해 개발자들은 웹사이트 코드를 조정할 수 있도록 유도 |
| 2020.06 | Chrome 83 | User-Agent 문자열에서 Chrome 브라우저 버전을 동결하고 OS 버전을 통합 |
| 2020.09 | Chrome 85 | User-Agent 데스크톱 OS 문자열을 데스크톱 브라우저의 공통 값으로 통합, 모바일 OS/기기 문자열을 비슷한 공통의 값으로 통합한다. |

구글은 User-Agent 대체 기술로 자사에서 만든 "Client Hint"를 제시했습니다. Client Hint는 사용자의 선택된 정보만 가지고 올 수 있는 
기술이며 정보의 노출을 최소한으로 할 수 있다는 이점이 있습니다. 자세한 내용은 아래 참고자료를 확인해주세요.

추가적으로 구글에서 사파리, 파이어폭스, 엣지 각 제조사들에게 User-Agent 지원 중단 정책 제안을 했다 하는데요. 
이에 대한 답변으로 지원 또는 동의 한다고 밝혔으며 아직 상세한 일정은 밝혀지지 않은 상태입니다.

#### 참고자료
- <https://www.zdnet.com/article/google-to-phase-out-user-agent-strings-in-chrome>
- <https://wicg.github.io/ua-client-hints>

<br/><br/>

### 기타 이슈
- 크롬 2020년 12월 이후 Adobe flash player 지원 중단.
- 유투브 Internet Explorer 지원 중단, Chromium 기반 Microsoft Edge 출시로 Internet Explorer의 점유율 큰 폭으로 하락 예상.

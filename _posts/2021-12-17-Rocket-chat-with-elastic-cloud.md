---
layout: post
title:  "엘라스틱 클라우드를 이용하여 로켓 챗으로 알람 받기"
description: "엘라스틱 클라우드를 이용하여 로켓 챗으로 알람 받기"
date:   2021.12.17. 10:00:00
writer: "선지호"
categories: Elastic
---
# 소개

엘라스틱서치는 자동으로 배포, 관리되는 클라우드 서비스를 제공하고 있습니다.

이번에는 엘라스틱 클라우드 서비스를 이용하여 로켓챗으로 알람을 받아 보겠습니다.

## 이전 글

엘라스틱 서치 클라우드에 가입, 배포 서비스 생성 및 설정하는 부분은 아래 링크를 참조해 주세요

[엘라스틱서치 모니터링 및 이상탐지](https://danawalab.github.io/elastic/2021/08/30/es-monitoring-and-alert.html, "엘라스틱서치 모니터링 및 이상탐지")
---

## 로켓챗 알람 설정

로켓챗에 알람을 적용하려면 관리자 계정으로 접속해야 합니다.

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/1.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/1.png)

관리자 계정으로 접속하여 계정 이름을 클릭한 뒤 '관리' 버튼을 클릭합니다.

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/2.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/2.png)

관리 버튼을 누르면 오른쪽엔 상세한 내용이 나오고, 왼쪽에는 관련된 탭들이 있습니다.

먼저, 인티그레이션 탭을 눌러 이동합니다.

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/3.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/3.png)

Incoming, OutGoing, Zapier 그리고 봇, 이렇게 4가지의 탭으로 구성되어 있습니다.

이 중에서 저희가 사용할 부분은 Incoming 부분입니다.

이미지에는 제가 미리 생성한 알람들이 적용이 되어 있습니다.

우측 상단의 '+ New' 버튼을 눌러 한번 생성해 보겠습니다.

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/4.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/4.png)

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/5.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/5.png)

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/6.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/6.png)


먼저 이름, Channel에 게시, 작성자 세가지를 작성해 줍니다.

이름은 테스트, Channel은 #blog-alarm, 작성자는 danawa 로 작성해 주겠습니다. (Channel은 이미 존재해야 하는 채널이어야 합니다.)

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/7.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/7.png)

그리고 스크롤을 조금 내려 스크립트 사용을 On 해주신 다음 저장을 누릅니다.

저장을 하게되면 아래처럼 토큰과 Webhook URL이 생깁니다. Webhook URL을 복사해 둡니다.

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/8.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/8.png)

하지만 아직 셋팅을 하지 않은 것이 하나 있습니다. 스크립트 입니다.

공식문서를 참조하여 스크립트를 한번 작성해 보겠습니다.

[공식문서](https://docs.rocket.chat/guides/administration/admin-panel/integrations, "공식문서")

```js
// JSON 포맷으로 받을 경우 
class Script {
  process_incoming_request({ request }) {
    return {
        content: {
            text: request.content.text
         }
    };
  }
}
```

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/9.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/9.png)

스크립트를 작성한 뒤, 저장을 눌러줍니다.

이때, 반드시 이름 부분에 있는 활성화를 Enable 해주셔야 합니다. Enable을 하지 않는다면 아래 이미지와 같이 정상적으로 알림을 받지 못합니다.

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/16.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/16.png)
---

## 엘라스틱 클라우드 알림 설정

이제, 엘라스틱 클라우드로 가서 webhook을 등록해보겠습니다. 
엘라스틱 클라우드에 대한 내용은 이전 블로그 포스팅에서 다루었기 때문에 상세한 내용은 다루지 않겠습니다.

Elastic Cloud의 Kibana - Stack management - Rules and Connectors 로 들어가 줍니다.

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/10.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/10.png)

그리고, Create Connector - Webhook 버튼을 클릭해 줍니다.

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/11.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/11.png)

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/12.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/12.png)

커넥터 이름과 Authentication 을 disable (만약 인증이 걸려 있다면 enable 상태로 입력을 더 해주셔야 합니다.) 한 뒤, 아까 저장해두었던 URL을 붙여넣기 합니다.

그리고 아래 이미지와 같이 입력해 주신 후 add 버튼을 눌러 헤더를 등록해 줍니다.

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/13.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/13.png)

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/14.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/14.png)

이후 Save & Test 버튼을 눌러 마지막 확인을 해보겠습니다.
---
### Test

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/17.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/17.png)

![/images/2021-12-17-Rocket-chat-with-elastic-cloud/18.png](/images/2021-12-17-Rocket-chat-with-elastic-cloud/18.png)
---

## 정리

엘라스틱서치 클라우드와 로켓챗의 Webhook 연동을 한번 알아보았습니다.

엘라스틱서치 클라우드와 사내 메신저로 로켓챗을 사용하여 연동을 고민하시는 분이 있다면 이 글을 보고 도음이 되었으면 합니다.

감사합니다.
--- 

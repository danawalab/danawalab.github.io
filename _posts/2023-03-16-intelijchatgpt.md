---
layout: post
title:  "IntelliJ "
description: "IntelliJ chatGPT Plugin Setting"
date:   2023.03.16.
writer: "반윤성"
categories: Common
---

## IntelliJ chatGPT 플러그인 ?
일반적으로 IntelliJ 플러그인을 사용하면 소프트웨어 개발에 필요한 여러가지 도구를 사용할 수 있습니다. 

ChatGPT와 관련하여, OpenAI에서 제공하는 API를 사용하여 플러그인을 사용할 수 있습니다. 
이 플러그인을 사용하면 IntelliJ 챗봇과 같은 대화형 도구를 이용할 수 있습니다.

보통 브라우저를 통해 이용하기도 하지만 접근성, 실용성 측면에서 IDE에 플러그인을 셋팅하는것이 효율적인 방법으로 생각됩니다.

## chatGPT 플러그인 셋팅 방법

#### 플러그인 설치
플러그인을 설치하기 위해 Setting -> IntelliJ Marketplace 로 이동하여 ``ChatGPT``를 설치합니다.
다른 플러그인도 있지만 다운로드 수와 인기가 가장 많은 플러그인입니다.

![/images/2023-03-16-ChatGPT/image0.png](/images/2023-03-16-ChatGPT/image0.png)


#### 플러그인 설정
![/images/2023-03-16-ChatGPT/image1.png](/images/2023-03-16-ChatGPT/image1.png)

설치가 완료되면 다음과 같이 우측 탭에 ChatGPT 항목이 생성됩니다.
하지만 지금 명령어를 전송하면 인증키가 없기 때문에 에러가 발생합니다.

아래 사진과 같이 셋팅으로 이동해 키를 설정합니다.

![/images/2023-03-16-ChatGPT/image2.png](/images/2023-03-16-ChatGPT/image2.png)

![/images/2023-03-16-ChatGPT/image3.png](/images/2023-03-16-ChatGPT/image3.png)

이 곳에서 API Key를 발급받아서 플러그인과 연동합니다.

<https://platform.openai.com/account/api-keys>

![/images/2023-03-16-ChatGPT/image4.png](/images/2023-03-16-ChatGPT/image4.png)


#### 플러그인 사용 예시
플러그인이 셋팅되었다면 이제 활용할 수 있습니다. 기존처럼 코드를 복사하거나, 주석을 얻을 수 있는 방법도 있지만
다음과 같이 코드 블럭을 잡아서 명령을 내릴수도 있습니다.

![/images/2023-03-16-ChatGPT/image5.png](/images/2023-03-16-ChatGPT/image5.png)

여기서 기본적으로 제공하는 Find Bug, Optimize 이외에도 커스텀 액션을 생성하여 개발자 입맛에 맞게 해당 코드에
대해 지시할 수 있습니다. 샘플로 테스트 코드를 생성하는 액션을 작성해보겠습니다.

![/images/2023-03-16-ChatGPT/image6.png](/images/2023-03-16-ChatGPT/image6.png)

![/images/2023-03-16-ChatGPT/image8.png](/images/2023-03-16-ChatGPT/image8.png)

![/images/2023-03-16-ChatGPT/image8.png](/images/2023-03-16-ChatGPT/image9.png)

다음과 같이 원하는 결과를 얻었습니다.

## 정리
ChatGPT를 점점 업무에 도입하는 기업과 실무자가 많은데, 브라우저를 통해 접근하기 보다는 각자의 IDE 환경에서 실행하는 것이 조금 더 생산성 있는 방법이라고 생각합니다. API key를 통해서 쉽게 이용할 수 있고 turbo 또한 지원하고 있어서 여러모로 쓸모있는 플러그인으로 보입니다.
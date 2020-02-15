---
layout: post
title:  "2020년 Vue.js UI 컴포넌트 라이브러리 종류"
description: "프론트 개발의 비용을 줄여줄 수 있는 UI 컴포넌트 라이브러리, Vue.js에는 어떤게 있을까요?"
date:   2020.02.15.
writer: "최순현"
categories: Common
---


![/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%201.png](/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%201.png)

Vue.js는 2.0버전 이상부터 React와 동일하게 Virtual DOM 기반의 자바스크립트 프레임워크입니다.  
브라우저의 DOM을 조작하는 것은 매우 무거운 작업인데, 이러한 성능적인 이슈를 개선할 수 있는 프레임워크입니다.

Vue.js가 출시한 지 6년이 지났고 2017년에는 Github 즐겨찾기 수(Star)가 jQuery를 앞서는 만큼의 인기를 누리고 있지만  
여전히 대부분 프론트에 의존도가 크지 않은 개발자들은 적용을 하지 않고 있습니다.


![/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%202.png](/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%202.png)

성능이 뛰어난 프레임워크여도 사용성이 복잡하거나 성능 비용의 이익에 비해 개발 비용이 커지게 되면 기존의 익숙한 기술에서 새로운 기술로 이전하기 꺼려지게 됩니다.

웹 프론트엔드 개발자(또는 퍼블리셔), 웹 그래픽 디자이너 없이 서버사이드의 개발을 선호하는 작업자로 이루어진 소규모 팀의 경우  
개발 비용의 축소와 편의성을 위해 UI 컴포넌트 라이브러리르 사용하곤 합니다.

다나외의 경우 관리자 페이지는 퍼블리셔와 디자이너의 손길이 닿지 않는 페이지가 대부분인데, 이럴 때 주로 UI 컴포넌트 라이브러리를 사용합니다.  
~~개발자 감성과 기능 위주로 만들어지다보니 이쁘지는...~~


## 1. Bootstrap Vue

![/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%203.png](/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%203.png)

- 공식 : [https://bootstrap-vue.js.org/](https://bootstrap-vue.js.org/)
- 깃허브 : [https://github.com/bootstrap-vue/bootstrap-vue/](https://github.com/bootstrap-vue/bootstrap-vue/)
- 라이센스 : MIT License

반응형, 컴포넌트 모듈 방식의 사용. 웹 접근성으로 무장한 Bootstrap이 jQuery 뿐만 아니라 Vue 기반도 지원합니다.

트위터 개발자 한 명이 시작한 프론트엔드 프레임워크입니다. 현재는 오픈소스로 운영되며 트위터에서 주도적으로 개발하고 있지 않는 상태입니다.

대부분의 컴포넌트가 HTML5에 맞춰져 있어 특정 이하 버전의 브라우저에서는 동작하지 않습니다.  
(2020년에는 IE8 이하 버전을 쓰는 사람은 없을 거라고 믿고 싶다.)

![/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%204.png](/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%204.png)  
_화려한 구성보다는 그리드 레이아웃과 여백이 많은 모던 디자인에 적합한 Bootstrap (디자이너분은 의견좀)_

Bootstrap Vue 공식 웹페이지에서 안내하는 필요 요소는 아래와 같습니다. (2020-02-08 기준)
- Vue.js 2.6 버전이 필요하며 2.6.11 버전을 권장합니다.
- Bootstrap 4.3.1 버전이 필요하며 4.4.1 버전을 권장합니다.
- PortalVue 2.1 버전이 필요하며 2.1.7 버전을 권장합니다. (Toasts 관련)
- jQuery는 필요하지 **않습니다.**

[https://bootstrap-vue.js.org/play](https://bootstrap-vue.js.org/play) 에서 간단한 템플릿과 자바스크립트를 작성해볼 수 있습니다.


## 2. Vue Material Kit

![/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%205.png](/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%205.png)

- 공식 : [https://www.creative-tim.com/product/vue-material-kit](https://www.creative-tim.com/product/vue-material-kit)
- 깃허브 : [https://github.com/timcreative](https://github.com/timcreative)
- 라이센스 : MIT License, Paid License(Personal License, Developer License)

Nodejs 및 Vue CLI를 바탕으로 개발 가능한 UI 컴포넌트 라이브러리입니다.  
화면 구성에 대한 예시가 다양하고 이미 맥 또는 리눅스 인프라가 구성된 개발 환경이라면 셋팅도 간단하다는 장점이 있습니다.

Vue Material Kit은 이름에서도 알 수 있듯이 Google 머티리얼 디자인에서 영감을 받아 제작된 컴포넌트입니다.

![/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%206.gif](/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%206.gif)  
_Google 머티리얼 디자인은 단순한 색상과 구성에 따른 직관적인 식별성(플랫 디자인)과 그림자 효과를 더한 구성이다._

기본적으로 MIT License를 적용 받지만 Pro 버전의 경우 유료 License입니다. 라이센스 비교는 아래와 같습니다.  
![/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%209.gif](/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%209.gif)  
_Pro 버전을 사용하지 않는다면 위 내용은 관계 없다._


## 3. Keen UI

![/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%207.jpg](/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%207.jpg)

- 공식 : [https://josephuspaye.github.io/Keen-UI/#/ui-alert](https://josephuspaye.github.io/Keen-UI/#/ui-alert)
- 깃허브 : [https://github.com/JosephusPaye/Keen-UI/tree/master#keen-ui](https://github.com/JosephusPaye/Keen-UI/tree/master#keen-ui)
- 라이센스 : MIT License

Vue Material Kit과 동일하게 Google 머티리얼 디자인에서 영감을 받은 구성입니다.

Keen UI는 패키지 전체에 사용할 필요 없이 필요한 부분에만 적용할 수 있는 구성 방식이라  
기존 서비스에 기능을 추가하고자 하는 경우 적합한 라이브러리입니다.


## 4. VuePress

![/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%208.png](/images/2020-02-15-Common-vuejs-uiComponentLibrary/Untitled%208.png)

- 공식 : [https://vuepress.vuejs.org/](https://vuepress.vuejs.org/)
- 깃허브 : [https://github.com/vuejs/vuepress](https://github.com/vuejs/vuepress)
- 라이센스 : MIT License

VuePress는 정적 사이트에 중점을 두어 UI 컴포넌트를 제공하는 점이 특징입니다.  
따라서 다이나믹한 화면이 요구되는 사용자 서비스가 아니라 기술 문서와 같이 내용이 핵심인 페이지에 적합합니다.

디렉토리 구조 가이드가 제공되어 개발자들간의 약속을 구성할 필요 없이 이를 표준으로 삼을 수 있습니다.


## 5. 기타 라이브러리
- Buefy ([https://buefy.org/#/](https://buefy.org/#/))
- Vux ([https://github.com/airyland/vux](https://github.com/airyland/vux))
- AT UI ([https://github.com/at-ui/at-ui](https://github.com/at-ui/at-ui))
- Element ([https://github.com/ElemeFE/element](https://github.com/ElemeFE/element))
- Eagle.js ([https://zulko.github.io/eaglejs-demo/#/](https://zulko.github.io/eaglejs-demo/#/))
- iView UI ([https://www.iviewui.com/](https://www.iviewui.com/))
- Mint UI ([http://mint-ui.github.io/](http://mint-ui.github.io/))

---
layout: post
title:  "React X Cypress 테스트 기초 셋팅 및 가이드"
description: 리액트로 개발된 프로젝트의 테스트 단계에서 유용하게 사용할 수 있는 테스트 프레임워크인 Cypress의 기초적인 사용 방법에 대해 알아봅니다.
date:   2021.07.14. 
writer: "반윤성"
categories: Common
---

## 소개 : Cypress 란?
리액트로 웹 어플리케이션을 개발하면서 디버거와 개발자 도구를 통해 기초적인 테스트를 진행했습니다.

화면 설계시 상, 하부 컴포넌트의 연결을 크게 고려하지 않고 작성했고, State와 Props의 전달 과정을 생략한
단편적인 형태의 테스트를 수행하다보니 실제 동작시 생각대로 움직이지 않는 어려움이 있었습니다.

이에따라 리액트와 연동하여 사용할 수 있는 테스트 프레임워크인 Cypress(싸이프레스)를 적용하게 되었습니다.
Cypress는 주로 프론트엔드의 E2E(End to End, 종단간 테스트) 시나리오를 수행하는 도구입니다. 개발을 진행하면서
표현되는 모든 화면의 테스트를 수행하고 브라우저를 통해 결과를 확인할 수 있습니다.

![/images/2021-07-14-Cypress/cypress-logo.png](/images/2021-07-14-Cypress/cypress-logo.png)

## 사용 방법
Cypress 설치 후 프로젝 테스트를 진행해 보겠습니다.

우선 리액트 프로젝트가 존재하는 곳에 다음의 명령어로 패키지를 설치합니다.

```jsx
npm install --save-dev cypress
```

한 줄의 명령어를 사용하여 Cypress와 테스트에 필요한 라이브러리를 받을 수 있습니다.

이후 프로젝트\cypress\integration 폴더를 작성하여 실제적으로 테스트를 수행하는 js 파일을 작성합니다.

혹시 jquery와 같이 쿼리를 통해 접근하는 문법을 사용해보셨다면, Query를 통해 DOM에 접근하고 있다는 사실을 파악하고
$(.class) 형태로 수월하게 작성할 수 있을 것입니다.

```jsx

# App.e2e.test.js

context("화면 테스트", () => {
 
  it("메인 로그인", () => {
    cy.visit("/");
    cy.get(".header").should("be.visible");
  });
 
  it("대시보드", () => {
    cy.visit("/dashboard");
 
    cy.get(".DetailPanel").should("be.visible");
    cy.get(".container").should("be.visible");
    cy.get(".container-text-lang").should("be.visible");
    cy.get(".container-text-zone").should("be.visible");
  });
 
  it("컨테이너 생성", () => {
    cy.visit("newContainer");
 
    cy.get(".loadingBar").should("be.visible");
    cy.get(".done-panel-header").should("be.visible");
    cy.get(".done-panel-body").should("be.visible");
    cy.get(".navigate-right-button").should("be.visible");
  });
});

```

화면 출력 테스트 파일입니다. 코드 흐름을 살펴보면 visit()는 실제 도메인에 접근하여 각 정의된 클래스와 

연결되는 should() 메소드를 통해 명령을 수행합니다. 위에서 부터 아래로 순서대로 테스트가 진행되면서, 

출력되는 결과가 Cypress 대시보드에 정리되어 결과를 확인할 수 있습니다.

```jsx

# package.json

"scripts": {
  ...
  "test:unit": "mocha --require babel-core/register --require ./test/helpers.js --require ./test/dom.js --require ignore-styles 'src/**/*.spec.js'",
  "test:unit:watch": "npm run test:unit -- --watch",
  "test:snapshot": "jest --config ./test/jest.config.json",
  "test:snapshot:watch": "npm run test:snapshot -- --watch",
  "test:cypress": "cypress open"
}

```

이어서 작성된 스크립트에 대해 실행할 명령어가 필요하기 때문에 package.json 부분을 수정하겠습니다.

테스트를 수행할때는 "npm run test:cypress"과 같이 명령어를 실행해주면 작성했던 테스트 파일이 실행됩니다. 


![/images/2021-07-14-Cypress/cypress-test.png](/images/2021-07-14-Cypress/cypress-test.png)

Cypress 사용하여 개발된 프로젝트의 테스트를 진행한 모습입니다. 각 페이지 별로 진행 상황을 파악할 수 있고
오류가 있을 경우 화면을 통해 로그를 표시해 줍니다. 이를 통해 조금더 명확하게 오류내용을 파악할 수 있는 장점이 있습니다.

화면에서 보이는 초록색으로 체크된 부분은 정상적으로 테스트 결과가 출력됐다는 뜻이고, 붉은색으로 표현된 부분은 오류가
발생한 부분입니다. 이 부분을 클릭하여 내용을 확인해보면 어떤 부분에서 에러가 발생했고, 어떤 문제가 있었는지 확인할 수
있습니다.


## 정리
우선 Jest와 같은 React에서 자체적으로 제공하는 테스트 프레임워크와 같은 경우는 어떠한 특정 시나리오에 맞춰

종단간 테스트를 진행하기에 제한되는 반면에 Cypress는 처음부터 E2E를 염두해 두고 출발한 모습입니다. 또한 좋은

디자인의 대시보드를 갖추고 있어서, 누구든지 직관적으로 화면을 통해 테스트 결과를 확인할 수 있다는 장점이 있습니다.

또한 프로젝트를 개발할 때 테스트에 중점을 두고 개발하는 방식도 생각해볼 수 있습니다. 프로젝트 개발과정의 각 단계마다

미리 테스트 문서를 작성해두면, 명령어 한 번으로 진행 과정을 파악해 볼수 있어서 개발 효율과 코드 품질 개선에 도움을 

줄 수 있을 것이라고 생각합니다. 추가적으로 브라우저에서 랜더링하며 비디오나 스냅샷을 찍어주는 기능도 개발되었다고 하는데

이 내용도 같이 확인해 보면 좋겠습니다.

## 참고 자료
- https://nx.dev/latest/react/cypress/overview
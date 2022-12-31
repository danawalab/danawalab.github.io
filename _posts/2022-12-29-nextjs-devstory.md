---
layout: post
title: "!프론트엔드 개발자의 next.js 개발기"
description: "!프론트엔드 개발자의 next.js 개발기 입니다."
date: 2022.12.29.
writer: "장민규, 이현우"
categories: Common
---

![!frontend](/images/2022-12-29-nextjs-devstory/0.png)   
출처 구글

# 소개

안녕하세요 장민규, 이현우 입니다.   
이번 포스팅은 평소와 다르게 두 명이서 찾아왔습니다, 저희 둘은 면접도 같이 보고 입사도 같이해서 같은 팀에 있는 둘도 없는 동기입니다.   
각자 개발한거는 다르지만 둘이서 비슷한 시기에 Next.js로 프론트를 개발하면서 막히는게 있으면 페어코딩도 하면서 문제를 해결하였습니다.   
그래서 둘이서 같이 포스팅 하기로 했습니다, 또한 기준의 블로그 포스팅을 방식이 아닌 QnA 형식으로 진행하고자 합니다.

## Q. 무슨 서비스를 개발했나요?
- 장민규 A. 저는 [is-deploy-console](https://github.com/danawalab/is-deploy-console)을 개발 하였습니다, 서비스에 대한 정보는 [아파치 톰캣 로드밸런싱 상태에서 WAS 무중단 자동 배포하기](https://danawalab.github.io/common/2022/12/21/is-deploy-part1.html)로 확인해 주세요.
- 이현우 A. 내부 트래픽 대용량 데이터에서 필요 내용들을 추출하고 분석하여 시각화하는 서비스를 개발하였습니다.   
Python을 통해 데이터 분석 및 추출을 진행하였고 시각화에서는 kibana, 파이썬 라이브러리 등 여러 툴을 사용했지만
원하는 모델을 찾기 어려워 시각화 관련 툴을 직접 개발하게 되었고, svg 파일을 통한 이미지와 Javascript를 통한 데이터 입력을 통해 구현하였습니다.   
후에 안정적인 서비스 운영과 생산성을 위해 Next.js를 이용하여 개발하였습니다.

## Q. 기술 스택은 어떻게 되나요?
- A. 제목과 같이 next.js를 사용했고 더불어 MUI도 이용해 개발하였습니다.

## Q. Next.js와 MUI 사용한 이유는 무엇인가요?
- A. 팀에서 프론트 개발을 react에 mui를 사용해 왔습니다, 그래서 Next.js가 React의 프레임워크로 React보다 편하게 동적 라우팅, SSR을 지원하고 팀이 프론트팀이 아니다 보니 생산성 문제 이유로 MUI를 사용했습니다.

## Q. 생산성을 위해 Next.js를 이용하셨다고 하는데 그럼 개발 기간은 어떻게 되나요?
- 장민규 A. 저는 기술검증부터 agent와 console까지 합치면 2달 반 정도 인데 console만 계산하면 1달 반정도 걸렸네요, 기능 구현보단 대부분 UI 구성하는데 시간이 많이 걸렸습니다.
- 이현우 A. next.js를 이용해 개발을 시작하고 나서는 한 달의 기간이 걸렸습니다.   
  중간에 요청사항으로 인해 데이터 셋이 변경되어 데이터 추출 부분을 변경하는 작업 때문에 시간이 더 지체되었던 것 같습니다.   
  또한 개발 완료된 초기 버전에서 더 필요한 요청 기능들을 받아 추가하며 개발하여 총 한 달의 시간이 소요되었습니다.

## Q.페어코딩은 어떻게 진행 했나요?
A. 자리 옆에 앉아 같이 코드와 로그를 보면서 무엇이 문제인지 빼먹은게 없는지 체크 하면서 진행했습니다, 아래의 사진처럼 메신저에 무언가 오면 도움을 요청 안 해도 옆자리로 넘아가 코드를 같이 봤습니다.

![3](/images/2022-12-29-nextjs-devstory/a.PNG)   
Elasticsearch를 통해 데이터를 받아 왔는데 데이터가 없어서 기쁜 현우 씨.png

![4](/images/2022-12-29-nextjs-devstory/b.PNG)   
데이터가 없어야 하는데 데이터가 들어갔다가 1초뒤 `undefined`가 나와서 기쁜 민규 씨.png

한 명은 데이터가 있어야 하는데 없고 한 명은 데이터가 없어야 하는데 있어서 기쁜 두 명이다.

## Q. 개발하면서 힘들고 오래 걸렸던 부분은 무엇인가요?
장민규 A. 저는 동적 ui 구성과 css가 힘들었습니다.

```jsx
return (
    <Grid container spacing={2}>
        {json.node.map((node, nodeIndex) => (
            nodeIndex === index ? node.podList.map((pod, podIndex) => (
                <Grid
                    key={pod} xs={12} md={6} xl={6}>
                    <Box
                        className={restore === true ? styles.box : excludeStatus === false ? styles.box
                            : podIndex === excludePodIndex ? styles.excludeBox : styles.box}
                    >
                        <Grid container>
                            <Grid xs={11}>
                                <div className={styles.podTitle}>
                                    {restore === true ? pod.name : excludeStatus === false ? pod.name
                                        : podIndex === excludePodIndex ? pod.name + " 제외되었습니다" : pod.name}
                                </div>
                            </Grid>
                            <Grid xs={1}>
                                <div className={styles.podTitle}>
                                    <Badge
                                        color={tomcatIndexLists.has(podIndex) ? 'error' : 'primary'}
                                        badgeContent={tomcatIndexLists.has(podIndex) ? 'OFF' : 'ON'}
                                        className={styles.badge}
                                    />
                                </div>
                            </Grid>
            ... 생략
```
코드를 간략하게 설명하면 json 설정에 따라 ui 구성이 동적으로 줄었다가 늘었다 하면서 agent 정보를 받으면 그에 맞게 ui 변경이 이루어져야 했습니다.   
또한 css에서 `float: left`를 넣었는데 왼쪽으로 안 간다거나 등 css 하면서 애를 먹었습니다.

![css](/images/2022-12-29-nextjs-devstory/10.gif)   
출처 구글

이현우 A. React 라이브러리를 사용해 본 적이 없어서 react 기본적인 사용법을 익히고,
Next.js의 ServerSide Rendering 등 프레임워크의 사용법을 익히는데 시간을 많이 사용했습니다.   
또한 아무래도 개발을 하다보면 필수적으로 마주하게 되는 디버깅이 힘들었습니다.   
변수의 상태변화, Elastic Cloud의 API를 통해 개발을 진행하면서 많은 에러를 마주하게 되었는데,
쉽게 디버깅 가능한 에러가 있는 반면 어떤 이유에서인지 모르겠는 에러에 대해 일일히 로그를 찍어가며
진행하며, web 개발자도구 콘솔과 ServerSide Rendering의 로그를 확인 하기 위해 IDE의 콘솔창을 번갈아가며 확인해 개발하였습니다.


![1](/images/2022-12-29-nextjs-devstory/3.PNG)
nextjs를 해서 아픈 곳도 혼돈하는 현우 씨

## Q.  아쉬운점이 있나요?
- 장민규 A. css 실력이 충분하고 미적 감각을 잃어버리지 않았다면 이쁜 UI를 만들지 않았을까 합니다 (예쁜 테마 만들어주세요...), 그리고 기능 구현 부분에만 테스트 코드 좀 대충 작성했는데 좀 더 정확하게 작성하고 UI에 대한 테스트 코드 작성하는 법을 몰라서 UI 개발은 라이브로 보면서 개발만 하고 끝냈는데 그 부분이 아쉽네요.
- 이현우 A. Next.js를 처음 사용하게 되어 코드 컨벤션이나 디렉토리 구성에 대해 미흡한 부분이 많은 것 같습니다.   
또한 초기 개발 단계에서 컴포넌트 분리를 진행하고 하지 않아 한 파일이 거대해지는 일을 겪고 난 뒤
리팩터링을 통해 컴포넌트의 분리, lib의 분리 등을 통해 파일 및 디렉토리를 정하였는데,
개발 초기 단계부터 이러한 방향성을 잡고 갔으면 더 빠르게 개발하고 클린한 코드가 될 수 있었을 것 같아 아쉬움이 남습니다.

![c](/images/2022-12-29-nextjs-devstory/2.PNG)   
css에 감동한 민규 씨

# 정리

장민규. 프론트 개발자분들은 프론트가 개발하면 바로 눈에 보여서 재밌다고 하는데 저는 반대로 안되는 게 눈에 보여서 화가 났습니다,
오히려 백엔드나 devops는 로그만 보면서 뭐가 문제인지 맞추는 소거법, 추리 게임과 같은 느낌이 나서 저는 오히려 백엔드랑 devops가 더 좋습니다.
여담이지만 다나와에 보드게임 동호회가 있는데 항상 소거법, 추리게임, 전략게임 에서 높은 승률을 보이고 있습니다.     

이현우. Next.js의 다양한 장점을 전부 사용하진 않았지만, ServerSide Rendering과 API Routes의 기능을 사용할 수 있다는 것이
제 프로젝트에선 큰 메리트였습니다.   
Python을 통해 데이터 추출 및 분석 그리고 ElasticCloud에 데이터 업로드를 진행하게 되는데
Next.js의 API Routes의 사용으로 따로 백엔드 API를 구축하지 않고 Next.js 프레임 워크 하나로 전부를 구축할 수 있는 것이 매우 큰 장점으로 느껴졌습니다.   
SSR을 통해 시각화 그래프 및 테이블 형식을 빠르게 구성할 수 있다는 점도 좋았습니다.   
웹 개발에 사용할 프레임 워크로 Next.js를 사용해 보시길 추천합니다.   

이상 '!프론트엔드 개발자의 next.js  개발기' 였습니다.   
감사합니다.

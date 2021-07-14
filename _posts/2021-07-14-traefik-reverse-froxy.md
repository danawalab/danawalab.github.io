---
layout: post
title:  "Traefik과 Docker를 활용한 Reverse Proxy 구축"
description: Traefik 기술을 적용하여 도커 컨테이너간의 Reverse Proxy 네트워크를 효율적으로 구축합니다.
date:   2021.07.14. 
writer: "반윤성"
categories: Common
---
## 소개 : Traefik, Docker 그리고 Reverse Proxy
우리가 누군가에게 채팅 메시지를 보낼때 서버의 이름과 주소를 몰라도 메시지는 정확하게 전달됩니다.
이렇게 통신의 중간점에서 연락책이 되는것이 바로 Proxy(중개 서버)입니다. Proxy는 네트워크의 중간에서 요청과 응답을 전송합니다. 

사내에서 프로젝트를 개발중에 사용자가 특정 동작을 하면 컨테이너를 생성하거나 삭제하는 등의 라이프 사이클 관리가 필요한 시스템이 있었습니다. 
초기에는 호스트 서버에서 랜덤한 포트를 골라 직접 컨테이너를 관리했는데 다만 누가 그 포트를 사용할 수도 있고, 사용을 원하는 경우가 발생했습니다.

이 문제를 해결하기 위한 방안으로 앞서 설명한 Proxy의 한 종류인 Reverse Proxy를 구축하여 풀어보고자 노력하였고 이 포스팅은 그 과정에 대해
다루고 있습니다. Reverse Proxy는 기본적으로 클라이언트 요청이 프록시가 끝단이 된다는 개념입니다. 요청은 프록시 앞까지 전달되고 이후의 내용은
중개 서버에서 본 서버끼리 통신하게 된다는 것이 주 골자입니다.

![/images/2021-07-14-Traefik/Traefik_proxy.png](/images/2021-07-14-Traefik/Traefik_proxy.png)
 
이렇게 네트워크가 통신하는 개념은 알수있었는데, 이것만으로는 복잡한 컨테이너를 관리하고 개념을 적용할만한 기술적인 난이도가 존재했습니다.
따라서 가능한 가시적이며 효율적인 도구를 찾게되었고 그것이 바로 Traefik이었습니다. 이 오픈소스 프로젝트는 탄생부터 시스템을 대신하여 요청을
수신하고, 구성 요소를 찾아내는 기술로 출발했습니다. Docker, AWS, Kubernetes와 같은 다양한 클러스터 기술들과 함께 사용할 수 있을 뿐만 아니라
별도의 제어가 필요없이 실행중에 실시간으로 통신되는 요소끼리 찾아서 연결해준다는 기능이 특징입니다.

실제로 Traefik과 Docker를 활용하여 개발을 진행했는데 docker.sock, label을 적절히 셋팅해주면 자동적으로 연결되는 것을 확인할 수 있었습니다.
혹시 초기 셋팅이 필요하다면 이곳에서 소개하는 예제를 활용하면 좋겠습니다.  

## Traefik 구성과 설명
이 문서에서는 Docker와 Traefik을 통해 Reverse Proxy를 구축합니다.

![/images/2021-07-14-Traefik/Traefik_process.PNG](/images/2021-07-14-Traefik/Traefik_process.PNG)

컨테이너 생성를 하고, 다시 Traefik으로 묶입니다. 이를 위해 우선 다음과 같이 도커 컴포즈 파일을 작성합니다.

```jsx
#docker-compose.yml
version: "3"

networks:
  web:
    external: true

services:
  traefik:
    image: traefik:alpine
    labels:
      - traefik.frontend.rule=Host:traefik.example.com
      - traefik.port=3333
      - traefik.enable=true

    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.toml:/etc/traefik/traefik.toml
    ports:
      - 3333:3333
      - 4444:4444
    networks:
      - web
```

```jsx
#traefik.yml
# defaultEntryPoints = ["http", "https"]

defaultEntryPoints = ["http"]
logLevel = "INFO"

[docker]
  endpoint = "unix:///var/run/docker.sock"
  exposedByDefault = false


# enabling api is not absolutely necessary, it is needed only if you need dashboard.
[api]
  dashboard = true
  entrypoint = "dashboard"


[entryPoints]
  [entryPoints.http]
  address = ":3333"

  [entryPoints.dashboard]
  address = ":4444"
```

이미지는 traefik:alpine을 사용합니다. 그리고 여기서 빌드된 이미지와 도커틀 연결하기 위해 labels, volumes, ports 등을
추가적으로 작성해줍니다.

하나씩 각 항목들을 살펴보면 이렇게 됩니다.

[도커 컴포즈 항목들]

- __labels__ : 컨테이너 연결부, 적용 여부, 연결할 포트
- __volumes__ : 도커로 연결할때는 다음과 같이 작성, Traefik 환경설정 파일 추가
- __ports__ : 80은 Traefik, 8080은 대시보드가 기본포트. 여기선 3333으로 묶임
- __networks__ : 네트워킹은 브릿지 모드 web (create network web으로 미리 생성)

## Traefik 동작과 대시보드 화면

위에서 작성한대로 파일을 생성했다면, 이제 실행하여 결과를 확인해볼 수 있습니다.

```jsx
docker run -l traefik.frontend.rule=PathPrefixStrip:/ -l traefik.backend=/ -l traefik.enable=true --network web -d nginx
```

간단한 방법으로 다음의 명령어를 bash에서 실행하여 nginx 이미지를 실행하고 어떻게 나오는지 확인해보겠습니다.

위의 명령어는 -v 옵션에서 html 파일 하나를 마운트하고 -l 옵션에서 경로, 포트, 적용 여부와 네트워크 명을 작성합니다.

이때 포트는 위에서 설정한 Traefik의 포트여야 합니다. 이제 명령어를 실행하겠습니다.

![/images/2021-07-14-Traefik/Traefik_sample.PNG](/images/2021-07-14-Traefik/Traefik_sample.PNG)

다음과 같이 화면이 출력되면 정상적으로 Reverse Proxy가 구성되어 연결된것입니다.

이 내용은 Traefik에서 자체적으로 제공하는 대시보드에서도 확인할 수 있습니다.

대시보드에서는 현재 연결된 도커 컨테이너와 주소를 비롯해 현재 상태와 검색 기능을 제공하고 있어서

한눈에 Traefik 작동 현황을 파악할 수 있어서 유용하게 사용할 수 있을것으로 보입니다.

![/images/2021-07-14-Traefik/Traefik_dashboard.PNG](/images/2021-07-14-Traefik/Traefik_dashboard.PNG)


## 정리
Reverse Froxy라는 개념이 생소하게 다가오기도 했지만 Traefik이라는 유용한 툴을 사용하여 생각보다 쉽게 프로젝트에 적용할 수 있었습니다.
또한 자체적으로 대시보드나 관리까지 제공하고 있어서 처음 사용하기에도 수월했습니다. 다만 아직 감춰진 좋은 기능이 많을것으로 생각하는데 
관련된 문서가 많지 않아서 레퍼런스할 내용이나 프로젝트가 많지 않았습니다.

한번 더 프로젝트에 Traefik을 적용해본다면 현재 영어로 작성된 공식 문서의 내용을 차근차근 살펴봐서 어떤 기능이 존재하는지 파악해봐도 
좋겠다는 생각이 들었습니다. 혹시 Traefik을 통해 프로젝트를 개발하고 계신다면 이 문서가 도움이 되었으면 좋겠습니다. 

## 참고 자료
- https://doc.traefik.io/traefik/
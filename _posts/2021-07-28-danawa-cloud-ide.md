---
layout: post
title:  "DANAWA CLOUDE IDE 알아보기"
description: 독립된 가상 환경에서 개발할 수 있는 IDE를 제공합니다.
date:   2021.07.27. 
writer: "반윤성"
categories: Common
---

| Danawa Cloud IDE 메인 | Danawa Cloud IDE 터미널 |
|--------|--------|
|![/images/2021-07-27-Danawa-Cloud-Ide/ide_main.PNG](/images/2021-07-27-Danawa-Cloud-Ide/ide_main.PNG)|![/images/2021-07-27-Danawa-Cloud-Ide/Ide_container.PNG](/images/2021-07-27-Danawa-Cloud-Ide/Ide_container.PNG)|

#

## Danawa Cloud IDE
개발 하면서 종종 `가상 IDE`가 필요한 경우가 있습니다. 보통 로컬 시스템과 격리된 공간에서 작업을 해야할 때라던지 현재 시스템과 다른 운영체제를 사용하여 개발할 때 보통 가상화 도구를 사용하기도 하지만 번거롭기도 하고 원인모를 오류가 생기기도 합니다.
   
마이크로소프트에서 개발한 `코드 서버(code-server)`가 있습니다. 비주얼 스튜디오 코드를 통째로 오픈 소스로 만들어 제공한것으로 개발사에서 지속적으로 유지보수도 제공하는 유용한 도구입니다. 이 기술을 웹 서버에 올려서 사용한다면, 언제 어디서든지 사용자가 원하는 개발 공간을 제공할 수 있습니다.

`Danawa Cloud IDE`는 여기서 출발했습니다. 어떤 웹페이지 안에서 코드를 만들고 고치는 작업 공간입니다. 이곳에서 내 환경과 독립적으로 개발환경을 운용하여 편리하게 사용할 수 있습니다. 최종적으로 개발자들의 편리성을 도모하기 위해 개발되었습니다.  

## 어떻게 만들었을까?

DANAWA-CLOUD-IDE의 시스템 구성은 다음과 같습니다.


![/images/2021-07-27-Danawa-Cloud-Ide/Ide_system.png](/images/2021-07-27-Danawa-Cloud-Ide/Ide_system.png)

##### DANAWA-CLOUD-IDE 시스템 구성도

본 시스템은 기본적으로 자바스크립트 라이브러리 React와 Node.js 서버 환경에서 구동됩니다. 추가적으로 다양한 오픈소스 라이브러리를 활용하여 제작되었습니다. 패키지 매니저인 Yarn을 사용하여 이러한 라이브러리를 관리하며, Express나 mysql, 도커와 같은 도구도 사용합니다.

사용자가 UI를 통해 특정 기능을 호출하면 서버쪽에서 이를 수신해서 Express를 사용해 DB와 연결합니다. 이어서 도커도 호출하는데, 이는 터미널 기능인 Code-Server 이미지를 빌드하여 사용하기 때문입니다. 생성된 컨테이너의 ID를 전달받아서 오픈할 때 도커를 통해 만들어진 터미널 컨테이너로 접속하게 됩니다.

<br>

## 핵심 개발내용 정리

이제 중요한 내용인 `코드서버 활용법`과 `URI을 통한 Docker API호출`, `Docker와 Traefik을 이용한 리버스 프록시 구축 방법`에 대해 다뤄 보겠습니다.

우선 구현을 위해 코드서버 프로젝트가 필요합니다.


```dockerfile
FROM ubuntu:18.04
COPY . /app

# code-server
WORKDIR /home/danawa/.code-server
RUN wget https://github.com/cdr/code-server/releases/download/3.2.0/code-server-3.2.0-linux-x86_64.tar.gz
RUN tar xzf code-server-3.2.0-linux-x86_64.tar.gz -C ./ --strip-components 1

WORKDIR /home/danawa

CMD ["/bin/bash", "-c", "& /home/danawa/.code-server/code-server --port 3333 --host 0.0.0.0 --auth none"]
```

코드서버(3.2.0 버전 기준)를 받아서 실행하는 내용입니다.

추가적으로, 오픈하고 바로 JAVA 패키지를 실행할 수 있게 설정합니다.

이를 위해 아래 내용을 수정 및 추가해줍니다.

```dockerfile
...
FROM openjdk:11
...

CMD ["/bin/bash", "-c", "/home/danawa/.code-server/code-server --install-extension vscjava.vscode-java-pack --force && /home/danawa/.code-server/code-server --port 3333 --host 0.0.0.0 --auth none"]
```

이제 JDK와 플러그인이 탑재된 IDE를 사용할 수 있습니다.  

여기까지 진행하면 터미널을 오픈할 수 있지만, 클릭 한 번으로 생성할 수 있도록 다음과 같은 코드를 작성합니다. Docker Api를 통해 URL로 호출하는 방식을 사용합니다.

```jsx
// 신규 터미널 컨테이너 생성 기능입니다.
async function createContainer(user_id, key, state) {
  let c_id;
  let pickImage = state.imageClicked;
  let useMysql = state.pkg_1;

  try {
    let newContainer = await axios({
      method: "post",
      url: "/containers/create",
      data: {
        Hostname: "test",
        Image:
          pickImage === "java"
            ? "dcr.danawa.io/java_spring_vscode:latest"
            : "dcr.danawa.io/nodejs_vscode:latest",
        ExposedPorts: {},
        Labels: default_label, // traefik 연동을 위한 라벨링
        HostConfig: {
          Binds: [],
          NetworkMode: "web",
          Privileged: true
        },
        NetworkingConfig: {
          EndpointsConfig: {
            web: {
              IPAMConfig: {},
              Links: [],
              Aliases: [],
            },
          },
        },
      },
    });
      

      // 컨테이너 실행 기능입니다.
      await axios({
        method: "post",
        url: "/exec/" + data.data.Id + "/start",
        data: {
          Detach: true,
          Tty: false,
        },
      });
    } else {
      console.log("컨테이너가 시작되지 않았습니다.");
    }
  } catch (e) {
    console.log(e);
  }
  return c_id;
}
```

이 길다란(?) createContainer() 메소드는 docker create api를 호출하기위한 메소드입니다. POST방식으로 컨테이너 생성에 필요한 인자(이미지, 네트워크 구성, 호스트)를 도커에게 전달하면 컨테이너가 만들어지고, 고유한 값인 컨테이너 ID를 반환받습니다.

정리해보면 생성 버튼 클릭 -> create api 호출 → 컨테이너 생성 → 컨테이너 실행 → DB에 값 저장순으로 진행됩니다. 이제 서비스에서 필요한 기능인 가상 IDE 생성 기능을 사용할 수 있습니다.

## 리버스 프록시도 같이 적용해보기

가상 IDE를 만들고 사용하는 것은 위에서 설명드렸던 내용으로도 충분히 가능합니다. 하지만 리버스 프록시를 적용하면 포트 관리를 효율적으로 할 수 있고, 서브도메인, 터미널에서 구동한 프로젝트 실행 확인을 해볼 수 있어서 좋습니다. 전에 작성했던 내용을 참고 했습니다. [Traefik과 Docker를 활용한 Reverse Proxy 구축](https://danawalab.github.io/common/2021/07/14/traefik-reverse-froxy.html)

우선 기능 구현을 위해 traefik이 필요합니다.

![/images/2021-07-27-Danawa-Cloud-Ide/ide_traefik.PNG](/images/2021-07-27-Danawa-Cloud-Ide/ide_traefik.PNG)

##### traefik 시스템





traefik은 최근에 부상한 모던 HTTP 리버스 프록시 기술입니다. 도커나 쿠버네티스와
같은 다양한 환경과도 맞물려서 사용할 수 있고 간편하게 구현이 가능해서 처음 사용하는 사람도 다룰 수 있게 되어있습니다.

또한 자체적으로 대시보드를 제공하고 있어서 네트워크가 어떻게 연결되어있는지 한눈에 파악할 수 있어서 개발 과정마다 상태를 파악하기 수월하다는 특징이 있습니다.

이를 구현하기 위한 준비물은 도커 파일과 약간의 코드 수정입니다.

```yml
#docker-compose.yml(Traefik용)
version: "3"

# 네트워크 연결
networks:
  web:
    external: true

services:
  traefik:
    # traefik 이미지
    image: traefik:alpine
    labels:
      - traefik.frontend.rule=Host:traefik.example.com
      - traefik.port=3333
      - traefik.enable=true
    # 도커로 사용하기위해 docker.sock연결
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.toml:/etc/traefik/traefik.toml
    # 트래픽 포트, 트래픽 대시보드 포트
    ports:
      - 3333:3333
      - 5555:5555
    networks:
      - web
```

```toml
#traefik.toml(traefik 설정파일)
defaultEntryPoints = ["http"]
logLevel = "INFO"

[docker]
  endpoint = "unix:///var/run/docker.sock"
  exposedByDefault = false


[api]
  dashboard = true
  entrypoint = "dashboard"

# 대시보드와 트래픽 포트설정
[entryPoints]
  [entryPoints.http]
  address = ":3333"

  [entryPoints.dashboard]
  address = ":5555"
```

트래픽 이미지를 통해 도커 컴포즈를 실행하고 트래픽과 같은 포트에서 IDE를 사용할 포트를 도커 커테이너 라벨링을 통해 연결해주면 됩니다. 라벨이 쓰여있으면 traefik 쪽에서 이를 인식하여 리버스 프록시가 연결되고 동일한 포트에 자동으로 묶입니다.

이를 위해 서비스 쪽에서는 컨테이너를 생성할 때 라벨부분에 다음과 같이 설정해주면 됩니다. 지정 포트에서 연결되고 traefik을 적용한다는 옵션 값이 들어있습니다. 이때 주소는 임의로 지정한 것인데 서비스에서는 [유저 아이디]-[고유 해시값].es2.danawa.io 패턴으로 구성되어 있습니다.


```jsx
async function createContainer(user_id, key, state) {
...
Labels: {
      "traefik.code-server.frontend.rule": "HostRegexp:es2.danawa.io,{subdomain:" + user_id + "-" + key + "}.es2.danawa.io",
      "traefik.code-server.port": "3333",
      "traefik.enable": "true",
      "traefik.passHostHeader": "true",
  }
...
}
```



## 정리하기
코드서버(code-server)라는 유용한 오픈소스와 도커를 활용하여 가상 IDE환경을 개발해봤습니다. IDE 안에서 내 로컬 환경에 구애받지않고 원하는 작업을 할 수 있다는 것은 매우 매력적인 일이라는 생각이 들었고 추가적으로 다른 기술과 접목하면 서비스를 개발하는데 있어서 효율적인 도구가 될것이라고 생각합니다.   

초기에는 단순히 웹페이지를 통해 가상 환경을 제공하는 서비스로 시작했습니다. 현재는 꽤 프로젝트가 커져서 회원가입과 로그인을 하기도 하고 traefik과 리버스 프록시를 적용해서 서브도메인과 연결하는 기능도 추가되었습니다. 프로젝트를 개발하면서 사용자에게 어떤 기능이 필요할지도 생각해보게 되었고 의미있는 시간이 되었던것 같습니다.


## 참고 자료
- https://docs.docker.com/engine/api/v1.41
- https://doc.traefik.io/traefik/
- https://github.com/cdr/code-server
---
layout: post
title: "아파치 톰캣 로드밸런싱 상태에서 WAS 무중단 자동 배포하기"
description: "L4로 묶인 아파치 톰캣 로드밸런싱 환경 WAS 무중단 자동 배포하는 방법을 알아보겠습니다"
date: 2022.12.21.
writer: "장민규"
categories: Common
---

# 개요

다나와에서는 일부 레거시 서비스는 L4로 로드밸런싱이 구성됐습니다.   
그래서 해당 서비스들을 배포하기 위해서는 서비스 담당 개발자들이 지라 이슈를 통해 배포 요청을 하면 시스템 관리팀이 배포 작업을 진행했는데    
배포 작업을 하는데 하루 일과에 오전 시간을 전부 사용하다 보니 다른 업무에 집중할 시간이 부족했고 개발자 입장에서는 배포마다 지라를 통해서 요청해야 하다 보니     
개발자와 시스템 관리팀 둘 다 불필요한 시간과 작업을 진행해야 하는 불편함이 있었습니다.

그래서 이 문제를 해결하기 위해서 개발하게 된 게 is-deploy입니다.

# is-deploy 

프로젝트를 시작하기 앞서 프로젝트 네이밍을 어떻게 할까 고민하다 WAS 배포를 하는 건데 WAS를 순수 영어 단어로 봤습니다.   
그래서 **was**가 **is**의 과거형이다 보니 현재형으로 바꾸고 배포하는 시스템이니 **deploy**를 붙여 **is-deploy**라는
네이밍을 정했는데 정하고 보니 **is deploy**를 직역하면 **배포 중입니다** 라는 뜻을 가져서 뭔가 배포 관련된 네이밍을 딱 가지게 되어 프로젝트를 시작했습니다.

is-deploy는 아래와 같은 구조를 가지고 있습니다.

![structure](/images/2022-12-21-is-deploy-part1/3.png)   

어떻게 하면 아파치 종료 없이 동적으로 로드밸런싱 상태에서 배포할 톰캣만 제거하여 무중단 배포를 할 수 있을까 고민을 했습니다.
그렇게 아파치와 톰캣 공식문서를 뒤지다 아파치 공식문서에서 **JkMountFileReload** 항목을 보게 되었습니다.

JkMountFileReload가 uriworkermap.properties을 동적으로 재로드 하는 것을 확인했고   
로드밸런싱 된 상태에서 uriworkermap.properties을 통해서 동적으로 톰캣을 뺄 수 있을 거라 생각했고 개발하기전에 먼저 기술 검증부터 진행 했습니다.    
로컬에서 윈도우에서 아파치 톰캣, wsl에서 도커를 통해 ubuntu를 띄우고 안에서 아파치 톰캣 환경을 구성하여 진행한 결과 두 환경다 성공적으로 기술 검증에 성공하여
개발에 들어갔습니다.   

**기술 검증 영상**   

[![기술검증](https://img.youtube.com/vi/XbpoWINb3CY/0.jpg)](https://youtu.be/XbpoWINb3CY)   

Q. jkstatus를 설정하면 되는 거 아닌가요?   
A. jkstatus를 설정하면 누구가 접속이 가능해 보안에 매우 취약하는 문제가 있었습니다.


# Agent 및 uriworkermap.properties 세팅

들어가기 앞서 아파치 경로와 설정은 다 다를 것이라 생각합니다 서버 운영체제와 아파치 버전에 따른 차이도 있을 거고 설정도 어떻게 했느냐에 따른 차이도 있을 것입니다.

대부분 리눅스 서버에 아파치 경로는 `/etc/apache2` 또는 `/etc/httpd`로 구성되어 있을 거라 생각 듭니다.

저는 우분투 서버에 진행했기에 아파치는 `/etc/apache2` 경로에 존재했었고 그 외 설정에 따라 경로가 다를 텐데
예를 들어 `/etc/apache2/conf`나 `/etc/apache2/conf.d` 경로에 config 파일 있을 텐데 저 같은 경우 `/etc/apache2/conf.d`로 설정되어 있었습니다.

일단 `/etc/apache2/conf.d` 경로에 `workers.properties`를 수정해 주겠습니다.

```properties
worker.list=worker1,worker2,balancer,balancer_ex1,balancer_ex2
 
worker.balancer.type=lb
worker.balancer.balance_workers=worker1,worker2

######################################################
# 톰캣이 제거된 로드 밸런스를 추가 해줍니다
# Q. balancer_ex1이 worker2만 보는거면 해당 부분은 필요 없는거 아닌가요?
# A. 네 톰캣이 2개면 해당 부분은 없어도 괜찮지만 톰캣이 3개 이상이라면 필요합니다! 

worker.balancer_ex1.type=lb
worker.balancer_ex1.balance_workers=worker2
 
worker.balancer_ex2.type=lb
worker.balancer_ex2.balance_workers=worker1
######################################################

worker.worker1.reference=worker.template
worker.worker1.host=localhost
worker.worker1.port=8009

worker.worker2.reference=worker.template
worker.worker2.host=localhost
worker.worker2.port=8109

worker.template.type=ajp13
worker.template.socket_connect_timeout=5000
worker.template.socket_keepalive=true
worker.template.ping_mode=A
worker.template.ping_timeout=10000
worker.template.connection_pool_minsize=0
worker.template.connection_pool_timeout=600
worker.template.reply_timeout=300000
worker.template.recovery_options=3
```

다음으로 `vhost.conf` 파일을 수정 하겠습니다.

```conf
Listen 80
  
<VirtualHost *:80>
        ServerName   deploy.danawa.com        
        
        ######################################################
        # 원래라면 JkMount balancer 를 보고 있겠지만 해당 부분을 동적으로 하기 위해
        # JkMountFile /home/conf/uriworkermap.properties 지정합니다
        # JkMountFileReload 5 는 uriworkermap.properties가 변경이 되면 아파치가 5초마다 동적으로 리로드 합니다
        # 또한 JkMountFileReload 를 따로 정의를 안 하면 기본 60초마다 동적으로 리로드 합니다, 0을 줄 경우 동적 리로드 기능을 끕니다 
        
        JkMountFile /home/conf/uriworkermap.properties
        JkMountFileReload 5
        ######################################################
  
        CustomLog      로그경로
        ErrorLog       로그경로
</VirtualHost>
  
<VirtualHost *:80>
        ServerName   deploy1-node1.danawa.com
  
        JkMount /* worker1
  
        CustomLog       로그경로
        ErrorLog        로그경로
</VirtualHost>
  
<VirtualHost *:80>
        ServerName   deploy1-node2.danawa.com
  
        JkMount /* worker2
  
        CustomLog       로그경로
        ErrorLog        로그경로
</VirtualHost>
```

`vhost.conf` 파일까지 수정을 마쳤다면 `/home/conf/` 해당 경로에 `uriworkermap.properties` 파일을 만들겠습니다.

Q. 왜 `/etc/apache2/conf.d` 에 `uriworkermap.properties` 안 만드나요?   
A. 이에 대해서는 다음에 포스팅에서 자세히 다루도록 하겠습니다.

```properties
/*=balancer
```

이제 마지막으로 agent를 설치하여 실행하면 끝입니다.

[is-deploy-agent](https://github.com/danawalab/is-deploy-agent) 에 가서 Releases에 빌드 된 파일을 다운로드해 서버에 넣어 줍니다. 

그리고
```shell
chmod +x is-deploy-agent

# 뒤 5000은 실행할 포트 입니다.
./is-deploy-agent 5000
```
권한을 주고 agent를 실행해 줍니다.

# Console 세팅

관리 도구인 is-deploy-console은 세팅 방법이 두가지 있습니다.

도커를 통해 실행하는 방법과 직접 코드를 다운로드해서 직접 실행하는 방법입니다.

먼저 도커를 통해 설치하는 방법입니다.

```shell
# 먼저 docker pull을 통해 이미지를 받습니다.
docker pull ghcr.io/danawalab/is-deploy-console:latest

# 다음 실행을 해줍니다 console 은 3000번 포트를 열려 -p 를 통해 포트를 지정해 주고 
# 화면을 동적으로 구성하는데 json을 통해서 하고 있습니다 해당 경로는 /home/is-deploy-console/config여서 -v 를 통해 volume을 지정해 줍니다
docker run -p 3000:3000 -v /home/is-deploy:/home/is-deploy-console/config ghcr.io/danawalab/is-deploy-console:latest
```

다음 두 번째는 직접 코드를 받아서 하는 방법입니다.

```shell
npm run build

npm start

또는

yarn build

yarn start
```

통해서 세팅해 주면 됩니다.

console 사용법은 영상으로 대체하겠습니다

[![유튜브](https://img.youtube.com/vi/CTDLOm4-Guo/0.jpg)](https://youtu.be/CTDLOm4-Guo)


# TO-BE

지금 버전은 console에서 ip를 통해 누가 해당 작업을 했는지 로그를 남겨서 확인하고 있는데   
2.0.0 버전에는 로그인과 OTP 기능을 추가함으로 어느 유저가 어느 작업을 했는지 로그를 남길 예정입니다.    
그리고 로그인 기능을 통해 담당 개발자 아니면 배포 작업을 못 하게 제한을 할 예정입니다.   

![업데이트](/images/2022-12-21-is-deploy-part1/2.png)   

또한 위 사진과 같이 agent 동적으로 업데이트 및 다운그레이드 기능을 제공할 예정입니다.

# 정리 

is-deploy를 도입하면서 서비스 담당 개발자는 지라 이슈를 통해 배포 요청을 안 하고 is-deploy를 통해 버튼 클릭으로 아파치 톰캣 로드밸런싱 상태에서 원하는 톰캣을 제거하고
해당 톰캣에 새 버전의 서비스 무중단 배포를 진행하면서 문제없는지 실시간으로 톰캣 로그를 보는 게 가능합니다.   
그뿐만 아니라 시스템 관리팀은 배포 작업을 안 해도 되어 그만큼의 시간을 다른 업무에 집중할 수 있다는 장점이 있습니다.

여러분들도 is-deploy를 도입하여 사용해 보시는 거 어떠신가요?

저는 is-deploy를 개발하면서 아파치와 톰캣이 얼마나 잘 만들었는지 세상 한번 느끼게 됐습니다.

-----
참고:
- https://github.com/danawalab/is-deploy-agent
- https://github.com/danawalab/is-deploy-console
- https://tomcat.apache.org/connectors-doc/reference/apache.html
- https://tomcat.apache.org/connectors-doc/reference/uriworkermap.html
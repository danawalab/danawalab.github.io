---
layout: post
title:  "Github Package (Container-Registry) 사용법"
description: "github에서 Container, RubyGems, npm, Apache Maven, Gradle, NuGet package registry를 제공하고 있습니다. 저희는 Container Registry를 알아보도록 하겠습니다."
date:   2021-12-27
writer: "김준우"
categories: Common
---
## 개요

github에서 Container, RubyGems, npm, Apache Maven, Gradle, NuGet package registry를 제공하고 있습니다.

저희는 Container Registry만 알아보도록 하겠습니다.

- 최근 깃헙 docker registry 서비스에서 container registry 로 변경되었고, 설명은 container registry로 하겠습니다.

## 사용법

**과정**

1. 사용자 토큰 발행
2. nginx 이미지 push/pull 확인
3. package 이미지 관리

### **사용자 토큰 발행**

현재 저희는 danawalab 기업형 계정을 통해 개인 깃헙 주소와 연결되어 있습니다.

최근 깃헙에서는 비밀번호사용을 없애고 ssh 또는 token을 발생받아 사용하도록 변경되었습니다. 개인 깃헙계정으로 토큰을 생성합니다.

접속링크: [https://github.com/settings/tokens](https://github.com/settings/tokens)

Generate new token을 클릭합니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled.png)

클릭을 하게 되면 비밀번호를 입력 후 아래 폼화면으로 이동하게 됩니다. 토큰의 이름을 입력하고, 토큰의 만료기간을 설정합니다.

이번에 wokrflow, package를 사용예정이기 아리 그림 처럼 체크 항목을 체크합니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%201.png)

완료가 되면 아래 그림처럼 ghp_ 시작하는 토큰을 받을 수 있습니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%202.png)

package의 주소는 ghcr.io 입니다. 로그인을 시도 해봅니다.

아이디: 개인 깃헙 이메일 또는 사용자명입니다.

비밀번호: ghp_ 시작하는 토큰입니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%203.png)

Login Succeeded 가 나오면 정상입니다.

### **이미지 push / pull 해보기**

기존 dcr.danawa.io/dsearch-server 이미지를 받습니다. 

다나와에서 오픈소스로 공개 중인 dsearch-server로 시험해보았습니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%204.png)

dcr.danawa.io/dsearch-server 이미지를 ghcr.io/danawalab/dsearch-server 태그 후 푸시 합니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%205.png)

깃헙에서 package 탭을 보면 dsearch-server 가 생성된걸 알 수있습니다.

링크: [https://github.com/orgs/danawalab/packages](https://github.com/orgs/danawalab/packages)

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%206.png)

로컬에 해당 이미지를 삭제합니다. 그리고 다시 pull  받아 확인해보았습니다.

648MB 정상적으로 다운받았습니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%207.png)

* 확인사항

1. docker push 후 github package에서는 약 2~3분 뒤에 표시될때가 있습니다. docker pull은 정상적으로 사용이 가능했습니다.

### **package 이미지 관리**

**프로젝트 연결**

조건:

- 프로젝트 이름과 동일
- dockerfile 라벨 명시 (LABEL org.opencontainers.image.source [https://github.com/](https://github.com/)OWNER/REPO)

또는 아래 이미지 처럼 connect Repository 를 선택합니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%208.png)

connect repository 클릭하게 되면 해당 프로젝트에 Packages 에 이미지 링크가 추가 된걸 확인할 수 있습니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%209.png)

Package 페이지에서 우측메뉴에는 작은 package settings을 통해 패키지를 삭제하거나 공개설정을 할 수 있습니다. 링크는 프로젝트의 우측 하단에 위치합니다.

![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/Untitled%2010.png)

여기까지 github pakcage 사용방법입니다.

docker hub와 사용방법이 많이 유사한걸 알 수 있습니다.

그리고 깃헙 패키지는 오픈소스에 대해서는 제한없이 무료이용 가능하며, 비공개시 아래 그림처럼 구독을 통해 제한적으로 사용할 수 있습니다.
![Untitled](/images/2021-12-27-Github-Package-(Container-Registry)-사용법/2021-12-27_18h07_22.png)



참고링크

- [https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [https://github.com/orgs/danawalab/packages](https://github.com/orgs/danawalab/packages)
---
layout: post
title: "CI/CD에 기존 Provisioning Profile 유지하기(Git으로 유지하기)" # 리스트에 보여질 제목
description: "CI/CD를 적용하면서 겪은 Provisioning Profile을 유지하는 방법에 대해서 고민하고 해결한 경험을 서술합니다." # 리스트에 보여질 설명
date: 2025.08.04
writer: "최인호" 
categories: iOS 
---
# provisioning profile 관리하기

기존 다나와 서비스는 개발자 각자의 `develop`용 `provisining profile`과 `distribution`용 `provisining profile`을 갖고 있었습니다.

왜냐하면 다나와 서비스가 ConnectWave로 합류하게 되면서 ConnectWave iOS 개발자 계정 내에 많은 앱들과 함께하게 되었거든요.

그래서 다음과 같은 제약사항이 있었습니다.

> Distribution 용은 서비스(앱) 당 1개만 유지할 것.
> 

# 어떻게 기존 1개를 관리할 것인가?

## 1개만 유지하자!

다나와 iOS 개발자분들은 모두 각자의 develop용 Provisioning Profile을 갖고 있지만, distribution은 같은 파일로 가지고 있습니다.

그래서 CI/CD를 구성하면서 `CI/CD기기에 어떻게 Provisioning Profile을 관리할까?` 가 문제의 요점이었습니다.

### 1. 기기에서 갖고 있게 하자.

CI기기(이하 맥미니)에 처음에 절대경로로 프로비저닝 프로파일을 갖게 하고자 했습니다.

하지만 만약 CI기기가 어떤 계기로 인해서 다른 기기로 바뀌게 된다면?

다시 CI기기에 Jenkins를 설정해줘야하는 불편함이 예상되었습니다.

### 2. Jenkins가 갖고 있게 하자.

Jenkins에 파일을 등록하고 Ceriticate 파일로 관리하는 방법을 고려했습니다.

Jenkins에 파일을 관리할 수 있는 파일을 쉽게 관리할 수 있는 서드파티가 존재하지만 아쉽게도 바이너리파일은 업로드가 할 수 없어서 Certiciate로 관리하는 방안을 고려했습니다.

하지만 이 또한 문제가 생겼습니다.

저희 ConnectWave 통합App개발팀은 크게 서비스가 2가지로 나눠져서 서로 다르게 망을 사용 중에 있습니다.

크게 `A 네트워크 망(Jenkins 기기가 속해있는 망)` `B 네트워크 망(다나와 서비스 개발하는 망)`으로 분리하겠습니다.

A망에서 개발하는 팀원분들은 내부 IP를 통해서 Jenkins에 접근이 가능했습니다.

하지만 B 네트워크 망에 속해 있는 저는 내부 IP를 통해서 Jenkins 접근이 불가능 했습니다.

그 말은 Jenkins를 원격으로 관리 할 수 없다는 뜻입니다.

(1)번 방법보다는 기기에 의존하지 않아 나은 방법이라 생각했지만, Jenkins를 원격접격할 수 없기에 문제가 생겼을 때 제가 컨트롤 할 수 없는 범위에서 벗어나기에 이 방법보다 더 나은 방법을 찾고자 했습니다.

### 3. match를 통해서 git으로 프로비저닝 프로파일을 관리하자.

fastlane에는 match라는 명령어가 있음을 발견했습니다.

[Code Signing Guide for Teams](https://codesigning.guide/)

주된 사용처를 보니 해당 명령어를 통해서 Provisioning Profile 및 Ceritication 생성 및 Git 업로드가 해주는 명령어 였습니다.

하지만 기존에 갖고 있는 Provisioning Profile 도 업로드가 가능해보였습니다.

그래서 제가 이루고자 하는 조건을 충족하기에 해당 방법을 통해서 Provisioning Profile을 관리하고자 했습니다.

1. 컨트롤 할 수 있을 것
2. 기존에 있는 Provisioning Profile을 사용할 것

# provisioning profile git에 올려 유지하기.

1. 파일을 암호화 해서 match 저장소에 저장하기

```bash
fastlane match import --type {타입} --username {Email} --team_id {팀 아이디}
```

type에는 여러가지가 올 수 있습니다. ( appstore, adhoc, developmnt, enterprise … 등 )

하지만 이번엔 appstore로 해봅시다.

match 명령어를 수행을 위해서는 크게 3가지가 필요합니다.

`.cer` `.p12` `.mobileprovision or .provisionprofile`

![image0.jpg](/images/2025-08-04-iOS-cicd-provisioning-profile/image0.jpg)

그러면 안내에 따라서 3개 파일의 절대경로를 하나씩 올립시다

![image1.png](/images/2025-08-04-iOS-cicd-provisioning-profile/image1.jpg)

그러면 이 파일들을 저장할 git 주소를 올려야합니다. 해당 파일을 유지할 레파지토리를 만들어서 링크를 올립니다.

![image2.jpg](/images/2025-08-04-iOS-cicd-provisioning-profile/image2.jpg)

그러면 암호를 적으라고 나옵니다.

match는 암호화해서 git에 올리고 fastlane에서 다운로드 받을 때도 암호를 통해서 자동으로 복호화 후 저장하게 됩니다. 그러니 잊어먹지 말고 반드시 기억해야 합니다.

만약에 `p12` 파일이 이미 암호화되어 있다면 이 과정은 Skip됩니다. 비밀번호는 복호화하는데 사용되니 잊어먹지 말아야 합니다.

![image3.png](/images/2025-08-04-iOS-cicd-provisioning-profile/image3.png)

해당 작업이 끝나면 위와 같이 레포가 만들어짐을 알 수가 있습니다.

# provisioning profile git에 있는 것 내려서 사용하기

이제 Fastfile을 수정하면서 fastlane이 우리가 의도한 대로 수행할 수 있도록 해봅시다.

[match - fastlane docs](https://docs.fastlane.tools/actions/match/)

### 앞서.

 프로비저닝 프로파일을 파일을 다운 받기 위해서 우리는 다음을 정하면 됩니다.

1. 무슨 타입으로 다운 받을 것인가?
2. Git URL
3. 어느 branch에서 다운로드 받을 것인가?
4. 어디에 저장할 것인가?

### 1. 무슨 타입으로 다운 받을 것인가?

우리가 아까 `fastlane match import`에서 정했던 type을 지정해주면 됩니다.

### 2. Git URL

우리가 이전에 만들었던 Git URL을 적어 올리면 됩니다.

### 3. 어느 branch에서 다운로드 받을 것인가?

우리가 왜 Distribution용 Provisiong profile을 1개만 유지하려는 이유는 하나의 엔터프라이즈 계정에 많은 앱이 있기 때문입니다.

무한정 Provisoing Profile을 늘릴 수 없는 상황에서 하나의 레포에 다양한 Provisioning Profile을 유지할 수 있는 방법은 하나의 Git Repo에 다양한 branch로 관리하는 방법입니다.

DanawaPC에 해당하는 branch인 `danawapc`로 branch로 만들고 관리합시다.

### 4. 어디에 저장할 것인가?

우리가 여태까지 달려온 이유입니다.

해당 Provisioning Profile을 어디에 다운로드 받을 것인가를 정해야 합니다.

fastlane과 관련되어 있고 매 CI/CD가 작동할 때마다 다운로드 받을 것이니

`./fastalane`에 다운로드 받아봅시다. 

# Project에 Provisioning Profile 적용하기

![image4.jpg](/images/2025-08-04-iOS-cicd-provisioning-profile/image4.jpg)

Bundle Identifier가 정해졌다는 가정하에 우리가 Project에 넣어줘야 할 정보는 다음과 같습니다.

1. Provisioning Profile
2. Signing Certificate
3. Team

*Team ↔ Signing Certificate 순서를 바꾼 이유는 아래에 기재.

---

먼저 Provisioning Profile과 Signing Certficate를 `update_project_provisioning`을 통해서 주입해봅시다. 

[update_project_provisioning - fastlane docs](https://docs.fastlane.tools/actions/update_project_provisioning/)

### 0. 어떤 Project 파일에 넣을지 정해야 합니다.

update_project_provisioning의 `xcodeproj` 파라미터를 이용해서 프로젝트 파일의 경로를 지정해줍니다.

### 1. Provisioning Profile

우리가 위에서 Git을 통해서 관리하고 `match`를 통해서 다운로드 받았던 Provisioning Profile 경로입니다.

우리가 `./fastlane`에 다운로드 받았으니 정상적으로 다운로드 받았다면

`./fastlane/AppStore_{Bundle Identifier}.mobileprovision`으로 다운로드 받아졌을 겁니다.

이 경로를 `profile` 경로에  넣어 줍니다.

### 2. Signing Certificate

어떤 Certificate로 진행할 것인지에 정하는 겁니다.

우리가 이전에 `match`설정하면서 올린 certificate에 대한 정보를 적어야 합니다.

![image5.jpg](/images/2025-08-04-iOS-cicd-provisioning-profile/image5.jpg)

Apple Developer에 들어가서 이전에 올린 Certificate에 대한 정보를 확인하면 `Certificate Name`을 알 수 있습니다.

`{Certificate Name} {(TEAM_ID)}` 형태로 정해지게 됩니다.

각자의 프로젝트에 맞게 String으로 `code_signing_identity` 파라미터에 지정해 줍니다.

```ruby
    update_project_provisioning(
      xcodeproj: "{.xcdoeproj 파일 경로}",
      profile: "./fastlane/AppStore_{Bundle Identifier}.mobileprovision",
      code_signing_identity: "{Certificate Name} {(TEAM_ID)}"
    )
```

### 3. Team

[update_code_signing_settings - fastlane docs](https://docs.fastlane.tools/actions/update_code_signing_settings/)

Team은 `use_automatic_signing`을 `xcodeproj`파일에 주입해 줍니다.

역시 `xcodeproj`파라미터를 통해서 xcodeproj 파일을 정해줍니다. 

`team_id` 파라미터를 통해서 어떤 team으로 signing 할 것인지 정해줍니다.

team_id는 우리가 이전에 `fastlane match import`를 통해서 지정해준 team_id를 사용하면 됩니다.

https://developer.apple.com/account에 나와 있는 멤버십 세부사항에 있는 팀 ID에서도 확인이 가능합니다.

```ruby
    update_code_signing_settings(
      use_automatic_signing: false,
      path: "{.xcodeproj 파일 경로}",
      team_id: "{team_id}"
    )
```

다만 여기서 use_automatic_signing을 false로 해두었습니다.

왜냐하면 automatic을 켜두면 프로파일이 없으면 자동 생성되는 과정을 거치기 때문입니다.

우리가 Distribution용 Provisioning Profile을 1개만 유지한다는 목적에 부합하지 않기 때문입니다.

이건 각자의 상황에 맞게 `true/false`를 지정하면 될 것 같습니다.

> 확인해보니, defulat가 `false`!!! 하지면 명확하게 하기 위해서 `false`를 유지합니다.
> 

# 마무리

fastlane의 다양한 명령어를 통해서 다음을 수행했습니다.

1. `fastlane match import` 
    
    위 명령어를 통해서 우리는 갖고 있는 provisioning Profile을 Git에 올릴 수 있었습니다.
    
2. Fastfile 설정
    
    2-1. `match`
    
    fastlane match 명령어를 통해서 우리는 provisioning profile을 다운로드 받을 수 있었습니다.
    
    2-2. `update_project_provisioning`
    
    해당 명령어를 통해서 xcodeproj 파일에 Provisioning Profile & Signing Certificate 를 지정할 수 있었습니다.
    
    2-3. `update_code_signing_settings`
    
    해당 명령어를 통해 xocdeproj 파일에 Team을 지정할 수 있었습니다.
    

기존 `fastlane match` 명령어를 쓰면 자동으로 provisiong profile과 certificate를 만들어 줍니다.

하지만 저희 ConnectWave 처럼 다양한 이해관계가 있는 상황에서 Provisioning Profile을 자유롭게 늘리지 못하는 상황에서 많은 도움이 되었으면 하는 바램에서 글을 작성합니다.

<aside>
💡

틀린 정보나 궁금한 점이 있으면 언제든지 댓글로 달아주시면 반영 및 답변을 달겠습니다.

</aside>


### 출처
- [Fastlane match](https://docs.fastlane.tools/actions/match/)
- [Fastlane Update code signing settings](https://docs.fastlane.tools/actions/update_code_signing_settings)
- [Fastlane Update project provisioning](https://docs.fastlane.tools/actions/update_project_provisioning/)

---
layout: post
title:  "크롬 원격 데스크톱 사용하기"
description: "사무실이 아닌 외부에서 업무를 해야할 상황이 발생할 때 크롬 원격 데스크톱을 이용하여 원격지에서 연결할 수 있습니다."
date:   2020.02.18.
writer: "하선호"
categories: Common
---

<!-- ## 준비사항

1. 회사 컴퓨터와 원격 사용 PC 모두 크롬 브라우저가 설치되어 있어야 합니다.
2. 원격 대상 / 원격 사용 PC 모두 동일한 구글 계정으로 로그인을 해야합니다.
2. 절전모드시 네트워크가 해제되어 원격이 종료되므로 원격대상 PC는 절전모드 해제하는 것이 좋습니다. -->

## 사전준비사항

- [ ] 회사 컴퓨터에 크롬 브라우저가 설치되어 있나요?
- [ ] 회사 컴퓨터의 크롬 브라우저에 구글 계정으로 로그인 되어 있나요?
- [ ] 집 컴퓨터에 크롬 브라우저가 설치되어 있나요?
- [ ] 집 컴퓨터의 크롬 브라우저에 동일한 구글 계정으로 로그인 되어 있나요?


## 회사 컴퓨터 설정

> WINDOW10의 경우

### 1. 절전 옵션 들어가기

윈도우 시작버튼에서 "우클릭" 후 전원옵션 메뉴를 선택합니다.
  
![/images/2020-02-18-Common-RemoteDesktop/p1-1.png](/images/2020-02-18-Common-RemoteDesktop/p1-1.png)   


### 2. 절전해제

절전모드에서 시간항목을 '없음' 으로 선택
   
![/images/2020-02-18-Common-RemoteDesktop/p3.png](/images/2020-02-18-Common-RemoteDesktop/p3.png)
   

> WINDOW7의 경우

### 1. 절전 옵션 들어가기


![/images/2020-02-18-Common-RemoteDesktop/win7-1.jpg](/images/2020-02-18-Common-RemoteDesktop/win7-1.jpg)


![/images/2020-02-18-Common-RemoteDesktop/win7-2.jpg](/images/2020-02-18-Common-RemoteDesktop/win7-2.jpg)


![/images/2020-02-18-Common-RemoteDesktop/win7-3.jpg](/images/2020-02-18-Common-RemoteDesktop/win7-3.jpg)


### 2. 절전해제

균형 조정이 선택되어 있는데 우측의 **'설정변경'**을 눌러서 편집화면으로 이동

![/images/2020-02-18-Common-RemoteDesktop/p2-1.png](/images/2020-02-18-Common-RemoteDesktop/p2-1.png)

절전 모드 화면에서 시간항목을 **'해당없음'** 으로 선택

![/images/2020-02-18-Common-RemoteDesktop/p3-1.png](/images/2020-02-18-Common-RemoteDesktop/p3-1.png)



## 회사 컴퓨터 크롬 데스크탑 설정

### 원격 액세스 설정

1. 크롬 데스크탑으로 이동

https://remotedesktop.google.com/access/

2. 크롬에 원격 데스크톱을 설치/추가합니다.

우측의 파란 버튼을 눌러 설치합니다.

![/images/2020-02-18-Common-RemoteDesktop/s1.png](/images/2020-02-18-Common-RemoteDesktop/s1.png)

![/images/2020-02-18-Common-RemoteDesktop/s2.png](/images/2020-02-18-Common-RemoteDesktop/s2.png)

3. 원격 데스크톱 이름 및 PIN 번호를 설정합니다.

이름은 디폴트 값을 사용해도 좋습니다.

PIN 번호는 나중에 원격에서 접속할때 물어보게 되는 비밀번호이므로 잘 기억해둡니다.

![/images/2020-02-18-Common-RemoteDesktop/s1.png](/images/2020-02-18-Common-RemoteDesktop/s3.png)

![/images/2020-02-18-Common-RemoteDesktop/s2.png](/images/2020-02-18-Common-RemoteDesktop/s4.png)
  
4. 원격 데스크톱 준비 완료

![/images/2020-02-18-Common-RemoteDesktop/s5.png](/images/2020-02-18-Common-RemoteDesktop/s5.png) 


## 집 컴퓨터에서 회사 컴퓨터에 연결하기


1. 크롬 데스크탑으로 이동

https://remotedesktop.google.com/access/

회사 컴퓨터와 동일한 구글 계정으로 로그인 해야합니다.

![/images/2020-02-18-Common-RemoteDesktop/list.png](/images/2020-02-18-Common-RemoteDesktop/view1.png) 


2. 접속할 컴퓨터를 클릭하고, 핀번호를 입력하여 접속합니다.
  
![/images/2020-02-18-Common-RemoteDesktop/c1.png](/images/2020-02-18-Common-RemoteDesktop/c1.png) 

3. 완료

![/images/2020-02-18-Common-RemoteDesktop/view22.png](/images/2020-02-18-Common-RemoteDesktop/view22.png) 



> 모니터가 두개라서 화면이 작게 보일 경우 화면 오른쪽 끝에 '반투명 화살표'를 클릭하면 메뉴가 나옵니다.
> 디스플레이 1, 2를 선택하면 화면을 하나씩만 볼수 있습니다.

![/images/2020-02-18-Common-RemoteDesktop/view22.png](/images/2020-02-18-Common-RemoteDesktop/view2.png) 


### 참고 자료

- https://support.google.com/chrome/answer/1649523?co=GENIE.Platform%3DDesktop&hl=ko
- https://dkathrhwlq.tistory.com/259
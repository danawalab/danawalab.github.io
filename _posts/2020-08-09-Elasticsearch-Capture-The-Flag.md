---
layout: post
title:  "Elastic Capture The Flag Korea - 보안경연대회"
description: "엘라스틱에서 주최한 보안경연대회 참가 후기입니다."
date:   2021.08.09.
writer: "김윤기"
categories: Elastic
---

## 소개 : Capture The Flag
![/images/2021-08-09-Elasticsearch-Capture-The-Flag/1.png](/images/2021-08-09-Elasticsearch-Capture-The-Flag/1.png)

지난 7월 16일, 엘라스틱에서 주최한 보안경연대회 Capture The Flag에 참가한 경험을 소개합니다.


## 참가 방법
[보안경연대회 참가 신청 링크](https://events.elastic.co/capturetheflag-korea/)
(현재는 이벤트가 종료되어서 참여할 수 없습니다.)

이름과 이메일주소와 같은 기본 정보만 등록하여 다음과 같은 안내 메일을 받았습니다.
![/images/2021-08-09-Elasticsearch-Capture-The-Flag/6.PNG](/images/2021-08-09-Elasticsearch-Capture-The-Flag/6.PNG)

행사 당일 제공된 링크로 접속하면 이벤트에 참여할 수 있었습니다.


## 보안경연대회
Capture the flag는 에러 로그 추적, 사냥, 이상징후 찾아내기 등 보안미션을 풀어내는 이벤트입니다. 
Elastic security 기술을 소개하기 위해 보안경연대회라는 흥미로운 이벤트를 활용하였습니다.

경연 시작 전, Elastic security 어플리케이션 소개 및 데모 영상을 보여주고 참가자들에게 data metric이 저장되어 있는 elastic kibana 링크가 주어졌습니다.

![/images/2021-08-09-Elasticsearch-Capture-The-Flag/2.png](/images/2021-08-09-Elasticsearch-Capture-The-Flag/2.png)
(키바나 데모)

경연은 27개의 보안 챌린지에 대한 답변을 적어내는 방식으로 진행되었습니다. 
정답을 맞추게 되면 점수를 획득하고 다음 문제로 넘어가는 방식입니다.
안타깝게도 모르는 문제를 패스 할 수는 없었지만 문제마다 힌트를 사용할 수 있었고, 힌트를 사용해서 문제를 맞추면 원래 점수의 반을 획득할 수 있었습니다. 
어렵다고 힌트를 너무 많이 사용하면 많이 풀어도 고득점을 할 수 없었습니다.
제한시간에 가장 많은 문제를 풀어 득점을 하는 것이 경연목표였습니다.

챌린지는 Kibana의 security 어플리케이션을 활용하여 맞출 수 있는 문제였습니다. 
문제 방식은 특정 시간에 어떤 이벤트가 발생하였고, 이벤트 내용에 대해 분석하거나 트래킹하여 원하는 정보를 알아내는 것이었습니다.

### 문제 예시
> 1. 8월동안 가장 많은 네트워크 이벤트를 생성한 것은?<br>A) Filebeat <br><br>
> 2. 8월 9일에 등록 된 도메인 중 'www.danawa.com' 의 질의 수는?<br>A) 100 <br><br>
> 3. 'admin' 계정을 사용하여 '127.0.0.1' 네트워크에 연결한 프로세스는 무엇입니까?<br>A) powershell.exe


키바나에서 시간만 설정하면 Dashboard나 Overview에서 보고 쉽게 답할 수 있는 문제도 있었고, security 어플리케이션의 정확한 탭(Detections, Network, Hosts 등)에서 알맞는 filter 쿼리를 설정해야 알아낼 수 있는 문제도 있었습니다.

다나와에서는 ElasticSearch로 검색운영을 하고 있고 Kibana의 사용에 익숙하여 문제의 해답을 내는 데 많은 도움이 되었습니다. 
또한 엔드포인트, 네트워크 기본 이해, SOC 또는 사고 대응 분석가와 같은 IT 또는 보안 운영 역할에서 근무한 경험이나 SIEM을 사용한 경험과 위협 헌팅(Threat hunting)에 지식이 있다면 미션수행에 도움이 될 수 있습니다. 
하지만 특별한 경험이나 기술이 없어도 Elastic에서 메일로 제공하는 사전준비영상만으로 경연 준비는 충분했습니다. 

### 사전 준비 영상
* [Capture The Flag 사전 준비 영상 - Elastic Security를 활용한 Investigation](https://videos.elastic.co/watch/xMDDnxKq3tuwHpysxQNy67)
* [Kibana Lens를 사용하여 쉽게 Visualization 추가하기](https://drive.google.com/file/d/15qtDCNRuwPrAZnECjf8DLeSKp15pBpi9/view)
* [Elastic Security 최신 기술 소개](https://www.elastic.co/kr/webinars/elastic-security-7-13-updates)

![/images/2021-08-09-Elasticsearch-Capture-The-Flag/3.png](/images/2021-08-09-Elasticsearch-Capture-The-Flag/3.png)

경연이 끝나고나서 가장 고득점자 3명을 선발하여 상품 증정이 있었습니다.

![/images/2021-08-09-Elasticsearch-Capture-The-Flag/4.PNG](/images/2021-08-09-Elasticsearch-Capture-The-Flag/4.PNG)

며칠 후 모든 참석자에게 Capture The Flag 수료 뱃지가 제공되었습니다.
elastic에서 여러모로 이번 이벤트를 위해 많은 준비를 했던 것이 돋보였습니다.

## 후기
보안 분야는 전문적이고 어렵게 다가와서 참가하기도 전부터 걱정이 되었습니다. 
하지만 경연을 시작하고 나서는 그런 걱정은 생각도 나지 않을 정도로 문제를 푸는 데 집중하게 되었습니다.
점수 분포가 어떻게 되는지 알 수는 없지만, 경연 시간동안 최선을 다하여 문제를 풀었고 재미있는 경험이었습니다.

다나와에서는 ElasticSearch로 검색운영을 하고 있기 때문에 에러로그 분석 및 이상징후 탐지와 같이 필요한 분야를 연습하는 좋은 기회였습니다. 


## Elastic Webinar
보안경연대회는 웨비나의 형태로 진행되었습니다.
웨비나(Webinar)란 Web + Seminar의 합성어로 온라인 상에서 하는 세미나를 말합니다.

![/images/2021-08-09-Elasticsearch-Capture-The-Flag/5.PNG](/images/2021-08-09-Elasticsearch-Capture-The-Flag/5.PNG)

[Elastic Videos & Webinars](https://www.elastic.co/kr/videos/?language=Korean&usecase=All&region=All&industry=All)

현재 Elastic에서는 보안경연대회 뿐 아니라 Elastic의 다양한 기술에 대해서 웨비나를 진행하고 있습니다.
웨비나 당일이 아니더라도 다시 볼 수 있는 영상과 내용이 있어서 부담없이 볼 수 있습니다. 

Elastic에 관심이 있거나 사용하고 계시다면 웨비나를 참여하여 관련된 기술 정보를 습득하는 것도 좋은 방법이 될 것 같습니다.

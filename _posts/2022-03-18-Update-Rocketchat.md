---
layout: post
title:  "로켓챗 설치 및 버전 업그레이드 방법"
description: "로켓챗 설치 및 업그레이드 방법에 대해 설명합니다."
date:   2022.03.18.
writer: "선지호"
categories: Common
---

## 로켓챗 이란 ?

Rocket.Chat 은 zulip, slack과 비슷한 오픈소스 메신저 서버/클라이언트 프로그램입니다. 

실시간 채팅을 할 수 있고, 음성과 화상채팅도 가능합니다. 

또한 다양한 서버 OS를 지원하고 있습니다.

## 다나와에서는..

다나와 Devops/검색 팀에서는 메신저로 로켓챗 서버를 구축해서 사용중입니다.
진행했던 이슈들에 대한 대화내역과 이미지, URL 등 저장을 위해 따로 서버를 구축하여 사용하고 있습니다.

## 로켓챗 설치 

stand alone 으로 설치하는 것이 아닌 docker를 이용하여 설치를 진행하겠습니다.
아래 내용은 최신 버전으로 사용하는 법으로 기술하겠습니다.

#### 1. curl -k -L https://go.rocket.chat/i/docker-compose.yml -O

#### 2. docker-compose 파일 안에서 mongodb의 버전을 변경합니다
- mongo:4.0 -> mongo:4.4

#### 3. docker-compose 파일 안에서 mongodb의 command를 변경합니다
- 기존: "command: mongod --smallfiles --oplogSize 128 --replSet rs0 --storageEngine=mmapv1"
- 변경: "command: mongod --oplogSize 128 --replSet rs0 --storageEngine=wiredTiger"

(주의, mongo-init-replica 가 아니라 mongo 서비스 여야만 합니다.)

#### 4. docker-compose를 실행합니다

#### 5. 아래 이미지 처럼 나오면 성공입니다.

![/images/2022-03-18-Update-Rocketchat/image1.png](/images/2022-03-18-Update-Rocketchat/image1.png)


## 로켓챗 기존 데이터 마이그레이션

### 마이그레이션을 해야 하는 이유

기존에 로켓챗이 설치 되어 있을 경우에는 마이그레이션이 필요할 수도 있습니다.
그 이유는 기존 로켓챗은 mongodb 버전이 4.0 버전 이하를 사용할 수도 있고, 또한 4.0 버전 이하를 사용하게 된다면 storageEngine이 mmapv1으로 되어 있을 수도 있기 때문입니다.
또한, mongodb 에서는 mongodb 4.2 버전부터는 storageEngine이 wiredTiger로 변경 되었기 때문에 이를 변경해 주어야 합니다.
만약, 기존 로켓챗에 storageEngine이 wiredTiger로 되었다면 아래 8번부터 12번까지만 수행하시면 됩니다.

### 마이그레이션 

#### 1. 기존 데이터를 백업
기존에 있던 데이터들이 삭제가 되면 안되기 때문에 백업을 진행합니다. 백업을 따로 gz 형태로 압축을 해둔다면 더 좋습니다.

#### 2. git clone https://github.com/RocketChat/docker-mmap-to-wiredtiger-migration ~/rocketchat-migration

#### 3. cd  ~/rocketchat-migration

#### 4. cp -R docker (rocketchat 경로)

#### 5. 기존의 도커 컴포즈 파일을 수정

```
version: '2'
 
services:
  rocketchat:
    image: registry.rocket.chat/rocketchat/rocket.chat:4.5.2  //최신 버전이 아니라면 최신버전으로 넣어주세요
    command: >
      bash -c
        "for i in `seq 1 30`; do
          node main.js &&
          s=$$? && break || s=$$?;
          echo \"Tried $$i times. Waiting 5 secs...\";
          sleep 5;
        done; (exit $$s)"
    restart: always
    volumes:
      - ./uploads:/app/uploads
    environment:
      - PORT=3000
      - ROOT_URL=http://localhost:3000
      - MONGO_URL=mongodb://mongo:27017/rocketchat
      - MONGO_OPLOG_URL=mongodb://mongo:27017/local
    depends_on:
      - mongo
    ports:
      - 3000:3000
    labels:
      - "traefik.backend=rocketchat"
      - "traefik.frontend.rule=Host: your.domain.tld"
 
  migrator: // 이 부분을 추가해 줍니다.
    build: ./docker/
    volumes:
      - ./data/db:/data/db

  mongo: // 이미지버전은 같게 하되, command만 변경합니다.
    image: mongo:4.0 
    restart: unless-stopped
    volumes:
      - ./data/db:/data/db
      - ./data/dump:/dump
    command: > 
      bash -c
        "while [ ! -f /data/db/WiredTiger ]; do
          echo \"wiredTiger migration hasn't started yet. Waiting 30 secs...\";
          sleep 30;
        done;
      docker-entrypoint.sh mongod --oplogSize 128 --replSet rs0 --storageEngine=wiredTiger;"
    labels:
      - "traefik.enable=false"
 
  // 수정할 필요는 없습니다.
  mongo-init-replica:
    image: mongo:4.0
    command: >
      bash -c
        "for i in `seq 1 30`; do
          mongo mongo/rocketchat --eval \"
            rs.initiate({
              _id: 'rs0',
              members: [ { _id: 0, host: 'localhost:27017' } ]})\" &&
          s=$$? && break || s=$$?;
          echo \"Tried $$i times. Waiting 5 secs...\";
          sleep 5;
        done; (exit $$s)"
    depends_on:
      - mongo
```
#### 6. 로켓챗 재시작

#### 7. docker logs -f rocketchat_mongo_1

mongodb가 정상적으로 잘 실행 되었는지 확인합니다.

정상적으로 실행이 되지 않았으면 원인을 찾아 수정합니다

#### 8. docker exec -it rocketchat_mongo_1 mongo

setFeatureCompatibilityVersion 같은 경우에는 올릴 버전보다 한 단계 전 버전을 입력해주시면 됩니다.

4.0 -> 4.4 같이 한번에 두 단계를 올리는것은 권장하지 않습니다.

다음 단계로 버전 업그레이드를 진행하여 주세요.
(현재 release 버전: 4.0, 4.2, 4.4, 5.0)

```
$ db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
$ db.adminCommand( { setFeatureCompatibilityVersion: "4.0" } ) 
```

#### 9. 정상적으로 실행이 다 된 것을 확인 한 이후에 docker-compose down을 하여 컨테이너를 내립니다

#### 10. docker-compose 파일을 수정합니다.

```
version: '2'
 
services:
  rocketchat:
    image: registry.rocket.chat/rocketchat/rocket.chat:4.5.2  //최신 버전이 아니라면 최신버전으로 넣어주세요
    command: >
      bash -c
        "for i in `seq 1 30`; do
          node main.js &&
          s=$$? && break || s=$$?;
          echo \"Tried $$i times. Waiting 5 secs...\";
          sleep 5;
        done; (exit $$s)"
    restart: always
    volumes:
      - ./uploads:/app/uploads
    environment:
      - PORT=3000
      - ROOT_URL=http://localhost:3000
      - MONGO_URL=mongodb://mongo:27017/rocketchat
      - MONGO_OPLOG_URL=mongodb://mongo:27017/local
    depends_on:
      - mongo
    ports:
      - 3000:3000
    labels:
      - "traefik.backend=rocketchat"
      - "traefik.frontend.rule=Host: your.domain.tld"

  mongo: // 이미지 버전을 변경하고, command를 수정합니다.
    image: mongo:4.2
    restart: unless-stopped
    volumes:
      - ./data/db:/data/db
      - ./data/dump:/dump
    command: mongod --oplogSize 128 --replSet rs0 --storageEngine=wiredTiger"
    labels:
      - "traefik.enable=false"
 
  // 이미지 버전을 수정합니다
  mongo-init-replica:
    image: mongo:4.2
    command: >
      bash -c
        "for i in `seq 1 30`; do
          mongo mongo/rocketchat --eval \"
            rs.initiate({
              _id: 'rs0',
              members: [ { _id: 0, host: 'localhost:27017' } ]})\" &&
          s=$$? && break || s=$$?;
          echo \"Tried $$i times. Waiting 5 secs...\";
          sleep 5;
        done; (exit $$s)"
    depends_on:
      - mongo
```

#### 11. docker-compose up -d 를 입력하여 재시작합니다

#### 12. 정상적으로 동작 및 데이터가 손실 되었는지 확인합니다.

#### 13. 관리자 계정으로 접속하여 관리탭으로 이동, 정상적으로 업데이트가 진행되었는지 확인하여 주세요

![/images/2022-03-18-Update-Rocketchat/image2.png](/images/2022-03-18-Update-Rocketchat/image2.png)

## 정리

팀 내 메신저로 로켓챗을 이용하며 있던 이슈에 대해 해결하며 있던 내용들을 일부 정리해보았습니다.
사내 메신저로 로켓챗을 고려 중인 팀 혹은 모임에서 카톡이나 라인과는 다른 메신저를 사용해보려는 분들 등 이 글을 읽고 도움이 되었으면 좋겠습니다.

## 참고 자료
- https://docs.rocket.chat/quick-start/installing-and-updating/rapid-deployment-methods/docker-and-docker-compose
- https://docs.mongodb.com/v4.4/tutorial/change-standalone-wiredtiger/
- https://linuxhandbook.com/mongodb-upgrade-rocket-chat/
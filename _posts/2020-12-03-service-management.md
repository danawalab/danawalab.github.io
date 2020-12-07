---
layout: post
title:  "서비스 운영플랫폼 - 설치"
description: "서비스 운영플랫폼 설치는 두가지 방식으로 실행가능합니다. 하나는 도커 컨테이너에 실행하거나 호스트에 직접 실행하는 방법입니다. 두 가지 실행방법을 정리해보았습니다. " 
date:   2020.12.03. 16:00:00
writer: "김준우"  
categories: Management
---
## 소개

서비스 운영플랫폼 설치는 두가지 방식으로 실행가능합니다. 하나는 도커 컨테이너에 실행하거나 호스트에 직접 실행하는 방법입니다. 두 가지 실행방법을 정리해보았습니다. 

## 도커 컴포즈

### 1. 깃헙에서 소스를 받습니다.

```jsx
$ git clone https://github.com/danawalab/service-management.git && cd service-management
```

### 2. 도커 빌드를하여 이미지를 생성합니다.

```jsx
docker build -t service-management .
```

### 3. 도커 컴포즈 파일을 생성합니다.

```jsx
version: "3.7"
services:
  svg-mng:
    image: dcr.danawa.io/service-management
    ports:
    - 3000:3000
    environment:
    - TZ=Asia/Seoul
    - smtp_user=<gmail userid>
    - smtp_password=<gmail password>
    - session_timeout=60
    - webssh_host=http://webssh:8080
    - docker_compose_home_path=/data
    - docker_compose_file_name=docker-compose.yml
  webssh:
    image: dcr.danawa.io/alpine-webssh
    ports:
    - 8080:8080
    command:
    - python
    - run.py
    - --address=0.0.0.0
    - --port=8080
    - --xsrf=False
    - --origin="*"
    - --maxconn=4000
    - --debug=True
    - --xheaders=False
    - --redirect=False
```

### 4. 도커 컴포즈를 실행합니다.

```jsx
docker-compose up -d
```

### 5. 실행 확인합니다.

```jsx
$ docker-compose ps

Name                       Command               State           Ports
----------------------------------------------------------------------------------------
svc-mng-demo_svg-mng_1   docker-entrypoint.sh npm start   Up      0.0.0.0:3000->3000/tcp
svc-mng-demo_webssh_1    python run.py --address=0. ...   Up      0.0.0.0:8080->8080/tcp
```

### 6. [localhost:3000](http://localhost:3000) 웹 접속을 확인합니다.

## 호스트 설치

### 1. 소스를 다운받습니다.

```jsx
$ git clone https://github.com/danawalab/service-management.git && cd service-management
```

### 2. 노드 빌드를 진행합니다.

```jsx
$ npm run build
```

### 3. next js를 실행합니다.

```jsx
$ npm start
```

### 4. [localhost:3000](http://localhost:3000) 접속하시면 서비스 운영플랫폼을 사용할 수 있습니다.


## 정리

서비스 운영플랫폼은 실행방법이 간단하며, React, NextJS를 사용하여 프로세스 하나로 모든걸 구동하게 됩니다. webssh는 서드파티 툴을 사용합니다. 현재 데모에서는 python의 wssh을 사용하였습니다.
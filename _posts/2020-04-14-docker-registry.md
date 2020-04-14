---
layout: post
title:  "# k8s 도커 레지스트리 설치"
description: "이번에 포스팅내용은 도커 레지스트리 구축방법입니다. 이미 도커 허브에 계정이 있다면 저장소가 자동 생성되었지만 이번에는 프라이빗하게 사용할 도커 저장소를 설치해보겠습니다. 레지스트리는 tls적용이 되어야 이미지를 push, pull 이 가능합니다. 이번에 설치할때는 Let's Encrypt의 인증서를 발급받아 레지스트리를 구축하는 방법을 알아보도록 하겠습니다."
date:   2020.04.14.
writer: "김준우"
categories: Kubernetes
---
## 소개

안녕하세요. 다나와 김준우입니다. 이번에 포스팅내용은 도커 레지스트리 구축방법입니다. 이미 도커 허브에 계정이 있다면 저장소가 자동 생성되었지만 이번에는 프라이빗하게 사용할 도커 저장소를 설치해보겠습니다. 레지스트리는 tls적용이 되어야 이미지를 push, pull 이 가능합니다. 이번에 설치할때는 Let's Encrypt의 인증서를 발급받아 레지스트리를 구축하는 방법을 알아보도록 하겠습니다.

## 구축방법

인증서를 우선 발급받고, 레지스트리에 인증서를 적용하여 설치해보겠습니다.

### 인증서 발급

EPEL repo 등록합니다.

    $ sudo yum -y install yum-utils
    $ sudo yum-config-manager --enable rhui-REGION-rhel-server-extras rhui-REGION-rhel-server-optional

certbot 설치합니다.

    $ sudo yum -y install certbot

certbot-auto 명령을 통해 인증서를 발급받습니다.

    $ sudo certbot-auto certonly \
    --manual \
    --preferred-challenges=dns \
    --email dnwlab@gmail.com \
    --server https://acme-v02.api.letsencrypt.org/directory \
    --agree-tos \
    --debug \
    --no-bootstrap \
    -d *.danawa.io

인증서 발급이 끝나면 /etc/letsencrypt/live/ 경로에 도메인 디렉토리에 인증서 파일이 생성이 됩니다. 여기서 사용할 파일은 fullchain.pem과 privkey.pem 입니다. 

![/images/2020-04-14-docker-registry/Untitled.png](/images/2020-04-14-docker-registry/Untitled.png)

### 레지스트리 설치

도커 레지스트리 설치는 k8s deployment.yaml을 작성하여 배포 하도록 하겠습니다.

우선 인증서 파일을 k8s secret에 등록합니다.

    $ kubectl create secret tls danawa.io \
       --key privkey.pem \
       --cert fullchain.pem

이미지는 registry:2를 사용하하였고,  secret에 등록한 인증서를 마운트 방식을 사용하여 컨테이너의 /etc/certs에 할당하였습니다. 환경변수에 registry 인증서 경로만 등록하면  완료입니다.

    containers:
          - name: registry
            image: registry:2
            resources:
              limits:
                memory: "128Mi"
                cpu: "500m"
            ports:
            - containerPort: 5000
            env:
            - name: REGISTRY_STORAGE_DELETE_ENABLED
              value: "true"
            - name: REGISTRY_HTTP_TLS_CERTIFICATE
              value: /etc/certs/tls.crt
            - name: REGISTRY_HTTP_TLS_KEY
              value: /etc/certs/tls.key
            volumeMounts:
            - name: tls
              mountPath: /etc/certs
              readOnly: true
       ... (중략)
          volumes:
          - name: tls
            secret:
              secretName: danawa.io

추가로 레지스트리 UI를 컨테이너 포함하여 이미지 정보를 웹으로 확인할 수 있도록 하겠습니다. 같은 POD로 띄우게 되면 네트워크가 동일하기 때문에 로컬의 5000포트로 접근하면 됩니다.

    - name: registry-ui
            image: joxit/docker-registry-ui:static
            resources:
              limits:
                memory: "128Mi"
                cpu: "500m"
            ports:
            - containerPort: 80
            env:
            - name: REGISTRY_URL
              value: http://127.0.0.1:5000
            - name: DELETE_IMAGES
              value: "true"

외부 접근 가능하도록 서비스를 등록하겠습니다. 5000은 레지스트리 포트이며, 80포트는 UI포트입니다. type을 NodePort로 설정하였기 때문에 3만번대 포트로 할당받을 겁니다.

    apiVersion: v1
    kind: Service
    metadata:
      name: registry
    spec:
      selector:
        app: registry
      type: NodePort
      ports:
      - name: http
        port: 5000
        targetPort: 5000
      - name: ui-http
        port: 80
        targetPort: 80

완성된 registry.yaml 파일의 내용입니다.

    kind: Deployment
    metadata:
      name: registry
    spec:
      selector:
        matchLabels:
          app: registry
      template:
        metadata:
          labels:
            app: registry
        spec:
          containers:
          - name: registry
            image: registry:2
            resources:
              limits:
                memory: "128Mi"
                cpu: "500m"
            ports:
            - containerPort: 5000
            env:
            - name: REGISTRY_STORAGE_DELETE_ENABLED
              value: "true"
            - name: REGISTRY_HTTP_TLS_CERTIFICATE
              value: /etc/certs/tls.crt
            - name: REGISTRY_HTTP_TLS_KEY
              value: /etc/certs/tls.key
            volumeMounts:
            - name: tls
              mountPath: /etc/certs
              readOnly: true
          - name: registry-ui
            image: joxit/docker-registry-ui:static
            resources:
              limits:
                memory: "128Mi"
                cpu: "500m"
            ports:
            - containerPort: 80
            env:
            - name: REGISTRY_URL
              value: http://127.0.0.1:5000
            - name: DELETE_IMAGES
              value: "true"
          volumes:
          - name: tls
            secret:
              secretName: danawa.io
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: registry
    spec:
      selector:
        app: registry
      type: NodePort
      ports:
      - name: http
        port: 5000
        targetPort: 5000
      - name: ui-http
        port: 80
        targetPort: 80

정상적으로 이미지가 push, pull 되는지 확인합니다. 아래 명령어는 도커 허브에서 nginx을 받아 이번에 설치한 레지스트리에 푸시하는 명령어 입니다. 

    $ docker pull nginx
    $ docker tag nginx danawa.io/nginx
    $ docker push danawa.io/nginx

http://<레지스트리 아이피>:<80으로 할당받은 노드 포트> 접근시 아래 이미지 처럼 ui도 같이 확인이 가능합니다.

![/images/2020-04-14-docker-registry/Untitled%201.png](/images/2020-04-14-docker-registry/Untitled%201.png)

## 정리

이번에 간단하게 lets encrypt 사용하여 무료로 인증서 생성하여 도커 레지스트리를 설치해보았습니다. 설치하는 방법은 간단하지만 인증서는 3개월만 유지되고 1개월 남은 시점부터 갱신을 해야하는 번거로움이 있습니다. 하지만 certbot에서 인증서 갱신도 지원하기 때문에 영구적으로 사용이 가능할걸로 보입니다. 현재 레지스트리와 뮤지엄을 통합 관리할 수 있는 CNCF에서 인큐베이팅중인 harbor는 프로젝트 단위로 도커 이미지와 helm 차트 저장소를 같이 관리할 수 잇는 프로젝트입니다. 기회가 되면 구축하여 프로젝트시 사용해보면 좋을거 같습니다. gitlab에서도 regisry를 등록하여 CI/CD에서 레지스트리를 사용할 수 있습니다.

## 참고링크

[certbot 서치]([https://certbot.eff.org/lets-encrypt/centosrhel7-apache.html](https://certbot.eff.org/lets-encrypt/centosrhel7-apache.html))

[habor demo]([https://demo.goharbor.io/](https://demo.goharbor.io/)) 

harbor 샘플계정: sample / Test1234

[GitLab Registry]([https://docs.gitlab.com/ee/user/packages/container_registry/](https://docs.gitlab.com/ee/user/packages/container_registry/))
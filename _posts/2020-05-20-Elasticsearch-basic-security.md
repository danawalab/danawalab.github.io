---
layout: post
title:  "[ElasticSearch] 기본 보안"
description: "안녕하세요. 다나와의 김준우입니다.  엘라스틱서치의 클러스터 통신에 암호화를 적용하고, 키바나 로그인을 활성화 해보는 내용을 포스팅하였습니다.  엘라스틱서치 클러스터를 구성하면 기본적으로 보안은 비활성화 상태가됩니다. 비활성화 상태로 사용하게 되면 외부에서 PUT/POST API 또는 키바나를 통해 위협이 될 수 있기 때문에 필수로 보안을 활성화하여 운영해야 됩니다."
date:   2020.05.20.
writer: "김준우"
categories: Elastic
---
## 소개

안녕하세요. 다나와의 김준우입니다.  엘라스틱서치의 클러스터 통신에 암호화를 적용하고, 키바나 로그인을 활성화 해보는 내용을 포스팅하였습니다.  엘라스틱서치 클러스터를 구성하면 기본적으로 보안은 비활성화 상태가됩니다. 비활성화 상태로 사용하게 되면 외부에서 PUT/POST API 또는 키바나를 통해 위협이 될 수 있기 때문에 필수로 보안을 활성화하여 운영해야 됩니다.

### 구성도

엘라스틱서치 클러스터는 아래 이미지처럼 파란박스로 표시한 부분이 암호화해야될 영역입니다. 이미지에서 노드가 겹쳐있는 부분도 동일하게 암호화됩니다.

![/images/2020-05-20-Elasticsearch-basic-security/Untitled.png](/images/2020-05-20-Elasticsearch-basic-security/Untitled.png)

### 테스트 버전

엘라스틱, 키바나 버전입니다.

- OS: CentOS 7
- ElasticSearch: 7.6
- Kibana: 7.6

## 엘라스틱 보안 설정방법

마스터 노드에서 p12 인증서를 생성합니다. 마스터 노드가 여러개 있을 경우도 인증서는 1개만 생성합니다. 비밀번호는 편의상 없이 진행합니다.

```
$ elasticsearch-certutil cert -out config/elastic-certificates.p12 -pass ""
```

생성된 인증서는 엘라스틱서치 config 디렉토리에 p12 인증서 파일이 생성 되었습니다. 생성된 인증서는 전체 노드에 동일하게 config 디렉토리에 복사합니다.

![/images/2020-05-20-Elasticsearch-basic-security/Untitled%201.png](/images/2020-05-20-Elasticsearch-basic-security/Untitled%201.png)

전체 노드에 elasticsearch.yml 파일 아래 내용 추가합니다.  시큐리티를 활성화하고 transport시 ssl을 적용하는 설정입니다. 추가 후 엘라스틱을 재시작합니다.

```
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```

엘라스틱 스택의 서비스를 사용할때 기본 계정의 비밀번호를 설정해야합니다. 방식은 두가지가 있습니다. 하나는 랜덤값을 비밀번호로 설정하는 방법이 있고, 다른 하나는 대화형으로 비밀번호를 입력하는 방법이 있습니다. 비밀번호는 변경 가능하여 자동으로 생성합니다. 

```
elasticsearch-setup-passwords auto         // 자동
or 
elasticsearch-setup-passwords interactive  // 대화형
```

아래와 같이 비밀번호가 표시됩니다.

![/images/2020-05-20-Elasticsearch-basic-security/Untitled%202.png](/images/2020-05-20-Elasticsearch-basic-security/Untitled%202.png)

### HTTP SSL 적용 (선택사항)

일단 공인인증서 또는 Let'sEncrypt를 통해 인증서를 가지고 있다면 http 요청을 https 로 변경해보도록 하겠습니다. 인증서파일 생성방법은 생력하도록 하겠습니다.

엘라스틱서치 elasticsearch.yml 에 아래 내용을 추가합니다. 인증서는 config 디렉토리 하위에 넣어두면 됩니다.

```
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.key: danawa.io.key
xpack.security.http.ssl.certificate_authorities: ca.crt
xpack.security.http.ssl.certificate: danawa.io.crt
```

테스트를 해봅니다. 정상적으로 Response가 나오면 정상입니다.

```
curl https://es.danawa.io:9200 
```

### 중간 정리

이것으로 엘라스틱 서치의 보안은 활성화 되었습니다. 보안적용 방식이 크게 두가지로 나뉘는데 노드간 보안과 REST API 프로토콜 보안으로 나뉘게 됩니다. 보안이 할성화되면 http/https로 요청을 보낼때 기본 사용자 인증이 필요하게 됩니다.

## 키바나 보안 설정방법

엘라스틱서치에서 생성한 계정에서 kibana 계정의 아이디와 비밀번호를 kibana.yml에 추가 후 키바나를 재시작합니다. 엘라스틱서치가의 http에 ssl 적용되어 있을경우 verificationMode 추가하기 바랍니다. 

```
elasticsearch.ssl.verificationMode: none
elasticsearch.hosts: "http://es.danawa.io:9200"
elasticsearch.username: "kibana"
elasticsearch.password: "7nfFbPGqweFDgHlQqsV4"
```

키바나로 접속해보면 기존엔 바로 대시보드가 보였지만, 이제는 로그인 페이가 활성화 된걸 확인할 수 있습니다. 사용자계정이 현재 없기 때문에 이번에 생성한 elastic 계정으로 로그인을 합니다.

![/images/2020-05-20-Elasticsearch-basic-security/Untitled%203.png](/images/2020-05-20-Elasticsearch-basic-security/Untitled%203.png)

### 사용자 계정 생성

이번에는 사용자 계정을 생성해보도록 하겠습니다. Management / Security 메뉴가 생성된걸 확인 할 수 있습니다. 아래 이미지를 보면 명령어로 만들었던 계정들이 보여지는걸 확인 할 수 있습니다. 여기서 Create User 버튼을 눌러 사용자를 생성합니다.

![/images/2020-05-20-Elasticsearch-basic-security/Untitled%204.png](/images/2020-05-20-Elasticsearch-basic-security/Untitled%204.png)

jwk 계정을 생성해보았습니다. 로그인 까지 정상적으로 진행이 되는걸 확인 할 수 있습니다.

![/images/2020-05-20-Elasticsearch-basic-security/Untitled%205.png](/images/2020-05-20-Elasticsearch-basic-security/Untitled%205.png)

![/images/2020-05-20-Elasticsearch-basic-security/Untitled%206.png](/images/2020-05-20-Elasticsearch-basic-security/Untitled%206.png)

### 키바나 HTTP SSL 적용[선택사항]

키바나에 ssl 적용이 필요하면 아래 내용을 kibana.yml에 추가합니다.

```
server.ssl.enabled: true
server.ssl.certificate: danawa.io.crt
server.ssl.key: danawa.io.key
```

## 정리

엘라스틱서치의 보안을 활성화해보고 키바나에 사용자를 생성하여 로그인을 해보았습니다.  엘라스틱에서는 여러가지 보안 설정이 있습니다. 이번 포스팅은 기본 보안에서도 노드 통신 ssl 적용, 키바나 로그인 활성화를 적용해보았습니다.  엘라스틱서치에서는 ssl 적용이 http와 tcp를 다르게 적용할 수 있는 장점이 있었던거 같습니다 . 노드 간의 통신시 사설 인증서를 사용하여 인증서를 가지지 않은 사용자는 전부 차단이 될 수 있는 장점이 있었고, 사용자가 사용하는 API 에서는 공인 인증서를 등록하여 좀더 보안에 안전하게 데이터를 전송할 수 있었습니다.
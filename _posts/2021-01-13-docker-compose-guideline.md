---
layout: post
title:  "도커 컴포즈 가이드라인"
description: "도커 컴포즈를 사용할 때 가이드라인이 필요하다고 생각하여 작성하여 공유 합니다." 
date:   2021.01.13. 16:00:00
writer: "선지호"  
categories: Docker
---
## 소개
>도커 컴포즈를 사용하여 여러개의 컨테이너를 같이 띄울 때, 쉽게 사용할 수 있도록 가이드라인을 작성해 보았습니다.
>간단한 예제를 통해 도커 컴포즈를 하실 수 있도록 진행해 보았으니, 부담 가지지 마시고 천천히 진행해 주시면 좋겠습니다.

## 도커 컴포즈 란?
>도커 컴포즈는 다중 컨테이너 도커 애플리케이션을 정의하고 실행하기위한 도구입니다.
>도커 컴포즈에서는 YAML 파일을 사용하여 애플리케이션의 서비스를 구성합니다. 
>그런 다음 단일 명령으로 YAML 파일에 정의된 모든 서비스를 만들고 시작합니다. 

## 예제 다운로드 및 실행
예제 스프링부트 프로젝트는 아래 링크에서 받으실 수 있습니다.
- [Docker-compose-demo.zip](https://github.com/zozond/dockerfile-demo/raw/master/Docker-compose-demo.zip)

해당 docker-compose.yml 파일 위치에서 docker-compose up 을 하시면 정상적으로 동작하는 것을 확인 할 수 있습니다.

## 예제 1) 일반적인 서버 - DB 구성 
아래는 docker-compose.yaml 파일의 내용입니다.
도커 컴포즈 YAML 파일에서는 탭이 인식되지 않기 때문에 하위 항목을 2칸씩 띄워서 구분할 수 있습니다.

```jsx
# 도커 컴포즈의 스키마 버전입니다. 이 스키마 버전은 docker의 버전에 따라 지원되는 버전이 달라집니다.
# 되도록 최근의 버전을 사용하는 것이 좋습니다.
version: "3"
 
# 애플리케이션의 일부로 실행하려는 서비스 목록을 정의 합니다.
# 서비스 이름은 임의로 선택 할 수 있습니다.
# 저는 이전글의 도커 파일로 생성했던 이미지를 한번 사용해 보았습니다.
services:
  app1:
    #build를 사용하게 된다면 image 항목을 사용하지 않아도 도커 컴포즈가 실행됩니다.
    #build :
      # 빌드 명령을 실행할 디렉터리 경로
      #context: .
      # 도커 이미지를 빌드하는데 사용할 도커 파일 위치
      #dockerfile: ./Dockerfile

    # 이미지 셋팅입니다
    image: demo:latest

    # 커맨드의 변경이 필요하다면 여기서 재 정의를 할 수 있습니다
    #command: /bin/bash -c "java -jar demo-0.0.1-SHAPSHOT.jar"

    # 노출시킬 포트 입니다.
    # - 로 여러개의 포트를 노출 시킬 수 있습니다
    ports:
    - 8080:8080
    # 작업 디렉토리를 지정해 줄 수 있습니다.
    # working_dir: /app

    # 마운트 할 볼륨입니다. 이 부분은 docker로 생성할 때 지정했던 부분과 거의 일치 합니다.
    # 상대경로로 입력도 가능합니다. ex) ./:/app
    volumes:
     - C:/Users/admin/Desktop/volume/log:/volume
    depends_on:
    # 의존 관계 설정
    - database
   
    # 데이터 베이스가 필요하거나 다른 컨테이너와의 통신이 필요하다면, 이 항목을 통해 연결할 수 있습니다.
    # 단, 파일내 정의된 다른 서비스여야 연결이 가능합니다.
    # 만약 버전을 도커 컴포즈 3 버전 이상을 사용했다면 docker-compose.yml 안에 있는 서비스들은 별도로 지정하지 않으면 하나의 네트워크에 속합니다.

    # links:
    #  - database
 
    # 네트워크 모드를 설정할 수 있습니다. 기본적으로는 도커안의 내부 네트워크를 사용하게 됩니다.
    # network_mode: host
 
  database:
    # 'database'서비스에서 참조할 이미지
    image: mariadb:latest
    ports:
    - 3306:3306
    # 만약 컨테이너가 예상치 못한 일로 kill 되어도 바로 다시 띄울 수 있는 옵션 입니다.
    restart: always
    environment:
        # 환경 설정에 필요한 설정들 입니다.
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE : database
      MYSQL_USER: root
      MYSQL_PASSWORD: 1234
```

아래 내용은 docker-compose를 사용할 때 자주 사용되는 커맨드들입니다.
```jsx
# 도커 컴포즈 컨테이너들을 백그라운드로 띄우기
$ docker-compose up -d
 
# 도커 컴포즈 컨테이너들을 포어그라운드로 띄우기
$ docker-compose up
 
# 도커 컴포즈 컨테이너들을 내리기
$ docker-compose down
 
# 도커 컴포즈 컨테이너들을 다시 시작하기
$ docker-compose restart
 
# 도커 컴포즈 컨테이너들의 로그를 계속해서 읽기
$ docker-compose logs -f
 
# 도커 컴포즈 컨테이너들의 상태 확인
$ docker-compose ps
 
# 도커 컴포즈 설정을 확인
# 주로 -f 옵션으로 여러 개의 설정 파일을 사용할 때, 최종적으로 어떻게 설정이 적용되는지 확인해볼 때 유용합니다.
$ docker-compose config
 
# 다른 경로에 있는 도커 컴포즈 파일 사용
# 도커 컴포즈로 다른 이름이나 경로의 파일을 Docker Compose 설정 파일로 사용하고 싶다면 -f 옵션으로 명시를 해줍니다.
$ docker-compose -f /app/docker-compose.yml up
# 여러개의 도커 컴포즈 설정 파일을 사용할 수 있습니다. 이 때는 나중에 나오는 설정이 앞에 나오는 설정보다 우선하게 됩니다.
$ docker-compose -f docker-compose.yml -f docker-compose-test.yml up
```

이제, 도커 컴포즈를 실행해 보겠습니다.
아래로그는 위에서 작성한 docker-compose.yml 을 실행시켰을때 나오는 로그입니다. 
```jsx
PS C:\Users\admin\Desktop\volume> docker-compose up
Starting volume_database_1 ... done
Recreating volume_app1_1   ... done
Attaching to volume_database_1, volume_app1_1
database_1  | 2021-01-12 06:44:24+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 1:10.5.8+maria~focal started.
database_1  | 2021-01-12 06:44:24+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
database_1  | 2021-01-12 06:44:24+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 1:10.5.8+maria~focal started.
database_1  | 2021-01-12  6:44:24 0 [Note] mysqld (mysqld 10.5.8-MariaDB-1:10.5.8+maria~focal) starting as process 1 ...
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Using Linux native AIO
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Uses event mutexes
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Number of pools: 1
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Using crc32 + pclmulqdq instructions
database_1  | 2021-01-12  6:44:24 0 [Note] mysqld: O_TMPFILE is not supported on /tmp (disabling future attempts)
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Initializing buffer pool, total size = 134217728, chunk size = 134217728
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Completed initialization of buffer pool
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: 128 rollback segments are active.
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Creating shared tablespace for temporary tables
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: 10.5.8 started; log sequence number 45142; transaction id 20
database_1  | 2021-01-12  6:44:24 0 [Note] Plugin 'FEEDBACK' is disabled.
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
database_1  | 2021-01-12  6:44:24 0 [Note] InnoDB: Buffer pool(s) load completed at 210112  6:44:24
database_1  | 2021-01-12  6:44:24 0 [Note] Server socket created on IP: '::'.
database_1  | 2021-01-12  6:44:24 0 [Warning] 'proxies_priv' entry '@% root@08645019a0f9' ignored in --skip-name-resolve mode.
database_1  | 2021-01-12  6:44:25 0 [Note] Reading of all Master_info entries succeeded
database_1  | 2021-01-12  6:44:25 0 [Note] Added new Master_info '' to hash table
database_1  | 2021-01-12  6:44:25 0 [Note] mysqld: ready for connections.
database_1  | Version: '10.5.8-MariaDB-1:10.5.8+maria~focal'  socket: '/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
app1_1      |
app1_1      |   .   ____          _            __ _ _
app1_1      |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
app1_1      | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
app1_1      |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
app1_1      |   '  |____| .__|_| |_|_| |_\__, | / / / /
app1_1      |  =========|_|==============|___/=/_/_/_/
app1_1      |  :: Spring Boot ::                (v2.4.1)
app1_1      |
app1_1      | 2021-01-12 06:44:25.798  INFO 1 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication v0.0.1-SNAPSHOT using Java 1.8.0_275 on bf63fc08ed5f with PID 1 (/app/target/demo-0.0.1-SNAPSHOT.jar started by root in /app/target)
app1_1      | 2021-01-12 06:44:25.800  INFO 1 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
app1_1      | 2021-01-12 06:44:26.583  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
app1_1      | 2021-01-12 06:44:26.595  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
app1_1      | 2021-01-12 06:44:26.595  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.41]
app1_1      | 2021-01-12 06:44:26.641  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
app1_1      | 2021-01-12 06:44:26.641  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 772 ms
app1_1      | 2021-01-12 06:44:26.800  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
app1_1      | 2021-01-12 06:44:26.961  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
app1_1      | 2021-01-12 06:44:26.969  INFO 1 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 1.491 seconds (JVM running for 1.806)
```

이제, 도커 컴포즈로 생성한 컨테이너들을 확인해 보겠습니다.

실행한 도커 컴포즈 컨테이너의 상태를 조회하면 아래와 같이 나옵니다.

![/images/2021-01-13-docker-compose-guideline/1.png](/images/2021-01-13-docker-compose-guideline/1.png)

실행한 도커 컴포즈 컨테이너를 재시작 했을 시 나오는 로그입니다.

![/images/2021-01-13-docker-compose-guideline/2.png](/images/2021-01-13-docker-compose-guideline/2.png)

실행한 도커 컴포즈 컨테이너를 종료 했을 시에 나오는 로그 입니다.

![/images/2021-01-13-docker-compose-guideline/3.png](/images/2021-01-13-docker-compose-guideline/3.png)

## 예제2) ELK-스택 
이번 예제는 ELK-스택을 로컬에서 구동하기위한 docker-compose.yaml 입니다.
대부분의 항목들은 위 예제에서 다루었기에, 여기서는 configs 항목에 대해서만 추가적으로 더 확인하면 될 것 같습니다.

```jsx
version: '3'
 
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.8.1
    ports:
      - "9200:9200"
      - "9300:9300"
    # 서비스 별 config를 사용하여 서비스별로 config에 대한 내용을 따로 정의해서 사용합니다.
    # source / target / uid / gid / mode 로 각 config 에 대한 설정들을 지정할 수 있습니다.
    # source : Docker에있는 구성의 이름입니다.
    # target : 서비스의 작업 컨테이너에 마운트 할 파일의 경로와 이름입니다. 지정되지 않은 경우 기본값은 / <source>입니다.
    # uid 및 gid : 서비스의 작업 컨테이너에 마운트 된 구성 파일을 소유하는 숫자 UID 또는 GID입니다. 지정되지 않은 경우 둘 다 Linux에서 기본값은 0입니다. Windows에서는 지원되지 않습니다.
    # mode : 8 진수 표기법으로 서비스의 작업 컨테이너 내에 마운트 된 파일에 대한 권한입니다.
    configs:
      - source: elastic_config
        target: /usr/share/elasticsearch/config/elasticsearch.yml
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
      ELASTIC_PASSWORD: password
      discovery.type: single-node
 
  logstash:
    image: docker.elastic.co/logstash/logstash:7.8.1
    ports:
      - "5000:5000"
      - "9600:9600"
    configs:
      - source: logstash_config
        target: /usr/share/logstash/config/logstash.yml
      - source: logstash_pipeline
        target: /usr/share/logstash/pipeline/logstash.conf
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
 
  kibana:
    image: docker.elastic.co/kibana/kibana:7.8.1
    ports:
      - "5601:5601"
    configs:
      - source: kibana_config
        target: /usr/share/kibana/config/kibana.yml
 
 
# config들에 대한 관리를 여기서 할 수 있습니다.
# 전부 상대 경로로 진행 했습니다.
configs:
  elastic_config:
    file: ./elasticsearch/config/elasticsearch.yml
  logstash_config:
    file: ./logstash/config/logstash.yml
  logstash_pipeline:
    file: ./logstash/pipeline/logstash.conf
  kibana_config:
    file: ./kibana/config/kibana.yml
```

## 정리
이번에는 도커 컴포즈에서 쓰이는 항목들과 간단한 예제를 통해 도커 컴포즈 파일을 어떻게 생성하고, 확인할지 알아 보았습니다.
예제로는 간단한 서버-DB구성과 ELK스택에 대해서 작성을 해보았습니다. 
만약 도커 컴포즈 파일을 어떻게 생성하고, 시작해야 할지 모르시는 분들께 이 예제와 설명이 도움이 많이 되었으면 좋겠습니다.

## 링크
- [https://docs.docker.com/compose/compose-file/compose-file-v3/#build](https://docs.docker.com/compose/compose-file/compose-file-v3/#build)
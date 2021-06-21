# 엘라스틱서치 플러그인 IDE 개발환경 사례

## 소개

안녕하세요. 다나와의 김준우입니다. 이번에 포스팅 내용은 지난 3월에 출시된 Jetbrains의 Projector IDE 활용한 개발환경 사례입니다. 다나와에선 엘라스틱서치의 플러그인을 활용하는 API가 있습니다. 엘라스틱서치의 플러그인을 수정하거나, 디버깅이 필요할때 IDE를 사용하게 되면 빠르게 버그를 찾을 수 있고, 추가적으로 발생가능한 버그에 대해 미리 파악에 도움을 받을 수 있습니다. 하지만 IDE를 통해 엘라스틱서치를 실행하기 위해서는 JVM 옵션, 엘라스틱서치의 설정파일과 경로 설정을 로컬 환경에 맞게 구성해야되기 때문에 처음구성하게 되면 러닝 커브가 존재합니다. 이러한 러닝 커브를 줄이기 위해 저희는 프로젝터 + 도커 컨테이너 조합으로 개발환경을 구성하여 환경을 이미지화하여 개발서버, 로컬환경에서 제한없이 이미지기반으로 개발환경을 구성합니다.

### 구성도

원격지에 있는 호스트에 Projector IDE를 도커 컨테이너로 실행하고, Projector IDE에서는 엘라스틱서치 플러그인 프로젝트를 오픈합니다. 그리고 프로그래머는 웹브라우저를 통해 Intellij와 동일한 UI를 통해 엘라스틱서치 플러그인 프로젝트를 개발 및 디버깅할 수 있습니다.

![/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled.png](/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled.png)

## 구성방법

구성방법은 2가지로 소개합니다.

1. Projector를 도커 컨테이너 구축방법
2. 엘라스틱서치를 Projector IDE 실행방법


### 1. Projector 구축

프로젝터는 jetbrains 에서 제공하는 도구를 도커 컨테이너 이미지도 제공하고 있습니다.  저희는 idea-u를 사용하도록 하겠습니다.  

프로젝터 깃헙: [https://github.com/JetBrains/projector-docker](https://github.com/JetBrains/projector-docker)

### 1-1. 도커 컴포즈 작성

도커 이미지를 도커 명령어로 실행해도 되지만 매번 타이핑이 불편하기 때문에 도커 컴포즈 파일 로 실행하도록 하겠습니다. 

docker-compose.yml

```python
version: "3"
services:
  projecter:
    image: registry.jetbrains.team/p/prj/containers/projector-idea-u
    environment:
      - TZ=Asia/Seoul
    ports: 
      - 8887:8887
    restart: always
    volumes:
      - ./data:/home/projector-user/data
```

### 1-2. 접속 확인

해당 호스트의 8887 접속하면 친숙한 Intellij UI를 확인할 수 있습니다. 

![/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%201.png](/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%201.png)

### 2. 엘라스틱서치 IDE 실행

프로젝터는 이미 설치가 성공하였으면 엘라스틱서치를 ide기반으로 실행해보도록 하겠습니다. 사용할 엘라스틱서치 버전은 7.8.1이며, 다른 버전을 원할때는 설정파일들의 버전을 수정하시면 됩니다.

### 2-1. 엘라스틱서치 다운로드

엘라스틱서치를 실행할땐 config 하위 lib, logs 필수 파일이 존재합니다. 최소한으로 구성해도 되지만 편의상 아래 링크에서 해당하는 환경의 엘라스틱서치를 다운로드 한 후 압축을 해제하합니다. 

엘라스틱서치 다운로드 링크: [https://www.elastic.co/kr/downloads/elasticsearch](https://www.elastic.co/kr/downloads/elasticsearch)

![/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%202.png](/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%202.png)

### 2-2. jdk 설정

엘라스틱서치에 사용되면 버전으로 프로젝트를 설정합니다. 7.8.1 버전은 JDK 14 버전을 사용하기 때문에 14버전으로 변경합니다.  설치 편의상 엘라스틱서치에 포함된 jdk를 사용합니다.

![/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%203.png](/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%203.png)

### 2-3. build.gradle

jdk, elasticsearch 버전을 체크합니다.

```python
plugins {
	id 'java-library'
	id 'maven-publish'
}

group = 'com.danawa.search'
version = '1.0.0'

sourceCompatibility = JavaVersion.VERSION_14
targetCompatibility = JavaVersion.VERSION_14
compileJava.options.encoding = 'UTF-8'
compileTestJava.options.encoding = 'UTF-8'

repositories {
	jcenter()
	mavenCentral()
	mavenLocal()
}

dependencies {
	testImplementation 'junit:junit:4.1+'
	compile (
		[group: 'commons-io', name: 'commons-io', version: '1.3.2'],
		[group: 'org.elasticsearch', name: 'elasticsearch', version: '7.8.1'],
		[group: 'org.codelibs.elasticsearch.lib', name: 'plugin-classloader', version: '7.8.1'],
		[group: 'org.apache.logging.log4j', name: 'log4j-core', version: '2.11.1'],
		[group: 'org.json', name: 'json', version: '2019+'],
		[group: 'junit', name: 'junit', version: '4.1+', transitive: true],
	)
}

task copyToDependencies(type: Copy) {
	from (configurations.runtime) {
		include "json-*", "slf4j-api*"
	}
	from ("src/main/resources") {
		include "*.properties"
		include "*.policy"
		include "*.yml"
	}
	into "$buildDir/libs"
}

test {
	exclude '**/*'
}

build.dependsOn(copyToDependencies)
```

### 2-4. 실행 설정

실행 설정에서 application 하나를 생성한뒤 아래 설정을 입력합니다.

```python
// VM Option
-Des.path.home=/home/projector-user/data/elasticsearch-7.8.1 -Des.path.conf=/home/projector-user/data/elasticsearch-7.8.1/config -Des.logs.base_path=/home/projector-user/data/elasticsearch-7.8.1/logs -Dlog4j2.disable.jmx=true -Xms2g -Xmx2g

// Main Class
org.elasticsearch.bootstrap.Elasticsearch

```

![/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%204.png](/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%204.png)

### 2-4. 실행 후 노드 확인

실행 로그

![/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%205.png](/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%205.png)

클러스터 상태

![/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%206.png](/images/2021-06-21-projector-docker-elasticsearch-plugins/Untitled%206.png)

## 정리

Jetbrains의 Projector와 docker container를 활용하여 elasticsearch를 IDE로 실행을 해보았습니다. 유사한 도구로는 visual code가 존재하지만, intellij를 오래도록 사용하였고, 다양한 기능을 제공하기 때문에 친숙하게 사용할수 있었습니다. 개발환경을 이미지로 관리하게 되면 새로운 로컬에 환경을 구성하는데 러능 커브가 낮아지고, 동일한 개발 환경이 유지될 수 있을 거로 보여집니다. 또한 다나와는 개발환경은 윈도우지만 운영서버는 리눅스를 사용하기 때문에 환경에 대한 버그를 고려하지 않아도 될거로 보여집니다. 그리고 프로젝터는 현재 여러 사용자가 하나의 컨테이너로 접속하게 하나의 세션의 화면만 보여지고 있습니다.


## 참고링크

- JetBrains News: [https://blog.jetbrains.com/ko/blog/2021/03/24/projector-is-out/](https://blog.jetbrains.com/ko/blog/2021/03/24/projector-is-out/)
- Projector Docs: [https://jetbrains.github.io/projector-client/mkdocs/latest/](https://jetbrains.github.io/projector-client/mkdocs/latest/)
- Projector-Docker Github: [https://github.com/JetBrains/projector-docker](https://github.com/JetBrains/projector-docker)

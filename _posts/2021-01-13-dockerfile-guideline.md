---
layout: post
title:  "도커 파일 가이드라인"
description: "도커 파일을 생성함에 있어서 가이드라인이 필요하다고 생각하여 작성하여 공유를 합니다." 
date:   2021.01.13. 15:00:00
writer: "선지호"  
categories: Docker
---
## 소개

도커 파일을 통해 이미지를 생성할 때, 초심자라도 알기 쉽게 쓸 수 있도록 가이드라인을 작성해 보았습니다.
간단한 예제를 통해 이미지를 생성하실 수 있도록 진행해 보았으니, 글을 따라서 천천히 진행해 주시면 좋겠습니다.

## 도커 파일에서 쓰이는 명령어

제일 먼저, 도커 파일에서 쓰이는 명령어를 정리해 보았습니다.

- FROM : 베이스 도커 이미지를 지정합니다. 보통 OS 나 프로그래밍 언어 이미지를 지정합니다. docker hub에서 찾아 볼 수 있습니다.
- RUN : 쉘 커맨드를 도커 이미지에서 실행합니다.
- CMD : 쉘 커맨드를 도커 이미지에서 실행합니다. 단, 이 커맨드는 도커 컨테이너가 시작될 때 실행합니다
- LABEL : KEY-VALUE 의 LABEL을 생성합니다. docker inspect 명령어로 생성한 LABEL을 확인할 수 있습니다.
- EXPOSE : 도커 컨테이너 외부에 노출할 포트를 지정합니다, 단, 컨테이너에서 포트를 자동으로 open 하지 않기 때문에 컨테이너 실행 시 지정된 포트를 열어 주어야 합니다.
- ENV : 환경 변수를 지정할때 사용됩니다.
- ADD : 파일과 디렉토리를 호스트에서 지정한 도커이미지 디렉토리 안으로 복사 합니다. 만약 디렉토리가 없다면 새로 생성해서 복사합니다. 
디렉토리를 ADD하려면 끝이 "/"로 끝나야 합니다.

파일 이름과 디렉토리 이외에도 URL이 될수도 있습니다.

ADD 할려고 하는 파일이 tar 압축파일 이면 docker가 자동으로 압축을 풀어서 ADD 합니다.

ADD 할려고 하는 파일이나 디렉토리와 같은 이름의 파일이나 디렉토리가 벌써 image 상에 존재 한다면 덮어 씌우지 않습니다.

- COPY : ADD와 기본적으로 동일하나 URL지정이 불가하며 압축파일을 자동으로 풀어주지 않는다.
- ENTRYPOINT : 도커 이미지가 실행될 때 기본 커맨드를 지정합니다. 
CMD와의 차이점은 CMD 는 오버라이드가 가능하지만, ENTRYPOINT는 오버라이드 할 수 없습니다. 

docker run 커맨드로 추가하는 커맨드들은 ENTRYPOINT instruction에 지정된 커맨드 옵션으로 추가됩니다.

- VOLUME : 호스트의 폴더를 도커 컨테이너에 연결 시킬 수 있습니다. 즉, 도커 내부에서 호스트 컴퓨터에서 지정한 곳의 파일을 읽거나 쓰거나 할 수 있습니다. 보통 로그 저장에 사용됩니다.
- USER : 도커 이미지를 실행할 user를 지정해 줍니다. user뿐 아니라 UID/GID도 사용 가능합니다.
- WORKDIR : 현재 작업폴더를 지정합니다. 리눅스 cd 커맨드를 떠올리면 됩니다.
- ARG : docker build 커맨드로 docker image를 빌드할 때 설정할 수 있는 옵션들을 지정해 줍니다. 
계정비밀번호 등 민감한 정보는 이렇게 지정하지 않는 것을 추천합니다. 이유는 그대로 docker image에 지정한 내용이 남아 있기 때문입니다.

- SHELL :  디폴트로 지정되어 있는 shell 타입을 바꿀 수 있게 해줍니다. 디폴트 쉘은 ["/bin/sh", "-c"] 입니다
- ONBUILD : 도커 이미지가 다른 빌드의 기반으로 사용될 때 나중에 실행될 트리거 명령어를 도커 이미지에 추가합니다.
즉, docker build로 생성한 이미지가 아닌, 그 이미지를 기반으로 다른 이미지를 생성을 했을 때 사용되는 커맨드 입니다.

- STOPSIGNAL : 컨테이너를 종료하기 위해 컨테이너로 보낼 시스템 호출 신호를 설정합니다.
docker container stop 명령을 입력하면 도커 데몬이 컨테이너에게 signal을 보내 중지 하는데, 기본적으로 STOPSIGNAL을 명시하지 않을 경우 SIGTERM을 사용 하게 됩니다.

즉, container stop 할 때 SIGNAL을 지정할 수 있습니다

docker container stop 실행 시 SIGTERM signal을 받은 컨테이너가 프로세스를 정상적으로 종료할 수 있을때 까지 기다리게 되는데, 지정된 시간 (기본 10초, 사용자 지정 가능)동안 종료가 되지 않으면 SIGKILL을 전송합니다. 

- HEALTHCHECK : 컨테이너의 HEALTHCHECK를 사용하여 컨테이너의 프로세스 상태를 체크할 수 있습니다.
HEALTHCHECK [OPTIONS] CMD command : 컨테이너 내부에서 명령 실행하여 컨테이너 상태를 확인합니다.

HEALTHCHECK NONE : 베이스 이미지에서 상속된 상태 확인을 비활성화합니다

## 예제 : 스프링부트 프로젝트 도커 이미지 만들기 

### 도커 파일 생성

예제 스프링부트 프로젝트는 아래 링크에서 받으실 수 있습니다.
[Dockerfile_demo_project.zip](https://github.com/zozond/dockerfile-demo/raw/master/Dockerfile_demo_project.zip)

```jsx
# 자바 버전 8 이미지로부터 시작합니다.
FROM openjdk:8
 
# LABEL을 설정합니다
LABEL MAINTAINER="danawa"
 
# 환경변수를 설정합니다
ENV PROJECT_PATH="/app/target"
 
# 쉘 커맨드로 volume 폴더를 생성합니다.
RUN mkdir volume
 
# 해당 이미지의 리눅스 패키지 업데이트, 설치도 가능합니다. (ex) RUN apt-get install -y net-tools
RUN apt-get update

# 현재 도커파일이 위치한 폴더의 모든 데이터를 도커 컨테이너 내부 /app 폴더로 복사합니다.
ADD ./ /app
 
# 디렉토리 위치를 /app으로 옮깁니다.
WORKDIR /app
 
# 패키징을 합니다
RUN ./mvnw package

# 위에서 설정했던 환경변수에 등록된 내용을 사용하여 디렉토리 위치를 옮깁니다
WORKDIR $PROJECT_PATH
 
# 노출 시킬 포트 번호를 정합니다
EXPOSE 8080
 
# 호스트 에서 마운트 시킬 볼륨을 설정합니다
VOLUME [ "/volume" ]
 
# 시작 될 때 실행할 쉘 커맨드를 설정합니다.
CMD ["java", "-jar", "demo-0.0.1-SNAPSHOT.jar"]
```

위의 내용으로 도커 파일을 만들어 스프링부트 프로젝트 최상단에 위치 시킵니다.
이후 도커파일이 있는 위치에서 아래의 커맨드를 입력하게 되면, 로컬 도커 이미지에 설정한 이름으로 이미지가 추가 됩니다.

```jsx
# 로컬 도커에 이미지를 생성합니다
$ docker build -t demo:latest .
 
$ docker build -t <이미지 이름>:<태그> <Dockerfile 위치>

# 로컬 도커에 저장된 이미지를 조회합니다
$ docker images 
```


이렇게 생성된 도커 이미지를 컨테이너 실행 시킬 때는 아래와 같이 입력하시면 됩니다.

```jsx
$ docker run -d -p 8080:8080 -v C:/Users/admin/Desktop/volume:/volume demo:latest
 
# -d 옵션은 백그라운드로 실행입니다.
# foreground로 실행하려면 -d 대신 -it 를 입력해주세요
$ docker run -d -p <호스트 포트>:<컨테이너 포트> -v <호스트 디렉토리 PATH>:<컨테이너 디렉토리 PATH> <이미지 이름>:<태그>
```

위의 내용 대로 입력하신다면, 아까 생성했던 도커 이미지가 호스트포트의 8080포트와 매핑이 되어 호스트PC에서 도커 컨테이너 안으로 접속이 가능하게 됩니다.

이번에는 제대로 실행이 되어있는지 한번 확인해 보겠습니다.

아래와 같이 입력하신다면, 생성된 도커 컨테이너들에 대한 정보와 그 중 하나의 컨테이너에 대한 상세 정보를 확인해 볼 수 있습니다.

저는 아까 생성한 도커 컨테이너를 확인해 보았습니다.

```jsx
# 현재 실행되어 있는 도커 컨테이너 목록을 불러옵니다. 정상적으로 삭제 되지 않은 컨테이너도 보고 싶으시면 -a 옵션을 추가하시면 됩니다.
$ docker ps
 
$ docker inspect 561bd45c0452
 
$ docker inspect <컨테이너 ID 혹은 컨테이너 이름>
```

위의 내용을 쉘 커맨드로 입력하면, 아래와 같은 내용이 나옵니다.

내용은 각 PC별로 다를 예정이니 간단하게 확인만 하고 넘어가시면 됩니다.

LABEL을 확인하려면 LABELS 항목을 찾아 KEY-VALUE로 저장된 LABEL을 확인 할 수 있습니다.

환경변수에 대한 내용 또한 여기에 가지고 있으니 확인 하고자 하실 때, 위의 커맨드를 입력하여 확인하시면 됩니다.

```jsx
[
    {
        "Id": "561bd45c0452f0bb25e5c9d54359415e1c3763ad18fe49bb775dba448763fd34",
        "Created": "2021-01-12T03:47:09.8411872Z",
        "Path": "java",
        "Args": [
            "-jar",
            "demo-0.0.1-SNAPSHOT.jar"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 2598,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-01-12T03:47:13.0327536Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:946e765a09e0523e75ce9c65ffcf29fb156fde5ae07c10ac742e1414b3fdc463",
        "ResolvConfPath": "/var/lib/docker/containers/561bd45c0452f0bb25e5c9d54359415e1c3763ad18fe49bb775dba448763fd34/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/561bd45c0452f0bb25e5c9d54359415e1c3763ad18fe49bb775dba448763fd34/hostname",
        "HostsPath": "/var/lib/docker/containers/561bd45c0452f0bb25e5c9d54359415e1c3763ad18fe49bb775dba448763fd34/hosts",
        "LogPath": "/var/lib/docker/containers/561bd45c0452f0bb25e5c9d54359415e1c3763ad18fe49bb775dba448763fd34/561bd45c0452f0bb25e5c9d54359415e1c3763ad18fe49bb775dba448763fd34-json.log",
        "Name": "/fervent_varahamihira",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "C:/Users/admin/Desktop/volume:/volume"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "8080/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/a3f9fbd61a3605511a31edc1249e78f5a9a2d5f8596e700d4933eacfb3acdccb-init/diff:/var/lib/docker/overlay2/m55ic5lspsm5ijnrdy0zogm0q/diff:/var/lib/docker/overlay2/1r8yv2e0elgh4yzsijsyoqbra/diff:/var/lib/docker/overlay2/kjifrcvv3xwlq99iou0c4brct/diff:/var/lib/docker/overlay2/z54oq2t1r0tlpywngj3f8s5su/diff:/var/lib/docker/overlay2/7tqphj25nf5hj1e9lb6o95bk5/diff:/var/lib/docker/overlay2/z0xxc9kopzlinq2voy9ripzue/diff:/var/lib/docker/overlay2/0018158ae4401294b163a8f39ea2cba68d20745977fa83d894581ae77af837c3/diff:/var/lib/docker/overlay2/96316f10a15c03abb632a70ee80b4d44a45176e2e20177452e9758b24486a6e1/diff:/var/lib/docker/overlay2/4b1f0c2223dc531a1293582e88df0352a45ceccbf415a5f1067145f7acf576ed/diff:/var/lib/docker/overlay2/f390a49c0bae6bc783161cb67141532a54201a8adb3f88a50f6146ab993068e4/diff:/var/lib/docker/overlay2/3c2f4562c2644a4b04d8bb7b70ef88d1339713c70299bcd514471bbe09375a01/diff:/var/lib/docker/overlay2/d50c51d7294c0b3c5d797fb25a84d9c72e8e952289c5b94e3708bd95587de998/diff:/var/lib/docker/overlay2/28aa62e7fb4af77800ee7611da09dcf73bc6700e7ee5a03f965c30e4155d5b59/diff",
                "MergedDir": "/var/lib/docker/overlay2/a3f9fbd61a3605511a31edc1249e78f5a9a2d5f8596e700d4933eacfb3acdccb/merged",
                "UpperDir": "/var/lib/docker/overlay2/a3f9fbd61a3605511a31edc1249e78f5a9a2d5f8596e700d4933eacfb3acdccb/diff",
                "WorkDir": "/var/lib/docker/overlay2/a3f9fbd61a3605511a31edc1249e78f5a9a2d5f8596e700d4933eacfb3acdccb/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "C:/Users/admin/Desktop/volume",
                "Destination": "/volume",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Config": {
            "Hostname": "561bd45c0452",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "8080/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/openjdk-8/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "JAVA_HOME=/usr/local/openjdk-8",
                "JAVA_VERSION=8u275",
                "PROJECT_PATH=/app/target"
            ],
            "Cmd": [
                "java",
                "-jar",
                "demo-0.0.1-SNAPSHOT.jar"
            ],
            "Image": "demo:latest",
            "Volumes": {
                "/volume": {}
            },
            "WorkingDir": "/app/target",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "MAINTAINER": "danawa"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "9a47ca0291b7cd5d59759927fb8360167fb817e30b032ff03a5775cb5b981011",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "8080/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8080"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/9a47ca0291b7",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "17e6351be1fd07827ec19e4bdf664e66dcba1a0f97383eb3e975d4f754d9c458",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "7d668bb2ae1fa872eea4e99ed9097edba6ac09f032bacd73589989c3e04ac19d",
                    "EndpointID": "17e6351be1fd07827ec19e4bdf664e66dcba1a0f97383eb3e975d4f754d9c458",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

이번에는 볼륨이 정확히 마운트 되었는지 한번 확인해보겠습니다.

```jsx
$ docker exec -it 561bd45c0452 bash
 
$ docker exec -it <컨테이너 ID 혹은 컨테이너 이름> <리눅스 쉘 이름>
```

위의 커맨드를 입력하면 컨테이너 안으로 직접 접근이 가능합니다.

저는 위의 커맨드를 입력하고, 이전에 볼륨으로 잡았던 /volume 디렉토리를 확인했습니다.

해당 폴더에는 확인하기 쉽게 txt파일을 하나 넣었고, 정상적으로 공유되었음을 확인 할 수 있었습니다.

![/images/2021-01-13-dockerfile-guideline/1.png](/images/2021-01-13-dockerfile-guideline/1.png)

![/images/2021-01-13-dockerfile-guideline/2.png](/images/2021-01-13-dockerfile-guideline/2.png)


이번에는 생성한 도커 컨테이너를 제거 해보겠습니다.

아래의 커맨드를 입력한다면, 도커 컨테이너를 제거할 수 있습니다.

```jsx
$ docker stop 561bd45c0452

$ docker rm 561bd45c0452
 
$ docker stop <컨테이너 ID 혹은 컨테이너 이름>

$ docker rm <컨테이너 ID 혹은 컨테이너 이름>
```

이후 "docker ps -a" 를 입력하여 도커 컨테이너를 확인해보면 해당 컨테이너가 제거된 것을 확인 할 수 있습니다.

### 도커 레지스트리에 생성한 이미지 푸시

1. 생성한 이미지이름 앞에 도커 레지스트리 경로를 붙여 같은 이미지 이지만 다른 태그의 이미지를 생성합니다

여기서 사용한 도커 레지스트리 경로는 사설 레지스트리이기 때문에, 만약 따로 사설 레지스트리를 이용하신다면 다른 경로를 입력해주세요.

만약 도커 허브 레지스트리를 이용하신다면, 먼저 레파지토리를 이미지 이름으로 생성해주세요

```jsx
$ docker tag demo:latest dcr-ui.danawa.io/demo:latest

$ docker tag demo:latest <도커 허브 아이디>/demo:latest

```

2. 확인해봅니다
```jsx
$ docker images
```

3. 변경한 이미지를 푸시 해줍니다
```jsx
$ docker push dcr-ui.danawa.io/demo:latest

$ docker push <도커 허브 아이디>/<이미지 이름>:<태그>

```

4. push한 도커 이미지를 받을때는 아래와 같이 사용하면 됩니다.
```jsx
$ docker pull dcr-ui.danawa.io/demo:latest

$ docker pull <도커 허브 아이디>/<이미지 이름>:<태그>
```

아래의 이미지는 도커레지스트리에 추가할 때의 로그입니다.
![/images/2021-01-13-dockerfile-guideline/3.png](/images/2021-01-13-dockerfile-guideline/3.png)


성공적으로 사설 레지스트리에 올라간 것을 확인 할 수 있었습니다.
![/images/2021-01-13-dockerfile-guideline/4.png](/images/2021-01-13-dockerfile-guideline/4.png)


만약 도커 허브 레지스트리를 사용하신다면, 아래에 접속하셔서 로그인을 한 이후, 정상적으로 푸시가 되었는지 확인해주세요.

[https://hub.docker.com/repositories](https://hub.docker.com/repositories) 

![/images/2021-01-13-dockerfile-guideline/5.png](/images/2021-01-13-dockerfile-guideline/5.png)


## 정리

도커 파일에서 쓰이는 커맨드와 간단한 예제를 통해 도커 이미지를 어떻게 생성하고, 확인할지 알아 보았습니다.

또한, 도커 레지스트리에 어떻게 로컬에 있는 이미지를 넣는지도 실습해 보았습니다.

만약 사설 레지스트리를 따로 구축해서 사용하신다면 이미지를 넣을때 위에서 설명한 방식을 사용하셔서 넣으시면 편할 것 같습니다.

도커 파일을 어떻게 사용해야 할지 모르시는 분들께 도움이 많이 되었으면 좋겠습니다.

## 링크
- [https://hub.docker.com/](https://hub.docker.com/)
- [https://docs.docker.com/engine/reference/builder/#from](https://docs.docker.com/engine/reference/builder/#from)
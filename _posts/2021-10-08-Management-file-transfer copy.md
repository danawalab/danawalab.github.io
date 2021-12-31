---
layout: post
title:  "서비스 운영플랫폼 #6 - 파일 전송"
description: "서비스 운영플랫폼의 파일 탐색기를 통해 파일 업로드/다운로드를 진행합니다"
date:   2021.10.08. 14:30:00
writer: "반윤성"
categories: Management
---
## 소개
서비스 운영플랫폼에서 원격 서버와 파일 전송을 위해 구현된 파일 탐색기와 각 기능들을 소개합니다.

원격 서버에 접속하여 파일을 가져오거나, 올릴 때 불편한 부분이 있었습니다. 이 점을 개선하기 위해 별도의 파일 탐색기 화면을 구성했고, 이 페이지를 통해 원격 서버에 접근하여 업/다운로드를 진행할 수 있습니다.

## 시스템 구성 및 세부 동작과정
### 시스템 구성
개발 초기에 파일 업/다운로드 구현을 위해 sFTP 방식을 비롯한 여러가지 방법을 조사했습니다. 원격서버와 SSH 연결을 통해 명령어를 이용하여 지정된 URL을 호출하는 방법으로 통해 서버간 파일에 접근할 수 있었고, 기존 시스템에서 SSH를 사용하고 있어서 조금 더 적용하기 수월하다는 점과 관리와 확장성 측면에서도 이점이 있었습니다. 

검토 결과 SSH를 사용하여 개발을 진행하게 되었습니다. 이를 위해 파일 전송 시스템을 다음과 같이 구성했습니다. 사용자가 위치한 로컬 시스템과 중간에서 요청과 응답을 전달해주는 운영관리 서비스가 있고 파일 전송의 목적지인 원격 서버가 존재합니다.

![/images/2021-10-08-Management-file-transfer/4.jpg](/images/2021-10-08-Management-file-transfer/4.jpg)

### 세부 동작과정
대상 파일의 저장이 어떻게 이뤄지는지 알아보겠습니다. Nextjs, NodeJs에서 파일 업로드 다운로드 기능을 구현하기 위해 formidable이라는 미들웨어 라이브러리를 사용했습니다. 업로드 및 다운로드 과정에서 대상 파일의 formData가 들어오면 다시 이것을 파싱하여 임시 폴더에 적재합니다.

다음으로 운영관리 서비스와 원격서버를 SSH로 연결합니다. 연결된 후에 curl 명령어를 사용하여 대상이 되는 파일을 주소로 접근하여 파일이 전송됩니다. 파일 전송 기능은 이 방법들을 사용하여 이뤄집니다.  

파일 업로드 API 호출시 운영관리 서비스에 formData를 전송하고 다시 파싱을 거쳐서 임시 폴더에 저장합니다. 이어서 curl 명령어를 실행하여 원격서버가 지정된 URL을 호출하여 운영관리서비스에 적재된 파일을 받아올 수 있도록 합니다.

파일 다운로드는 업로드와 반대 방향으로 진행됩니다. 다운로드 API는 curl -F 명령을 통해 원격 서버의 파일을 운영관리 서비스로 전달합니다. 마찬가지로 formidable을 이용한 파싱을 통해 임시폴더로 위치시킵니다. 그 다음 서비스에서 로컬로 해당하는 파일을 가져오는 API를 호출하여 브라우저를 통해 다운로드를 받습니다.

```JSX
// 파일 전송을 위한 curl 호출 메소드
let process_cmd = (id, processType, filename, path, filekey) => {
    const uploadUrl = localhost_url + `/api/servers/${id}/download/${filekey}?fileName=${filename}`;
    const downloadUrl = localhost_url + `/api/servers/${id}/explorer`;

    if (processType === "upload") {
        return `curl "${encodeURI(uploadUrl)}" > "${path + filename}"`;
    } else if (processType === "download") {
        return `curl -F "file=@${path + filename}" ${downloadUrl}?filekey=${filekey}`;
    }
}
```


## 기능 설명

![/images/2021-10-08-Management-file-transfer/2.jpg](/images/2021-10-08-Management-file-transfer/2.jpg)

### 디렉토리 조회
화면 중간에 위치한 패널에서는 기본적으로 현재 위치가 최상단에 고정됩니다. 'ls -al' 명령어를 통해 조회한 내용을 가져오며 조회 내용으로는 권한, 유저, 변경 일자, 파일명이 표시됩니다. 하지만 다른 형태로 조회하거나 다른 내용을 조회하고 싶다면 명령어 창을 통해 기본적인 명령어들이 수행되기 때문에 수행 결과를 전달받아 볼 수도 있습니다.

![/images/2021-10-08-Management-file-transfer/1.jpg](/images/2021-10-08-Management-file-transfer/1.jpg)

### 각 단위별 특징
- 해더영역에는 사용자가 접속한 서버의 정보가 표시됩니다.

- 명령어 입력 시 현재 디렉토리 내의 파일들에 대해 자동완성을 지원하며 입력 후 엔터키 혹은 검색버튼으로 실행합니다.

- 현재 자동완성이 지원되는 명령어는 cd, get, rm, rmdir, cat 입니다.

- 하단부 업로드/다운로드 파일리스트에서는 파일을 업로드하고 다운로드 받은 파일과 같은 전송 파일들이 표시됩니다.

- 파일 업로드는 드래그앤드랍 방식과 클릭하여 업로드하는 두 가지 방식이 있습니다.

- 파일 업로드 위치는 기본적으로 현재 위치가 기준이 됩니다.

- 파일 다운로드는 현재 위치가 아닌 다른 경로에 있는 파일도 다운로드 가능합니다(ex. get /apps/explorer/query.sql)

- 파일 다운로드 시 디렉토리, 심볼릭 파일은 다운로드가 제한됩니다.

![/images/2021-10-08-Management-file-transfer/3.jpg](/images/2021-10-08-Management-file-transfer/3.jpg)

### 업로드/다운로드 파일리스트
하나의 파일은 각각 업로드/다운로드 구분, 파일명, 파일 사이즈, 진행 단계, 시작-종료일시, 버튼으로 구성됩니다. 그리고 해당 로우의 버튼을 통해 기능을 제어할 수 있습니다.

### 파일 업로드 방법

- 드래그앤드랍 방식의 경우 로컬 시스템에서 파일을 선택하여(다중 파일도 선택 가능) 패널에 올립니다
- 탐색기를 통해 업로드하는 방식의 경우 표시된 영역을 클릭해 창을 띄워 업로드를 진행할 수 있습니다.
- 목록에 올린 후에 [업로드] 버튼을 클릭하면 업로드가 시작됩니다.
- Local -> Server -> Remote 순으로 진행상황이 업데이트 됩니다.
- '완료'로 표기되면서 업로드가 완료됩니다.

### 파일 다운로드 방법

- 명령어창에 get '파일이름' ex) get aaa.txt
- Remote -> Server -> Local 순으로 진행상황이 업데이트 됩니다.
- 바로 다운로드가 진행되며 브라우저를 통해 다운로드가 완료됩니다.
- '완료'로 표기되면서 다운로드가 완료됩니다.

## 정리
 서비스 운영플랫폼에서 파일 탐색기를 통해 파일 업로드/다운로드 기능의 동작과 사용 방법을 알아보았습니다. 명령어나 버튼 클릭으로 손쉽게 파일 전송이 가능해서 유용하게 사용할 수 있습니다. 서비스 운영시에 도움이 되었으면 좋겠습니다.
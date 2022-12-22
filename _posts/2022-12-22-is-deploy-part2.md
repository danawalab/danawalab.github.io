---
layout: post
title: "is-deploy 트러블 슈팅 및 회고"
description: "is-deploy를 개발하고 테스트하면서 겪었던 이슈 및 트러블슈팅 회고록입니다"
date: 2022.12.22.
writer: "장민규"
categories: Common
---

# Agent 회고

개발은 agent 먼저 개발하고 다음에 console로 개발이 진행되었습니다.

## go언어를 선택한 이유

제가 에이전트를 개발할 때 go 언어를 선택한 이유로는 그냥 devops 하면 go 언어 go 언어 하면 devops가 떠올라서입니다.    
는... 농담이고 실은 나중에 아파치 톰캣이 아닌 다른 환경에서도 자바가 없어도 에이전트가 실행되길 원했고 또한
개발 생산성도 고려했습니다.

```go
// go code
func fileWrite(text string) {
	file, _ := os.Open("/home/test.txt")
	defer file.Close()
	
	_, err = file.write([]byte(text))
	... 생략
}
```

```java
// java code
private void fileWrite() {
    File file = new File("/home/test.text");
    FileWrite fw = new FileWrite(file);
    BufferedWriter writer = new BufferedWriter(fw);
    ... 생략
    writer.close();
}
```

간단하게 파일을 읽고 그다음에 쓰는 예제 코드를 준비했습니다.

go 언어는 file을 열고 close를 바로 다음 줄에 `defer`를 붙여 close를 하면 알아서 해당 함수가 리턴하기 직전에 호출하여 close를 해주지만
java는 함수 끝나기 전에 close를 적어줘야 해서 잊으면 대참사가 일어나기도 합니다.

또한 자바 예제 코드에 try catch가 빠져 있음에도 file write 하는데 코드 길이의 차이가 극심한 것을 볼 수 있습니다.

무엇보다 is-deploy 프로젝트를 하기 전에 ISMS 보안 패치를 해야 하는 프로젝트가 있었는데 레거시 스프링으로 로그 파일 읽다 보니
많은 코드에서 try catch 묶어주고 close를 해주기 바쁜 기억이 있어서 go 언어를 선택해 에이전트를 개발했습니다.

## 테스트 코드 

```go
func TestFindByName(t *testing.T) {
	worker := "worker1"
	node := GetJsonToTest()

	length := len(node.PodList)
	var newArray []ExcludeMap

	for i := 0; i < length; i++ {
		pod := node.PodList[i]
		name := pod.Name

		if worker == name {
			exLength := len(pod.LbMap)

			for j := 0; j < exLength; j++ {
				key := pod.LbMap[j].Key
				value := pod.LbMap[j].Value

				newArray = append(newArray, ExcludeMap{key, value})
			}
			break
		}
	}

	println("TestFindByName = ", newArray, len(newArray))
}

func TestTailLog(t *testing.T) {
    cmd := exec.Command("tail", "-n",  "10", "../../sample/catalina.out")

    output, _ := cmd.Output()

    fmt.Println(string(output))
}
```

에이전트를 개발 당시 테스트 코드를 작성할 때 저는 위 코드와 같이 작성했는데요.

TDD에서 테스트 코드는 success, fail 두 개 작성하고 로그나 프린트 출력을 하는 것은 권장하지 않고
`assertThat` 같은 코드를 작성하라고 학습을 했었는데 A라는 데이터가 들어가면 B라는 데이터가 나오는 이런 데이터가 중요한 백엔드면 모르겠는데
API 호출되면 설정 파일을 읽고 설정파일에 정의된 수정할 파일 경로로 가서 수정할 파일을 수정하거나 값을 가져오는 devops의 툴을 만들다 보니   

읽고 쓰는 파일이 json 이면 json의 파일 구조가 변경될 때마다 테스트 코드를 리팩토링이 아닌 갈아엎는 수준으로 작업이 필요했고    
또는 `exec.Command()`를 통해 쉘 스크립트를 실행하거나 tail을 찍어보든 해당 값들은 success, fail와 같은 테스트 코드를 작성하기에는 부적합하여
대부분의 테스트 코드의 결괏값을 `println`으로 출력하여 테스트 코드를 작성했습니다.

## 좋았던 점

go 언어는 제가 is-deploy를 하면서 처음 접해본 거는 아니었습니다, 취업 준비 당시 나만의 남들과 다른 기술 스택을 가지고 싶어 공부했던 언어였습니다.   
덕분에 이번 프로젝트를 진행할 때 언어 선택에 있어 빠르게 선택하고 일정 안에 개발을 완료했었습니다.

is-deploy에는 적용은 안됐지만 go 언어를 공부할 때 이해가 잘 안됐던 go에서 인터페이스, 덕타입 인터페이스 등 go에 대한 전체적인 이해도가 올라갔습니다.

뿐만 아니라 쉘 스크립트 한 번도 작성한 적 없었던 저는 이번 프로젝트를 통해서 쉘 스크립트랑 덩달아 리눅스까지 많이 친해진 계기가 되었습니다.

## 아쉬운 점

`tail -f` 기능을 구현하고 했습니다, 개발안은 log 파일을 백그라운드에서 tail을 통해 고 루틴으로 실시간으로 보다 변경 점이 있으면 해당 부분을 채널을 통해 일정 로그를 쌓아서 console에 보내는 기능을
구현하고자 했는데 실패 했습니다.

원인은 고루틴과 채널의 이해도 부족 그리고 `exec.Command("tail", "-f", filePath)`이 output이 제대로 안됐던 점을 뽑았습니다.

그래서 대안으로 `exec.Command("tail", "-n", line, filePath)`로 console에서 agent의 log api를 특정 간격마다 호출하여 `tail -f`의 기능을 유사하게 구현했습니다.

# Console 회고

console에 대한 회고록은 다음 포스팅인 next.js 개발기로 동료와 함께 찾아뵙겠습니다. 

# 트러블 슈팅

[아파치 톰캣 로드밸런싱 상태에서 WAS 무중단 자동 배포하기](https://danawalab.github.io/common/2022/12/21/is-deploy-part1.html)   
포스팅에서 **Q. 왜 /etc/apache2/conf.d 에 uriworkermap.properties 안 만드나요?** 에 해당하는 답변이자 트러블 슈팅 이이갸 입니다.

`/etc/apache2/conf.d`에 uriworkermap.properties를 만들고 `/etc/apache2/conf.d/vhost.conf` 설정에서 JkMountFile을
`/etc/apache2/conf.d/uriworkermap.properties` 로 지정하고 아파치에 설정을 적응하기 위해 재시작하고서 
deploy.danawa.com 이라는 사이트가 있을 때 해당 사이트는 JkMountFile을 통해 uriworkermap.properties에 정의한 `/*=balancer` 값을 보고 1번 톰캣하고 2번 톰캣 로드밸런싱이
되어있을 겁니다.

다음에 is-deploy를 통해서 uriworkermap.properties를 수정하거나 직접 vi를 통해서 uriworkermap.properties를 
`/*=worker1` 또는 `/*=worker2`로 수정해도 제대로 아파치에서 동적 리로드를 못해서 계속 로드밸런싱 상태를 유지하고 아파치를 재시작해야지만 그제야 변경된
uriworkermap.properties의 값에 맞게 변경되었습니다.
이 문제는 로컬에서 기술검증과 is-deploy 테스트가 끝나고 실제 개발서버에 올려서 테스를 진행 도중 발견한 이슈였습니다.

## 원인 분석
해당 이슈가 발생한 원인으로 3가지를 뽑았습니다.

1. 아파치, 톰캣, mod_jk 버전 문제
2. 설정이 잘못 됨
3. 권한 문제

**1번째**

아파치, 톰캣, mod_jk 버전이 문제일 경우도 있어
개발서버의 우분투 버전, 아파치, 톰캣 그리고 mod_jk 버전을 확인하고, 로컬 테스트 환경에서 버전도 확인했지만
아파치랑 mod_jk는 버전이 동일했고 우분투와 톰캣은 마이너 버전만 달랐습니다.

**2번째**

여기서 부터 팀원이 합류해서 같이 트러블 슈팅을 진행 했습니다.

뭔가 설정이 빠진 게 있거나 로컬 테스트 환경에서 설정은 아파치 초기에 설치했을 때 있는 그대로에서 설정을 진행한 반면
개발서버 설정은 이미 다나와 환경에 맞게 변경된 설정이라 맞춰서 하려면 다른 추가 설정이 필요한가 싶어서 급하게 root 권한을 받아서 설정 파일 하나하나 들여다봤습니다.

그러나 결국 큰 차이를 발견하지도 못했고 여러 번 설정을 바꾸고 아파치 재시작을 해도 이슈는 사라지지 않았습니다.

**3번째**

버전 문제도, 설정 문제도 아니였습니다.

mod_jk.log를 확인해 봤습니다.

![mod_jk.log](/images/2022-12-22-is-deploy-part2/1.png)

해당 로그에는 Unable to stat <uriworkermap.properties 경로> (errno=13) 으로 남겨져 있었고 

![mod_jk_code](/images/2022-12-22-is-deploy-part2/2.png)

mod_jk의 원본 소스 코드를 확인해 본 결과 로그는 uriworkermap.properties 파일을 정상적으로 읽지 못하여 발생하는 로그였습니다.

![path](/images/2022-12-22-is-deploy-part2/3.png)

경로를 다시 확인해도 uriworkermap.properties 파일을 정상적으로 존재했습니다.

## 권한 문제

개발 서버 설정을 진행할 때 저는 서버 관리팀과 협업을 통해 설정을 진행하면서 권한에 대해서 이야기를 나눈 적이 있었고
해당 이슈가 권한 문제 때문이 아닐까 싶어 팀원하고 정보를 공유했습니다.

처음에는 아파치 폴더와 작동은 root로 설정이었습니다, 거기서 uriworkermap.properties를 추가하고 해당 파일을 agent를 통해 수정하기 위해서는
root 권한이면 안됐습니다. 그래서 그룹을 user로 주고 `/etc/apache2`, `/etc/apache2/conf.d` 폴더의 그룹도 user로 설정하였습니다.

저와 팀원은 이를 바탕으로 다시 이슈를 확인 했습니다, 저는 그룹을 다시 root로 원복 하고서 재실행을 해봤지만 해당 이슈는 똑같이 존재해 다시 user로 변경했었던
반면 팀원은 `/home/conf` 경로에 uriworkermap.properties를 넣고 설정 파일에 경로 수정 후 재시작을 했습니다.

![path2](/images/2022-12-22-is-deploy-part2/4.png) 

![mod_jk.log2](/images/2022-12-22-is-deploy-part2/6.png)

그 결과 정상적으로 아파치가 uriworkermap.properties 파일을 동적으로 리로드 하는 것을 확인할 수 있었습니다.

## 이유

아파치를 mpm worker로 실행을 했는데 아래 사진을 보면
하위 프로세스들이 nobody로 실행이 되어서 `/etc/apache2/conf.d` 경로 uriworkermap.properties 파일을 권한 문제로 인해 동적으로 리로드를 못하는 것으로 생각하고 있습니다.

![apahce](/images/2022-12-22-is-deploy-part2/7.png)


# 정리

is-deploy를 개발하면서 많은 팀과의 협업이 있었고 덕분에 충분하다 생각했었던 커뮤니케이션이 더욱더 발전하였습니다.

또한 개발에 대한 트러블 슈팅이 아닌 서버에 대한 트러블 슈팅을 겪고 나서 이슈 해결 능력이 좋아졌는지
이번 주 월요일인 2022.12.19일에 이슈가 생겨 트러블 슈팅을 진행 했는데 이에 대한 트러블 슈팅은 다음 포스팅을 통해 더 자세하게 올리도록 하겠습니다.

트러블슈팅을 할 때는 참 스트레스 받는다 생각했는데 해결하고 나서의 그 쾌감과 뒤에서 물려오는 재미를 잊을 수 없습니다.   
물론 아무 이슈 없는 게 좋지만요, 그래도 성장하는데 좋은 경험이였다 생각합니다.








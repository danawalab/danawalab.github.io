---
layout: post
title:  "SVN을 Git으로 마이그레이션"
description: "프로젝트를 진행할 때 각자 업무를 나눠서 맡은 부분을 개발하게 됩니다. 각자 개발한 코드 혹은 문서를 하나의 관리 도구에서 통합적으로 버전별로 관리하게 되는 것을 형상관리(Configuration Management) 혹은 버전관리(Version Management)라고 부릅니다. 형상관리 방식에는 크게 중앙집중식과 분산관리식으로 나뉘는데 대표적으로 사용되는  도구가 중앙집중관리식에는 Subversion (이하 SVN) 그리고 분산관리식은 GIT을 사용합니다.   "
date:   2020.02.07.
writer: "김준우"
categories: CM
---
# 소개

프로젝트를 진행할 때 각자 업무를 나눠서 맡은 부분을 개발하게 됩니다. 각자 개발한 코드 혹은 문서를 하나의 관리 도구에서 통합적으로 버전별로 관리하게 되는 것을 형상관리(Configuration Management) 혹은 버전관리(Version Management)라고 부릅니다. 형상관리 방식에는 크게 중앙집중식과 분산관리식으로 나뉘는데 대표적으로 사용되는  도구가 중앙집중관리식에는 Subversion (이하 SVN) 그리고 분산관리식은 GIT을 사용합니다.   

SVN의 장점은 직관적입니다. 모든 사용자들이 중앙서버에 있는 같은 자료를 받아오고 내가 Commit을 하는 순간 모든 사람에게 공유가 됩니다. 이러한 방식의 단점은 만약 두 사람이 하나의 파일을 동시에 수정하고 commit하였을때 충돌이 발생합니다. 

Git의 장점은 모든 작업이 로컬에서 이루어지고 네트워크 사용은 원격 저장소로 Push 할 때 한번만 이루어 집니다. 여러 사람이 프로젝트를 branch, merge 상호작용을 잘 할 수 있도록 편의성을 제공하고 있습니다. 원격 저장소의 내용이 로컬에 저장되어 있기 때문에 원격저장소에 장애가 발생하여도 로컬에서 복구가 용이합니다.


## SVN을 GIT으로 변경이유

SVN도 좋은 형상관리 도구 이지만 Git을 사용하게 되면 다양한 서비스를 이용할 수 있습니다. 예를 들면 원격저장소를 github으로 사용하게 되면 서버관리가 필요가 없고 파일을 영구적으로 보관합니다. gitlab을 사용하게 되면 CI/CD등 다양한 서비스를 사용할 수 있게 됩니다.

## SVN을 GIT으로 이관작업

테스트방식은 docker를 활용하여 SVN 서버를 가동하여 임의 프로젝트를 SVN에 commit한뒤 로컬 git으로 마이그레이션을 합니다. 이후 gitlab으로 저장하는 작업을 진행해보겠습니다.


### 도커로 SVN서버 구성
아래 명령어를 실행하여 SVN 서버를 컨테이너를 생성합니다.
```
$ docker run -d --name svn-server -p 80:80 -p 3690:3690 elleflorio/svn-server
```


신규 레파지토리 생성
```
$ docker exec -it svn-server svnadmin create sample
```

또는 컨테이너에 접근하여 생성할 수 있습니다. 파일 타입은 fsfs와 bdb 두가지를 제공하지만 bdb는 deprecate되었습니다. 

```
$ docekr exec -it svn-server sh 


$ svnadmin create --fs-type fsfs /home/svn/<레파지토리명>
```
    

레파지토리 권한 설정

```
vi <레파지토리명>/conf/svnserve.conf

    # 인증받지않은 사용자 권한
    anon-access=none
    # 인증받은 사용자 권한
    auth-access=write
    # passwd 파일기반으로 사용자 관리
    password-db=passwd
```


신규 사용자 생성
```
$ docker exec -t svn-server htpasswd -b /etc/subversion/passwd <username> <password>
```

또는 <레파지토리명>/conf/passwd 경로 직접 사용자명과 비밀번호를 파일에 작성해야합니다.   
<사용자 아이디>=<사용자 비밀번호>


trunk, branches, tags 디렉토리 구성하기

```
$ export SVN_EDITOR=vi
$ svn mkdir svn://localhost/<레파지토리명>/trunk --username <사용자 아이디> --password <사용자 비밀번호>
$ svn mkdir svn://localhost/<레파지토리명>/branches --username <사용자 아이디> --password <사용자 비밀번호>
$ svn mkdir svn://localhost/<레파지토리명>/tags --username <사용자 아이디> --password <사용자 비밀번호>
```


클라이언트에서 http으로 접근이 필요하여 권한을 부여합니다.

```
$ chown -R svn:www-data <레파지토리명>
```

### SVN을 로컬 Git으로 마이그레이션

svn 사용자정보를 생성해야 합니다. 이번 블로그에서는 TortoiseSVN을 사용하겠습니다. svn의 레파지토리를 로컬에 체크아웃의 자세한 설명은 생략하도록 하겠습니다. 체크아웃 받은 폴더에서 오른쪽 클릭하여 Show log를 선택합니다.

![/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled.png](/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled.png)

선택하면 아래 이미지처럼 화면에서 Statistics를 선택합니다.

![/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%201.png](/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%201.png)

Graph Type을 Commits by author를 선택합니다. 그러면 오른쪽에 컴밋했던 사용자명이 보입니다. 

![/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%202.png](/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%202.png)

SVN에서 컴밋했던 사용자명을 users.txt 파일을 규칙에 맞게 작성합니다.

작성규칙: SVN 사용자명= Github 사용자명 <Github 사용자이메일>

```
jwk = joonwoo8888 <dev.joonwoo@gmail.com>
```

SVN 레파지토리에서 로컬 Git으로 복사합니다. 옵션정보는 대소문자를 구분합니다. 정확하게 입력해야합니다.

git svn clone <SVN URL> --no-metadata -A <users.txt 경로> -T <trunk명> -b <branches명> -t <tags명> <로컬git경로>

```
$ git svn clone <SVN URL> --no-metadata -A users.txt -T trunk -b branches -t tags ./sample-git
```
    

정상적으로 복사되었으면 source tree를 이용하여 확인합니다. 원격 하위에 Subversion이 생성되었고 Branches, Tags가 보입니다. 그리고 히스토리도 정상적으로 보입니다.

![/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%203.png](/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%203.png)

원격에 표시되는 branches, tags는 svn에 연결이 되어 있습니다. 앞으로 git으로 사용하기 위해서는 svn의 branches, tags의 history 정보와 새로 만든 git branches, tags의 history가 연결되어야 합니다.

git branches 생성 하겠습니다. git clone을 통해 snv branch, tag가 생겼습니다. 아래 명령어를 통해 git branch, tag를 생성합니다. 주의 사항으로는 git branch -r 명령어를 통해 목록을 확인하여 경로에 마춰 branch를 입력해야합니다.
```
$ for branch in `git branch -r | grep "branches/" | sed 's/ branches\///'`; do
    git branch $branch refs/remotes/$branch
done
```
    

git tags 생성하겠습니다.

```
$ for tag in `git branch -r | grep "tags/" | sed 's/ tags\\///'| cut -d'/' -f 3`; do
    git tag -a -m "Converting SVN tags" $tag origin/tags/$tag
done
```


source tree를 이용하여 확인해보겠습니다.

![/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%204.png](/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%204.png)

현재 상태에서 github, gitlab등으로 푸시하여 저장할 수 있습니다.

테스트로 gitlab에 푸시하여 브렌치정보를 확인합니다.

![/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%205.png](/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%205.png)

태그도 정상적으로 저장되었는지 확인합니다.

![/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%206.png](/images/2020-02-07-ConfigureManagement-SvnToGit-Migration/Untitled%206.png)


정리

SVN은 사용이 직관적이고 사용이 단순하다는 장점이 있지만 git을 통해 다양한 서비스를 이용할 수 없는 단점이 있어 svn을 git으로 마이그레이션 해보는 테스트를 해보았습니다. git 명령어를 통해 svn의 프로젝트를 가져오는 기능이 포함되어 있어  어렵지 않게 진행되었습니다.

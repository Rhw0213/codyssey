# 오디세이 학습

## 미션 개요 요약

### 🎯 핵심 목표
"내 컴퓨터에서만 돌아가는 문제"를 없애고, 팀원 누구나 동일하게 실행·배포·디버깅할 수 있는 재현 가능한 개발 환경을 직접 구축하는 것

### 🛠️ 다루는 3가지 핵심 도구
리눅스 CLI (터미널)	작업 디렉토리·권한 관리
Docker (컨테이너)	격리된 실행 환경 구성
Git / GitHub		버전 관리 및 협업
### 📋 실습 흐름
터미널로 작업 디렉토리와 권한 정리
Docker 설치 및 점검 → 컨테이너 실행/관리
웹 서버를 Dockerfile로 컨테이너화
포트 매핑으로 접속 확인
바인드 마운트 / 볼륨으로 "변경 반영"과 "데이터 영속성" 검증

### 💡 학습 포인트
단순 따라치기가 ❌ → 실행 결과(로그/접속/데이터 유지)로 흐름 확인 ✅
구조적 원칙 이해:
이미지와 컨테이너의 분리
격리된 실행 환경
포트·스토리지 연결 방식
"왜 이런 설계가 필요한지" 설명 가능한 형태로 정리

한 줄 요약: 터미널·Docker·Git을 직접 손으로 세팅하며, **"여러 번 실행해도 똑같이 재현되는 환경"**을 만드는 사고방식을 체득하는 미션이에요! 💪

## 터미널 조작 로그 기록

```bash
rhw02133670@c4r1s8 codyssey % pwd
/Users/rhw02133670/codyssey

rhw02133670@c4r1s8 codyssey % ls -al
total 24
drwxr-xr-x   5 rhw02133670  rhw02133670   160 Jul 28 11:20 .
drwxr-x---+ 23 rhw02133670  rhw02133670   736 Jul 28 11:13 ..
-rw-r--r--@  1 rhw02133670  rhw02133670  6148 Jul 28 11:10 .DS_Store
drwxr-xr-x  13 rhw02133670  rhw02133670   416 Jul 28 10:57 .git
-rw-r--r--   1 rhw02133670  rhw02133670  1260 Jul 28 10:41 README.md

rhw02133670@c4r1s8 codyssey % mkdir test
rhw02133670@c4r1s8 codyssey % cd test

rhw02133670@c4r1s8 test % touch hello.txt
rhw02133670@c4r1s8 test % cat hello.txt

rhw02133670@c4r1s8 test % mv hello.txt hi.txt
rhw02133670@c4r1s8 test % mv hi.txt hi_backup.txt

rhw02133670@c4r1s8 test % cp hi_backup.txt hi.txt
rhw02133670@c4r1s8 test % rm hi.txt

rhw02133670@c4r1s8 test % cd ..
rhw02133670@c4r1s8 codyssey % ls
README.md	test

rhw02133670@c4r1s8 codyssey % rm -r test

```

## 리눅스 파일 권한 변경 실습

```bash
rhw02133670@c4r1s8 codyssey % touch test.txt
rhw02133670@c4r1s8 codyssey % ls -l test.txt 
-rw-r--r--  1 rhw02133670  rhw02133670  0 Jul 28 11:39 test.txt
rhw02133670@c4r1s8 codyssey % chmod 700 test.txt 
rhw02133670@c4r1s8 codyssey % ls -l test.txt 
-rwx------  1 rhw02133670  rhw02133670  0 Jul 28 11:39 test.txt

rhw02133670@c4r1s8 codyssey % mkdir myFolder
rhw02133670@c4r1s8 codyssey % ls -ld myFolder 
drwxr-xr-x  2 rhw02133670  rhw02133670  64 Jul 28 13:07 myFolder
rhw02133670@c4r1s8 codyssey % chmod 700 myFolder 
rhw02133670@c4r1s8 codyssey % ls -ld
drwxr-xr-x  7 rhw02133670  rhw02133670  224 Jul 28 13:07 .
rhw02133670@c4r1s8 codyssey % ls
README.md	myFolder	test.txt
rhw02133670@c4r1s8 codyssey % ls -al
total 24
drwxr-xr-x   7 rhw02133670  rhw02133670   224 Jul 28 13:07 .
drwxr-x---+ 23 rhw02133670  rhw02133670   736 Jul 28 13:05 ..
-rw-r--r--@  1 rhw02133670  rhw02133670  6148 Jul 28 12:56 .DS_Store
drwxr-xr-x  13 rhw02133670  rhw02133670   416 Jul 28 13:05 .git
-rw-r--r--   1 rhw02133670  rhw02133670  3664 Jul 28 13:05 README.md
drwx------   2 rhw02133670  rhw02133670    64 Jul 28 13:07 myFolder
-rwx------   1 rhw02133670  rhw02133670     0 Jul 28 11:39 test.txt
rhw02133670@c4r1s8 codyssey % ls -ld myFolder 
drwx------  2 rhw02133670  rhw02133670  64 Jul 28 13:07 myFolder
rhw02133670@c4r1s8 codyssey % chmod -R 755 myFolder 
rhw02133670@c4r1s8 codyssey % ls -al
total 24
drwxr-xr-x   7 rhw02133670  rhw02133670   224 Jul 28 13:07 .
drwxr-x---+ 23 rhw02133670  rhw02133670   736 Jul 28 13:05 ..
-rw-r--r--@  1 rhw02133670  rhw02133670  6148 Jul 28 12:56 .DS_Store
drwxr-xr-x  13 rhw02133670  rhw02133670   416 Jul 28 13:05 .git
-rw-r--r--   1 rhw02133670  rhw02133670  3664 Jul 28 13:05 README.md
drwxr-xr-x   2 rhw02133670  rhw02133670    64 Jul 28 13:07 myFolder
-rwx------   1 rhw02133670  rhw02133670     0 Jul 28 11:39 test.txt
rhw02133670@c4r1s8 codyssey % 
```

파일 생성 및 초기 상태 확인
명령어: touch test.txt / ls -l test.txt
초기 권한: -rw-r--r--
상태 분석:
소유자 (rhw02133670): rw- (읽기 및 쓰기 가능)
그룹 (rhw02133670): r-- (읽기만 가능)
기타 사용자 (Others): r-- (읽기만 가능)
설명: 파일을 생성하면 시스템의 기본 설정(umask)에 따라 소유자에게는 편집 권한이, 타인에게는 읽기 권한만 부여된 상태로 생성됩니다.

권한 변경 실행
명령어: chmod 700 test.txt
숫자 모드(700)의 의미:
7 (소유자): 4(읽기) + 2(쓰기) + 1(실행) = 모든 권한 부여
0 (그룹): 권한 없음
0 (기타): 권한 없음

변경 후 상태 확인 및 비교
명령어: ls -l test.txt
변경 권한: -rwx------
결과 분석:

소유자: 기존 rw-에서 rwx로 변경되어 이제 이 파일을 실행할 수도 있게 되었습니다.
그룹 및 기타 사용자: 기존의 읽기(r--) 권한이 모두 제거되어, 소유자 외에는 이 파일의 내용을 보거나 수정할 수 없는 보안이 강화된 상태가 되었습니다.

chmod -R 755 myFolder
하위 폴더까지 전부 권환 변경


## Docker 기본 운영 명령 수행

### 1. 이미지 검색 (docker search)
```bash
rhw02133670@c4r1s8 codyssey % docker search ubuntu
NAME                DESCRIPTION                                     STARS     OFFICIAL
ubuntu              Ubuntu is a Debian-based Linux operating sys…   17862     [OK]
ubuntu/squid        Squid is a caching proxy for the Web. Long-t…   129       
ubuntu/nginx        Nginx, a high-performance reverse proxy & we…   141       
... (생략)
```
> Docker Hub에서 `ubuntu` 관련 이미지를 검색합니다.  
> `OFFICIAL [OK]` 표시는 공식 이미지를 의미하며, `STARS`는 인기도를 나타냅니다.

---

### 2. 이미지 다운로드 (docker pull)
```bash
rhw02133670@c4r1s8 docker_test % docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
ed819469700f: Pull complete 
a3679419df18: Pull complete 
Digest: sha256:3131b4cc82a783df6c9df078f86e01819a13594b865c2cad47bd1bca2b7063bb
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```
> 태그를 생략하면 자동으로 `latest` 버전을 받습니다.  
> 이미지는 여러 **레이어(layer)** 로 나뉘어 다운로드됩니다.

---

### 3. 컨테이너 실행 (docker run -it)
```bash
rhw02133670@c4r1s8 codyssey % docker run -it ubuntu /bin/bash 
root@a2911da37e52:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@a2911da37e52:/# exit
exit
```
> `-i`(interactive) + `-t`(tty) 옵션으로 **컨테이너 내부 셸에 접속**합니다.  
> 프롬프트가 `root@컨테이너ID`로 바뀌며, 격리된 우분투 환경에 진입합니다.  
> `exit`로 컨테이너를 빠져나옵니다.

---

### 4. 이미지 목록 확인 (docker images)
```bash
rhw02133670@c4r1s8 codyssey % docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    de7345b16e94   2 weeks ago   100MB
```
> 로컬에 다운로드된 이미지 목록을 보여줍니다.  
> `IMAGE ID`, `SIZE` 등 이미지 메타정보를 확인할 수 있습니다.

---

### 5. 실행 중인 컨테이너 확인 (docker ps)
```bash
rhw02133670@c4r1s8 codyssey % docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
> **현재 실행 중인** 컨테이너만 보여줍니다.  
> `exit`로 컨테이너를 종료했기 때문에 목록이 비어 있습니다.  
> 💡 종료된 컨테이너까지 보려면 `docker ps -a`를 사용합니다.

## 컨테이너 실행 실습

### 1. hello-world 실행 (설치 검증)
```bash
rhw02133670@c4r1s8 codyssey % docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
Digest: sha256:c3cbe1cc1aa588a64951ac6286e0df7b27fe2e6324b1001c619bb358770c0178
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
```
> ✅ **`Hello from Docker!` 메시지 출력 = Docker 설치·실행 환경 정상 작동!**  
> 로컬에 이미지가 없어 자동으로 pull → 컨테이너 생성 → 실행 → 출력의 흐름을 확인할 수 있습니다.

**Docker 동작 4단계 (출력 내용 정리)**
1. Docker 클라이언트가 Docker 데몬에게 요청
2. 데몬이 Docker Hub에서 `hello-world` 이미지를 다운로드
3. 이미지로 컨테이너를 생성하고 실행
4. 실행 결과를 터미널에 출력

---

## 2. Ubuntu 컨테이너 접속 (심화 실습)
```bash
rhw02133670@c4r1s8 codyssey % docker run -it ubuntu /bin/bash       
root@f6f9cf42c238:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
> `hello-world` 안내 메시지에서 추천한 **"더 도전적인 실습"** 을 이어서 진행!  
> `-it` 옵션으로 우분투 컨테이너 셸에 접속하여 리눅스 디렉토리 구조를 확인했습니다.


## 기존 Dockerfile 기반 커스텀 이미지 제작


## 🎯 에러 원인 분석

에러 메시지의 핵심:
```
ERROR [2/2] COPY index.html /usr/share/nginx/html/index.html
"/index.html": not found
```
👉 **"index.html 파일을 찾을 수 없다"** 는 뜻이에요!

---

### 🔍 왜 못 찾았을까요?

빌드 위치의 폴더 구조를 보면:
```
codyssey/
├── Dockerfile
├── README.md
└── src/
    └── index.html   ← 파일이 여기 있음!
```

그런데 Dockerfile에는 이렇게 적혀 있었어요:
```dockerfile
COPY index.html ...   ← codyssey 바로 아래에서 찾음 → 없음! ❌
```

> `index.html`이 `src` 폴더 **안에** 있는데, 밖에서 찾으니 못 찾은 거예요!

---

## ✅ 해결 방법: 경로에 `src/` 추가

```dockerfile
FROM nginx:latest

# 커스텀 포인트: src 폴더 안의 index.html을 교체
COPY src/index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

> **핵심**: `COPY 원본경로 대상경로`에서  
> **원본경로**는 Dockerfile 위치 기준으로 적어야 해요!  
> → `index.html` ❌ → `src/index.html` ✅

---

## 📌 정리

| `index.html not found` | 파일이 `src/` 안에 있음 | `COPY src/index.html ...` |

---


### 🚀 다시 빌드하기

```bash
docker build -t my-nginx .
```

```
=> [internal] load build definition from Dockerfile                                                      0.1s
 => => transferring dockerfile: 295B                                                                      0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                                           0.8s
 => [internal] load .dockerignore                                                                         0.1s
 => => transferring context: 2B                                                                           0.0s
 => [internal] load build context                                                                         0.1s
 => => transferring context: 318B                                                                         0.0s
 => [1/2] FROM docker.io/library/nginx:latest@sha256:5a88c9c45479443d7be2eadc894b4ed0a9801bae03d97a5760a  4.0s
 => => resolve docker.io/library/nginx:latest@sha256:5a88c9c45479443d7be2eadc894b4ed0a9801bae03d97a5760a  0.1s
 => => sha256:5a88c9c45479443d7be2eadc894b4ed0a9801bae03d97a5760ae13b5c2005942 10.23kB / 10.23kB          0.0s
 => => sha256:4e5db4761e0ff445f7fd29aad680ad28e8abf7d204895557f145d65535abcc1c 9.09kB / 9.09kB            0.0s
 => => sha256:db4f612f385437d11eb26620a4f1d7efb3ff44e1296a3c21540b30454e6e2bf3 2.29kB / 2.29kB            0.0s
 => => sha256:062e450697faa5f02a3a74eba9864ee4d79bc9cfbd65769fc6cdff2c05c6a053 29.78MB / 29.78MB          0.7s
 => => sha256:82454cdbf456a77f9ff1bb88b121c2a739e38c30ea689c135c7cca6249eabe4e 33.33MB / 33.33MB          1.0s
 => => sha256:3c7ab7949321f47c96fc0918f9f72e8f51bd452cdef1e0dad9599880317380b9 626B / 626B                0.7s
 => => sha256:cacfcdd01f309c65d69372716e799ea741065ac1b1e60880b3a6981ae105cb55 955B / 955B                1.0s
 => => extracting sha256:062e450697faa5f02a3a74eba9864ee4d79bc9cfbd65769fc6cdff2c05c6a053                 1.0s
 => => sha256:b6698f04e005497a7f495c0719358d43890cb3997ad7b4ab0b06748247c574a3 403B / 403B                1.1s
 => => sha256:2bedaf25031a24fb70b9dc2d56cb17139186d1ae5fd2054ecbd0dfe1a69585ba 1.21kB / 1.21kB            1.2s
 => => sha256:d26f27cc8c41e321394cb3c9a80915d90d5f1f1d3cbbbcda3be00f13c53b041e 1.40kB / 1.40kB            1.3s
 => => extracting sha256:82454cdbf456a77f9ff1bb88b121c2a739e38c30ea689c135c7cca6249eabe4e                 0.7s
 => => extracting sha256:3c7ab7949321f47c96fc0918f9f72e8f51bd452cdef1e0dad9599880317380b9                 0.0s
 => => extracting sha256:cacfcdd01f309c65d69372716e799ea741065ac1b1e60880b3a6981ae105cb55                 0.0s
 => => extracting sha256:b6698f04e005497a7f495c0719358d43890cb3997ad7b4ab0b06748247c574a3                 0.0s
 => => extracting sha256:2bedaf25031a24fb70b9dc2d56cb17139186d1ae5fd2054ecbd0dfe1a69585ba                 0.0s
 => => extracting sha256:d26f27cc8c41e321394cb3c9a80915d90d5f1f1d3cbbbcda3be00f13c53b041e                 0.0s
 => [2/2] COPY src/index.html /usr/share/nginx/html/index.html                                            0.4s
 => exporting to image                                                                                    0.2s
 => => exporting layers                                                                                   0.1s
 => => writing image sha256:f8532955e1df869f9af96de669342940851bb5f08be21c334b47428d4c7dde0b              0.0s
 => => naming to docker.io/library/my-nginx                                                               0.0s

```

---

## ✅ 빌드 성공 확인

로그에서 성공 신호들을 짚어볼게요!

```
=> [2/2] COPY src/index.html /usr/share/nginx/html/index.html   ✅
```
> 아까 실패했던 그 부분! 이번엔 **에러 없이 통과**했어요! 경로 수정이 딱 맞았죠 👏

```
=> exporting to image                                          ✅
=> => naming to docker.io/library/my-nginx                     ✅
```

```
[+] Building 5.9s (7/7) FINISHED
```
> **7단계 전부 완료(FINISHED)** = 완벽한 빌드! 🎯

---

## 🔍 이미지가 잘 만들어졌는지 확인하기

`ls`로 파일은 보셨으니, 이번엔 **Docker 이미지 목록**을 확인해봐요!

```bash
docker images
```

여기에 `my-nginx`가 보이면 확실하게 성공한 거예요! 아마 이렇게 나올 거예요:

```
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
my-nginx     latest    f8532955e1df   ...             ...
```

> `IMAGE ID`가 로그에 나온 `f8532955e1df...`와 일치하면 완벽! ✅

---

Dockerfile 내용

# 1. 기존 베이스 이미지 선택 (공식 NGINX)
FROM nginx:latest

# 2. 커스텀 포인트: 기본 웹페이지를 내 파일로 교체
COPY index.html /usr/share/nginx/html/index.html

# 3. 문서화용: 80번 포트 사용을 명시
EXPOSE 80

FROM nginx:latest → 공식 NGINX 이미지를 베이스로 사용
COPY → NGINX 기본 페이지 위치에 내 index.html을 덮어씀 (커스텀 핵심!)
EXPOSE 80 → 이 컨테이너가 80번 포트를 쓴다고 명시 (문서화 역할)

---

## 실행 결과

![포트 매핑 결과](portMapping.png)



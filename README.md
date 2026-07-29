# Codyssey 학습 미션 — 재현 가능한 개발 환경 구축

---

## 1. 미션 개요

### 1.1 핵심 목표

**"내 컴퓨터에서만 돌아가는 문제"를 없애는 것.**
팀원 누구나 동일하게 실행·배포·디버깅할 수 있는 **재현 가능한 개발 환경**을 직접 손으로 구축한다.

### 1.2 다루는 도구

| 도구 | 역할 |
|---|---|
| 리눅스 CLI (터미널) | 작업 디렉토리·권한 관리 |
| Docker (컨테이너) | 격리된 실행 환경 구성 |
| Git / GitHub | 버전 관리 및 협업 |

### 1.3 실습 흐름

1. 터미널로 작업 디렉토리와 권한 정리
2. Docker 설치 및 점검 → 컨테이너 실행·관리
3. 웹 서버를 Dockerfile로 컨테이너화
4. 포트 매핑으로 접속 확인
5. 바인드 마운트 / 볼륨으로 "변경 반영"과 "데이터 영속성" 검증

### 1.4 학습 포인트

- 단순 따라치기 ❌ → **실행 결과(로그 / 접속 / 데이터 유지)로 흐름 확인** ✅
- 구조적 원칙 이해
  - 이미지와 컨테이너의 분리
  - 격리된 실행 환경
  - 포트 · 스토리지 연결 방식
- "왜 이런 설계가 필요한지"를 설명 가능한 형태로 정리

> **한 줄 요약:** 터미널 · Docker · Git을 직접 세팅하며 *여러 번 실행해도 똑같이 재현되는 환경*을 만드는 사고방식을 체득하는 미션.

---

## 2. 리눅스 CLI 실습

### 2.1 디렉토리 · 파일 조작

#### 현재 위치 및 목록 확인

```console
rhw02133670@c4r1s8 codyssey % pwd
/Users/rhw02133670/codyssey

rhw02133670@c4r1s8 codyssey % ls -al
total 24
drwxr-xr-x   5 rhw02133670  rhw02133670   160 Jul 28 11:20 .
drwxr-x---+ 23 rhw02133670  rhw02133670   736 Jul 28 11:13 ..
-rw-r--r--@  1 rhw02133670  rhw02133670  6148 Jul 28 11:10 .DS_Store
drwxr-xr-x  13 rhw02133670  rhw02133670   416 Jul 28 10:57 .git
-rw-r--r--   1 rhw02133670  rhw02133670  1260 Jul 28 10:41 README.md
```

- `pwd` : 현재 작업 디렉토리의 절대 경로 출력
- `ls -al` : 숨김 파일(`.git`, `.DS_Store`)까지 포함한 상세 목록 출력

#### 생성 → 이름 변경 → 복사 → 삭제

```console
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

| 명령어 | 동작 | 확인 결과 |
|---|---|---|
| `mkdir test` | 디렉토리 생성 | `ls` 결과에 `test` 등장 |
| `touch hello.txt` | 빈 파일 생성 | `cat` 출력이 비어 있음 (내용 없음) |
| `mv` | 이름 변경 / 이동 | `hello.txt` → `hi.txt` → `hi_backup.txt` |
| `cp` | 복사 | 백업본에서 `hi.txt` 복원 |
| `rm` / `rm -r` | 파일 / 디렉토리 삭제 | 삭제 후 목록에서 사라짐 |

---

### 2.2 파일 권한 관리 (chmod)

#### ① 파일 생성 및 초기 상태 확인

```console
rhw02133670@c4r1s8 codyssey % touch test.txt
rhw02133670@c4r1s8 codyssey % ls -l test.txt 
-rw-r--r--  1 rhw02133670  rhw02133670  0 Jul 28 11:39 test.txt
```

초기 권한 **`-rw-r--r--`** 분석

| 대상 | 권한 | 의미 |
|---|---|---|
| 소유자 | `rw-` | 읽기 · 쓰기 가능 |
| 그룹 | `r--` | 읽기만 가능 |
| 기타 사용자 | `r--` | 읽기만 가능 |

→ 파일을 생성하면 시스템 기본 설정(`umask`)에 따라 **소유자에게는 편집 권한, 타인에게는 읽기 권한만** 부여된 상태로 만들어진다.

#### ② 권한 변경 실행

```console
rhw02133670@c4r1s8 codyssey % chmod 700 test.txt 
rhw02133670@c4r1s8 codyssey % ls -l test.txt 
-rwx------  1 rhw02133670  rhw02133670  0 Jul 28 11:39 test.txt
```

숫자 모드 `700`의 의미

| 자리 | 값 | 계산 | 결과 |
|---|---|---|---|
| 소유자 | `7` | 4(읽기) + 2(쓰기) + 1(실행) | 모든 권한 |
| 그룹 | `0` | — | 권한 없음 |
| 기타 | `0` | — | 권한 없음 |

**변경 결과 분석**

- 소유자: `rw-` → `rwx` — 이제 이 파일을 **실행**까지 할 수 있게 되었다.
- 그룹 · 기타: 기존 읽기(`r--`) 권한이 모두 제거되어, **소유자 외에는 내용을 보거나 수정할 수 없는** 보안이 강화된 상태가 되었다.

#### ③ 디렉토리 권한과 재귀 변경 (-R)

```console
rhw02133670@c4r1s8 codyssey % mkdir myFolder
rhw02133670@c4r1s8 codyssey % ls -ld myFolder 
drwxr-xr-x  2 rhw02133670  rhw02133670  64 Jul 28 13:07 myFolder

rhw02133670@c4r1s8 codyssey % chmod 700 myFolder 
rhw02133670@c4r1s8 codyssey % ls -ld myFolder 
drwx------  2 rhw02133670  rhw02133670  64 Jul 28 13:07 myFolder
```

```console
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
```

- `ls -ld` : 디렉토리 **자체**의 정보를 확인 (내부 목록이 아니라)
- `chmod -R 755` : **하위 폴더·파일까지 전부** 재귀적으로 권한 변경
  - `drwx------` → `drwxr-xr-x` 로 되돌아온 것을 최종 목록에서 확인

---

## 3. Docker 기초

### 3.1 설치 검증 — hello-world

```console
rhw02133670@c4r1s8 codyssey % docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
Digest: sha256:c3cbe1cc1aa588a64951ac6286e0df7b27fe2e6324b1001c619bb358770c0178
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

✅ `Hello from Docker!` 출력 = **Docker 설치 · 실행 환경 정상 작동**

로컬에 이미지가 없어 **자동 pull → 컨테이너 생성 → 실행 → 출력** 흐름이 그대로 확인된다.

**출력이 설명하는 Docker 동작 4단계**

1. Docker 클라이언트가 Docker 데몬에게 요청
2. 데몬이 Docker Hub에서 `hello-world` 이미지를 다운로드
3. 이미지로 컨테이너를 생성하고 실행
4. 실행 결과를 터미널로 스트리밍 출력

---

### 3.2 기본 운영 명령

#### ① 이미지 검색 — `docker search`

```console
rhw02133670@c4r1s8 codyssey % docker search ubuntu
NAME                DESCRIPTION                                     STARS     OFFICIAL
ubuntu              Ubuntu is a Debian-based Linux operating sys…   17862     [OK]
ubuntu/squid        Squid is a caching proxy for the Web. Long-t…   129       
ubuntu/nginx        Nginx, a high-performance reverse proxy & we…   141       
... (생략)
```

- Docker Hub에서 이미지를 검색
- `OFFICIAL [OK]` = 공식 이미지, `STARS` = 인기도

#### ② 이미지 다운로드 — `docker pull`

```console
rhw02133670@c4r1s8 docker_test % docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
ed819469700f: Pull complete 
a3679419df18: Pull complete 
Digest: sha256:3131b4cc82a783df6c9df078f86e01819a13594b865c2cad47bd1bca2b7063bb
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

- 태그를 생략하면 자동으로 `latest`를 받는다
- 이미지는 여러 **레이어(layer)** 로 나뉘어 다운로드된다

#### ③ 컨테이너 실행 — `docker run -it`

```console
rhw02133670@c4r1s8 codyssey % docker run -it ubuntu /bin/bash 
root@a2911da37e52:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@a2911da37e52:/# exit
exit
```

- `-i`(interactive) + `-t`(tty) 로 컨테이너 내부 셸에 접속
- 프롬프트가 `root@컨테이너ID` 로 바뀌며 **격리된 우분투 환경**에 진입
- `exit` 로 컨테이너를 빠져나옴

#### ④ 이미지 목록 — `docker images`

```console
rhw02133670@c4r1s8 codyssey % docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    de7345b16e94   2 weeks ago   100MB
```

로컬에 내려받은 이미지 목록과 `IMAGE ID` · `SIZE` 등 메타정보를 확인.

#### ⑤ 실행 중인 컨테이너 — `docker ps`

```console
rhw02133670@c4r1s8 codyssey % docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

- **실행 중인** 컨테이너만 표시된다
- 위에서 `exit` 로 종료했기 때문에 목록이 비어 있음
- 💡 종료된 컨테이너까지 보려면 `docker ps -a`

---

## 4. 커스텀 이미지 제작 (Dockerfile)

### 4.1 최종 Dockerfile

```dockerfile
# 1. 베이스 이미지 선택 (공식 NGINX)
FROM nginx:latest

# 2. 커스텀 포인트: 기본 웹페이지를 내 파일로 교체
COPY src/index.html /usr/share/nginx/html/index.html

# 3. 문서화용: 80번 포트 사용을 명시
EXPOSE 80
```

| 지시어 | 역할 |
|---|---|
| `FROM nginx:latest` | 공식 NGINX 이미지를 베이스로 사용 |
| `COPY` | NGINX 기본 페이지 위치에 내 `index.html`을 덮어씀 (**커스텀 핵심**) |
| `EXPOSE 80` | 이 컨테이너가 80번 포트를 쓴다고 명시 (문서화 역할) |

---

### 4.2 트러블슈팅 — `COPY ... not found`

#### 에러 메시지

```
ERROR [2/2] COPY index.html /usr/share/nginx/html/index.html
"/index.html": not found
```

#### 원인 분석

빌드 시점의 폴더 구조는 다음과 같았다.

```
codyssey/
├── Dockerfile
├── README.md
└── src/
    └── index.html   ← 파일이 여기 있음!
```

그런데 Dockerfile에는 이렇게 적혀 있었다.

```dockerfile
COPY index.html ...   ← codyssey 바로 아래에서 찾음 → 없음! ❌
```

`index.html`이 `src/` 안에 있는데 **바깥에서 찾았기 때문에** 실패한 것.

#### 해결

```dockerfile
COPY src/index.html /usr/share/nginx/html/index.html
```

| 증상 | 원인 | 해결 |
|---|---|---|
| `index.html not found` | 파일이 `src/` 안에 있음 | `COPY src/index.html ...` |

> **핵심 원칙:** `COPY 원본경로 대상경로` 에서 **원본경로는 Dockerfile 위치(빌드 컨텍스트) 기준**으로 적어야 한다.
> `index.html` ❌ → `src/index.html` ✅

---

### 4.3 재빌드 결과

```console
% docker build -t my-nginx .
 => [internal] load build definition from Dockerfile                     0.1s
 => => transferring dockerfile: 295B                                     0.0s
 => [internal] load metadata for docker.io/library/nginx:latest          0.8s
 => [internal] load .dockerignore                                        0.1s
 => => transferring context: 2B                                          0.0s
 => [internal] load build context                                        0.1s
 => => transferring context: 318B                                        0.0s
 => [1/2] FROM docker.io/library/nginx:latest@sha256:5a88c9c454794...    4.0s
 ...   (레이어 다운로드 / extracting 로그 생략)
 => [2/2] COPY src/index.html /usr/share/nginx/html/index.html           0.4s
 => exporting to image                                                   0.2s
 => => exporting layers                                                  0.1s
 => => writing image sha256:f8532955e1df869f9af96de669342940851bb5f0...  0.0s
 => => naming to docker.io/library/my-nginx                              0.0s

[+] Building 5.9s (7/7) FINISHED
```

**성공 신호 체크**

| 로그 | 의미 |
|---|---|
| `[2/2] COPY src/index.html ...` | 앞서 실패했던 단계를 에러 없이 통과 ✅ |
| `exporting to image` | 이미지로 내보내기 완료 ✅ |
| `naming to docker.io/library/my-nginx` | 태그 `my-nginx` 부여 완료 ✅ |
| `Building 5.9s (7/7) FINISHED` | 7단계 전부 완료 = 완벽한 빌드 🎯 |

**이미지 생성 확인**

```console
% docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
my-nginx     latest    f8532955e1df   ...             ...
```

빌드 로그의 `writing image sha256:f8532955e1df...` 와 `IMAGE ID`가 일치 → 정상 생성 확인 ✅

---

### 4.4 실행 및 포트 매핑

```console
$ docker run -d -p 8080:80 my-nginx
```

**커스텀 페이지 접속 결과**

![포트 매핑 증거 — 컨테이너 실행 및 접속 확인](web/portMapping.png)

포트 매핑의 핵심: `-p 호스트포트:컨테이너포트` — `EXPOSE 80`은 문서화일 뿐이므로,
격리된 컨테이너의 80번 포트를 호스트의 8080번으로 **연결해야** 외부에서 접근할 수 있다.

> 📌 이 시점의 `docker ps` 출력은 원본 로그에 없지만, **8.2에서 같은 매핑이 실측으로 확인**된다.
> ```
> 0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   codyssey-web-1
> ```

---

## 5. 볼륨 — 데이터 영속성 검증

### 5.1 실습 목적

컨테이너가 **삭제되어도 데이터가 유지되는 영속성(Persistence)** 을 Docker 볼륨으로 검증한다.

### 5.2 실습 과정 및 실제 출력

#### ① 볼륨 생성

```console
$ docker volume create my-volume
my-volume
```

💬 `my-volume` 이라는 이름의 볼륨이 정상 생성됨.

#### ② 컨테이너 실행 및 데이터 저장

```console
$ docker run -it --name worker-container -v my-volume:/mnt/data alpine sh
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
55afa1ecc21d: Pull complete 
Digest: sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b
Status: Downloaded newer image for alpine:latest

/ # echo "data" > /mnt/data/hello.txt
/ # cat /mnt/data/hello.txt 
data
/ # exit
```

💬 컨테이너 내부 `/mnt/data` 에 `hello.txt` 를 생성하고 `"data"` 저장 확인.

#### ③ 컨테이너 삭제

```console
$ docker rm worker-container
worker-container
```

💬 데이터를 생성했던 컨테이너를 완전히 삭제.

#### ④ 영속성 최종 검증

```console
$ docker run --rm -v my-volume:/mnt/data alpine cat /mnt/data/hello.txt 
data
```

💬 컨테이너를 삭제했음에도 **새 컨테이너에서 `data` 출력 확인 → 영속성 검증 성공!**

### 5.3 결론

- 컨테이너를 삭제해도 **볼륨에 저장된 데이터는 호스트에 그대로 유지**된다.
- 동일한 볼륨을 다시 마운트하면 **어떤 컨테이너에서든 데이터를 복구**할 수 있다.

---

## 6. Git 설정 및 GitHub 연동

### 6.1 설정 확인 — `git config --list`

```console
rhw02133670@c4r1s8 codyssey % git config --list
credential.helper=osxkeychain
init.defaultbranch=main
user.name=Rhw0213
user.email=***@***.com
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
remote.origin.url=https://github.com/Rhw0213/codyssey.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
branch.main.merge=refs/heads/main
```

### 6.2 항목별 의미

| 항목 | 값 | 의미 |
|---|---|---|
| `credential.helper` | `osxkeychain` | macOS 키체인에 인증정보 저장 → 매번 로그인 불필요 |
| `init.defaultbranch` | `main` | 새 저장소의 기본 브랜치 |
| `user.name` / `user.email` | `Rhw0213` / … | 커밋에 기록되는 작성자 정보 |
| `core.filemode` | `true` | 파일 실행 권한 변경을 Git이 추적 (2장 `chmod` 실습과 연결) |
| `core.ignorecase` | `true` | macOS 파일시스템 특성상 대소문자 구분 안 함 |
| `remote.origin.url` | `.../Rhw0213/codyssey.git` | GitHub 원격 저장소 연결 완료 |
| `branch.main.remote` / `.merge` | `origin` / `refs/heads/main` | 로컬 `main` ↔ 원격 `origin/main` 추적 관계 설정 |

✅ `remote.origin.url` 과 `branch.main` 추적 설정이 모두 존재 → **GitHub 연동 정상 완료**

### 6.3 GitHub 연동 화면

![깃허브 연동 결과](web/githubScreen.png)


---

## 7. Docker Compose — 단일 서비스 전환

### 7.1 트러블슈팅 — Docker 데몬 미실행

```console
rhw02133670@c4r1s4 codyssey % docker-compose up -d
unable to get image 'codyssey-web': failed to connect to the docker API at unix:///var/run/docker.sock; check if the path is correct and if the daemon is running: dial unix /var/run/docker.sock: connect: no such file or directory
```

**원인:** OrbStack(Docker 런타임)을 실행하지 않은 상태였다.

3.1에서 확인한 4단계 구조를 떠올리면 이해가 쉽다. **Docker CLI는 요청을 전달할 뿐, 실제 작업은 데몬이 한다.** 데몬이 떠 있지 않으면 둘을 잇는 소켓 파일(`/var/run/docker.sock`) 자체가 존재하지 않는다. 에러 메시지의 `no such file or directory`가 정확히 그 뜻이다.

**해결:** OrbStack 실행 후 재시도 → 정상 빌드.

---

### 7.2 docker-compose.yml 작성

```yaml
services:
    web:
      build: ./web
      ports:
        - "8080:80"
```

파일들을 `web/` 폴더로 옮기고, 그 폴더를 빌드 대상으로 지정했다.

| 항목 | 의미 | 기존 CLI 대응 |
|---|---|---|
| `build: ./web` | `web/` 폴더의 Dockerfile로 이미지 빌드 | `docker build ./web` |
| `ports: "8080:80"` | 호스트 8080 → 컨테이너 80 | `-p 8080:80` |
| (파일 전체) | 실행 조건을 파일로 고정 | `docker run` 옵션들 |

**핵심 변화:** 기존에는 실행할 때마다 옵션을 손으로 쳐야 했다.

```console
$ docker build -t my-nginx .
$ docker run -d -p 8080:80 my-nginx
```

이제는 옵션이 파일에 적혀 있으므로 명령 하나로 끝난다.

```console
$ docker-compose up -d
```

**포트 번호, 빌드 경로 같은 실행 조건까지 Git으로 버전 관리되는 파일이 된 것**이다. 4장에서 Dockerfile이 "환경의 명세서"였다면, docker-compose.yml은 **"실행 방법의 명세서"** 다.

---

### 7.3 실행 결과

```console
rhw02133670@c4r1s4 codyssey % docker-compose up -d
[+] Building 9.8s (9/9) FINISHED
 => [internal] load build definition from Dockerfile                            0.2s
 => [internal] load metadata for docker.io/library/nginx:latest                 2.5s
 => [internal] load build context                                               0.6s
 => [1/2] FROM docker.io/library/nginx:latest@sha256:5a88c9c45479443d7be2ea...  4.5s
 ...   (레이어 다운로드 / extracting 로그 생략)
 => [2/2] COPY src/index.html /usr/share/nginx/html/index.html                  0.4s
 => exporting to image                                                          1.0s
 => => naming to docker.io/library/codyssey-web:latest                          0.0s
 => => unpacking to docker.io/library/codyssey-web:latest                       0.1s

[+] up 3/3
 ✔ Image codyssey-web       Built                                               9.9s
 ✔ Network codyssey_default Created                                             0.1s
 ✔ Container codyssey-web-1 Started                                             0.5s
```

출력에서 읽어야 할 세 줄

| 로그 | 의미 |
|---|---|
| `Image codyssey-web Built` | 이미지명이 **`폴더명-서비스명`** 규칙으로 자동 생성 (`-t` 안 줘도 됨) |
| `Network codyssey_default Created` | **전용 네트워크를 자동으로 만들어줌** — 8장 통신의 기반이 되는 부분 |
| `Container codyssey-web-1 Started` | 컨테이너명도 `프로젝트-서비스-번호`로 자동 부여 |

`docker run`으로는 네트워크를 직접 만들어 붙여야 했지만, Compose는 이걸 **기본으로 깔고 시작한다.**

---

## 8. 멀티 컨테이너와 컨테이너 간 통신

### 8.1 3개 서비스 구성

```yaml
version: "3.8"

services:
    web:
      image: nginx:latest
      ports:
        - "8080:80"
    cache:
      image: redis:latest

    app:
      image: alpine:latest
      command: sleep infinity
```

> ⚠️ **`version: "3.8"` 은 삭제 권장**
> 실행할 때마다 이 경고가 붙는다.
> ```
> WARN[0000] the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
> ```
> Compose V2부터 무시되는 속성이다. 7.2에서 쓴 파일처럼 `services:`로 바로 시작하면 경고가 사라진다.

> ⚠️ **`web`이 `build: ./web` → `image: nginx:latest` 로 바뀌었다**
> 7장에서 만든 커스텀 이미지가 아니라 **공식 원본**을 쓰게 된다.
> 8.3의 `curl` 결과가 커스텀 페이지가 아닌 이유가 바로 이것이다. (아래에서 다시 설명)

**`command: sleep infinity` 가 필요한 이유:** 컨테이너는 **주 프로세스가 끝나면 함께 종료된다.** `alpine`은 기본적으로 할 일이 없어 즉시 죽는다. 그래서 "무한 대기"를 시켜 살려두고, 그 안에 들어가 테스트할 발판으로 삼은 것이다.

---

### 8.2 실행 및 상태 확인

```console
rhw02133670@c4r1s4 codyssey % docker-compose up -d
[+] up 17/17
 ✔ Image alpine:latest        Pulled                                            4.6s
 ✔ Image redis:latest         Pulled                                            5.7s
 ✔ Image nginx:latest         Pulled                                            3.8s
 ✔ Container codyssey-app-1   Started                                           1.3s
 ✔ Container codyssey-cache-1 Started                                           1.3s
 ✔ Container codyssey-web-1   Started                                           1.3s
```

```console
rhw02133670@c4r1s4 codyssey % docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                     NAMES
725c0514b6b5   nginx:latest    "/docker-entrypoint.…"   12 seconds ago   Up 11 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   codyssey-web-1
e25246f9655b   redis:latest    "docker-entrypoint.s…"   12 seconds ago   Up 11 seconds   6379/tcp                                  codyssey-cache-1
d40e1afd0aa8   alpine:latest   "sleep infinity"         12 seconds ago   Up 11 seconds                                             codyssey-app-1
```

**`PORTS` 컬럼이 세 컨테이너 모두 다르다. 이게 4장 포트 매핑의 복습이다.**

| 컨테이너 | PORTS | 해석 |
|---|---|---|
| `web` | `0.0.0.0:8080->80/tcp` | **매핑됨** — 호스트 브라우저에서 접근 가능 |
| `cache` | `6379/tcp` | 컨테이너가 쓰는 포트일 뿐, **매핑 없음** → 호스트에서 접근 불가 |
| `app` | (비어 있음) | 노출 포트 자체가 없음 |

여기서 중요한 결론이 나온다. **매핑이 없어도 컨테이너끼리는 서로 통신할 수 있다.** 매핑은 "**바깥**에서 들어오는 길"을 뚫는 것이지, 내부 통신과는 별개다. Redis를 외부에 열지 않는 건 보안상 오히려 정상이다.

---

### 8.3 컨테이너 간 통신 테스트

#### 트러블슈팅 — 옵션 오타

```console
/ # apk add --nocache bind-tools curl
ERROR: command line: unrecognized option 'nocache'

/ # apk add --no-cache bind-tools curl
(1/23) Installing fstrm (0.6.1-r4)
...
(23/23) Installing curl (8.21.0-r0)
OK: 20.0 MiB in 39 packages
```

`--nocache` ❌ → `--no-cache` ✅ (하이픈 누락)

#### ① 이름으로 주소 찾기 — `nslookup`

```console
/ # nslookup cache
Server:		127.0.0.11
Address:	127.0.0.11#53

Non-authoritative answer:
Name:	cache
Address: 192.168.97.4
```

`Server: 127.0.0.11` — 이건 일반 DNS 서버가 아니라 **Docker가 컨테이너마다 넣어주는 내장 DNS**다. 여기에 `docker-compose.yml`의 **서비스 이름이 그대로 등록**된다.

#### ② 이름으로 접속 — `curl`

```console
/ # curl web
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
...
```

**IP를 몰라도 `web`, `cache` 라는 이름만으로 붙는다.** 이게 Compose가 자동 생성한 `codyssey_default` 네트워크 덕분이다.

이 점이 실무에서 결정적인 이유는, **컨테이너 IP는 재시작할 때마다 바뀌기 때문**이다. `192.168.97.4`를 코드에 적어두면 다음 `up`에서 깨진다. 반면 서비스 이름은 고정이므로, 애플리케이션 설정에 `redis://cache:6379` 처럼 **이름으로 적어두면 계속 동작한다.**

> 💡 **확인 포인트:** `curl web` 결과가 4장에서 만든 커스텀 페이지("🚀 Codyssey Docker 미션 성공!")가 아니라 **nginx 기본 페이지**다.
> 8.1에서 `web`을 `image: nginx:latest`로 바꿨기 때문에 커스텀 이미지가 쓰이지 않은 것.
> 커스텀 페이지로 통신 테스트를 하려면 `build: ./web` 으로 되돌리면 된다. 통신 검증 자체에는 문제없다.

---

### 8.4 Compose 운영 명령

#### 상태 확인 — `docker-compose ps`

```console
rhw02133670@c4r1s4 codyssey % docker-compose ps
NAME               IMAGE           COMMAND                  SERVICE   CREATED         STATUS         PORTS
codyssey-app-1     alpine:latest   "sleep infinity"         app       8 minutes ago   Up 8 minutes   
codyssey-cache-1   redis:latest    "docker-entrypoint.s…"   cache     8 minutes ago   Up 8 minutes   6379/tcp
codyssey-web-1     nginx:latest    "/docker-entrypoint.…"   web       8 minutes ago   Up 8 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```

`docker ps`와 달리 **`SERVICE` 컬럼이 있고, 이 프로젝트의 컨테이너만** 보여준다.

#### 로그 확인 — `docker-compose logs --tail 20`

```console
cache-1  | 1:M 29 Jul 2026 01:50:22.578 * Server initialized
cache-1  | 1:M 29 Jul 2026 01:50:22.578 * Ready to accept connections tcp
cache-1  | 1:M 29 Jul 2026 01:50:22.578 # WARNING: Redis does not require authentication and is not protected by network restrictions.
web-1    | /docker-entrypoint.sh: Configuration complete; ready for start up
web-1    | 2026/07/29 01:50:22 [notice] 1#1: nginx/1.31.3
web-1    | 2026/07/29 01:50:22 [notice] 1#1: OS: Linux 6.19.13-orbstack-gbd1dc07b8cf4
web-1    | 2026/07/29 01:50:22 [notice] 1#1: start worker processes
web-1    | 192.168.97.3 - - [29/Jul/2026:01:56:27 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/8.21.0" "-"
```

**여러 컨테이너의 로그가 `서비스명 |` 접두사와 함께 한 화면에 합쳐진다.** 컨테이너를 하나씩 열어볼 필요가 없다.

가장 중요한 건 마지막 줄이다.

```
192.168.97.3 - - [29/Jul/2026:01:56:27 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/8.21.0" "-"
```

8.3에서 `app` 컨테이너가 보낸 `curl web` 요청이 **받는 쪽 로그에 그대로 찍혔다.** `200`(성공), `curl/8.21.0`(보낸 도구)까지 일치한다. **통신이 됐다는 양쪽 증거가 맞춰진 셈**이라, 통신 검증 자료로는 이 줄이 가장 강력하다.

한편 Redis 경고도 눈여겨볼 만하다. `authentication ... not protected` — 인증 없이 떠 있다는 뜻이다. 지금은 외부 포트를 열지 않아 괜찮지만, `ports`로 노출한다면 반드시 비밀번호 설정이 필요하다.

#### 정리 — `docker-compose down`

```console
rhw02133670@c4r1s4 codyssey % docker-compose down
[+] down 4/4
 ✔ Container codyssey-cache-1 Removed                                           0.4s
 ✔ Container codyssey-app-1   Removed                                          10.3s
 ✔ Container codyssey-web-1   Removed                                           0.4s
 ✔ Network codyssey_default   Removed                                           0.1s

rhw02133670@c4r1s4 codyssey % docker compose ps
NAME      IMAGE     COMMAND   SERVICE   CREATED   STATUS    PORTS
```

컨테이너 3개 + 네트워크까지 한 번에 제거되고, `ps` 결과가 비었다.

여기서 5장과 이어지는 중요한 점 — **`down`은 볼륨을 지우지 않는다.** 컨테이너와 네트워크는 사라져도 데이터는 남는다. 이것이 "컨테이너는 버릴 수 있고 상태는 밖에 둔다"는 원칙이 실제로 동작하는 모습이다. (볼륨까지 지우려면 `down -v`)

> `app-1` 제거에만 10.3초가 걸린 건 `sleep infinity`가 종료 신호에 바로 반응하지 않아, Docker가 강제 종료까지 기다린 시간이다.

---

### 8.5 `docker-compose` vs `docker compose`

로그를 보면 두 표기를 섞어 쓰셨다.

| 표기 | 정체 |
|---|---|
| `docker-compose` (하이픈) | 예전 V1, 별도 파이썬 도구 |
| `docker compose` (공백) | 현재 V2, Docker CLI에 내장된 플러그인 |

지금 환경은 하이픈으로 쳐도 V2로 연결되어 결과가 같지만(경고 메시지가 동일한 것으로 확인 가능), **공백 표기가 현재 표준**이다. 새로 쓰는 문서나 스크립트에는 `docker compose`로 통일하는 편이 좋다.

---

## 9. 환경변수로 설정 분리

### 9.1 구성

```yaml
services:
    app:
      image: alpine:latest
      command: sh -c "echo 포트는 $$PORT, 모드는 $$MODE 입니다 && sleep infinity"
      environment:
        - PORT=5000
        - MODE=development
```

**`$$` 로 쓴 이유** — Compose는 `$VAR`를 **자기가 먼저 치환**하려 든다. 여기서 원하는 건 컨테이너 **안의 셸**이 값을 읽는 것이므로, `$$`로 이스케이프해서 `$`를 그대로 넘긴 것이다.

| 표기 | 누가 해석 | 결과 |
|---|---|---|
| `$PORT` | Compose (호스트) | 호스트에 그 변수가 없으면 빈 값 |
| `$$PORT` | 컨테이너 안의 셸 | `environment`의 `5000` ✅ |

### 9.2 실행 결과

```console
rhw02133670@c4r1s4 codyssey % docker-compose logs app
app-1  | 포트는 5000, 모드는 development 입니다
```

✅ 컨테이너 안에서 환경변수가 정상적으로 읽혔다.

### 9.3 왜 중요한가

**이미지는 그대로 두고 설정만 바꿔 끼울 수 있기 때문**이다.

```
같은 alpine 이미지  +  MODE=development  →  개발 환경
같은 alpine 이미지  +  MODE=production   →  운영 환경
```

설정값을 코드나 이미지 안에 박아두면, 환경이 바뀔 때마다 **이미지를 다시 빌드**해야 한다. 밖으로 빼두면 **테스트한 그 이미지를 그대로 운영에 올릴 수 있다.** 미션 목표인 "재현 가능한 환경"이 여기서 완성된다. 빌드는 한 번, 실행은 여러 환경에서.

DB 비밀번호나 API 키를 이미지에 넣지 않는 이유도 같은 맥락이다. 이미지는 공유되지만 환경변수는 실행 시점에 주입된다.

---

## 10. GitHub SSH 키 설정

### 10.1 키 생성

```console
rhw02133670@c4r1s4 codyssey % ssh-keygen -t ed25519 -C "***@***.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/rhw02133670/.ssh/id_ed25519): 
Enter passphrase for "/Users/rhw02133670/.ssh/id_ed25519" (empty for no passphrase): 
Your identification has been saved in /Users/rhw02133670/.ssh/id_ed25519
Your public key has been saved in /Users/rhw02133670/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:******************************** ***@***.com
```

| 옵션 | 의미 |
|---|---|
| `-t ed25519` | 키 알고리즘. RSA보다 짧고 빠르며 안전해 현재 권장 방식 |
| `-C "..."` | 주석(comment). 키 식별용 라벨일 뿐, 인증에는 쓰이지 않음 |

### 10.2 생성 결과와 파일 권한

```console
rhw02133670@c4r1s4 codyssey % ls -al ~/.ssh
total 24
drwxr-xr-x   5 rhw02133670  rhw02133670  160 Jul 29 11:13 .
drwxr-x---+ 21 rhw02133670  rhw02133670  672 Jul 29 11:06 ..
-rw-r--r--   1 rhw02133670  rhw02133670  210 Jul 29 10:33 config
-rw-------   1 rhw02133670  rhw02133670  411 Jul 29 11:13 id_ed25519
-rw-r--r--   1 rhw02133670  rhw02133670   99 Jul 29 11:13 id_ed25519.pub
```

**2장에서 배운 권한이 여기서 실제로 의미를 갖는다.**

| 파일 | 권한 | 숫자 | 이유 |
|---|---|---|---|
| `id_ed25519` (**개인키**) | `-rw-------` | `600` | **나만 읽을 수 있어야 한다.** 유출되면 곧 내 신원 |
| `id_ed25519.pub` (**공개키**) | `-rw-r--r--` | `644` | 남에게 **주라고 만든 것**이므로 읽기 허용 |

`ssh-keygen`이 개인키를 `600`으로 만드는 건 우연이 아니다. **SSH는 개인키 권한이 느슨하면 아예 사용을 거부한다.** 권한이 실제 보안 장치로 작동하는 사례다.

### 10.3 공개키 확인

```console
rhw02133670@c4r1s4 codyssey % cat ~/.ssh/id_ed25519.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA******************************** ***@***.com
```

이 **공개키만** GitHub 설정에 등록한다. 개인키는 절대 밖으로 내보내지 않는다.

> 공개키는 이름 그대로 공개돼도 되는 값이라 문서에 남겨도 무방하다. 다만 **`id_ed25519`(`.pub` 없는 쪽)의 내용은 어떤 문서에도 붙여넣지 않는다.**

### 10.4 인증 확인

```console
rhw02133670@c4r1s4 codyssey % ssh -T git@github.com
The authenticity of host 'github.com (20.200.245.247)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.

Hi Rhw0213! You've successfully authenticated, but GitHub does not provide shell access.
```

✅ **`Hi Rhw0213!`** — 내 계정으로 인식됐다는 뜻. 등록 성공.

뒤의 `does not provide shell access`는 에러가 아니다. GitHub은 SSH를 **Git 통신 용도로만** 열어두고 서버 접속은 허용하지 않는다는 정상 안내다.

첫 접속 시 물어본 질문은 **"이 서버가 진짜 GitHub이 맞는지"** 확인하는 절차다. 승인하면 `~/.ssh/known_hosts`에 지문이 저장되어 다음부터는 묻지 않고, 만약 지문이 달라지면 경고를 띄운다.

### 10.5 HTTPS vs SSH

| | HTTPS | SSH |
|---|---|---|
| 주소 | `https://github.com/Rhw0213/codyssey.git` | `git@github.com:Rhw0213/codyssey.git` |
| 인증 | 토큰 (키체인 저장) | 키 쌍 |
| 준비 | 없음 | 키 생성 + 등록 |

> ⚠️ **아직 남은 작업이 있다.**
> 6.1의 `git config` 출력을 보면 현재 원격 주소는 여전히 **HTTPS**다.
> ```
> remote.origin.url=https://github.com/Rhw0213/codyssey.git
> ```
> SSH 키를 등록했어도 이 상태에서는 `push`가 계속 HTTPS 경로로 나간다. 실제로 SSH를 쓰려면 원격 주소를 바꿔야 한다.
> ```console
> $ git remote set-url origin git@github.com:Rhw0213/codyssey.git
> $ git remote -v          # 변경 확인
> ```
> 즉, **10.4까지는 "키 등록 성공"이고, 저장소가 그 키를 쓰도록 연결하는 건 별개 단계다.**

---

## 11. 최종 정리 — 이 미션에서 얻은 원칙

| 원칙 | 근거가 된 실습 결과 |
|---|---|
| **이미지와 컨테이너는 분리된다** | 같은 `ubuntu` 이미지로 컨테이너를 만들고 지워도 이미지는 `docker images`에 그대로 남았다 |
| **컨테이너는 격리된 실행 환경이다** | `docker run -it` 진입 시 호스트와 무관한 독립 파일시스템이 보였다 |
| **경로는 빌드 컨텍스트 기준이다** | `COPY index.html` 실패 → `COPY src/index.html` 성공으로 직접 확인 |
| **연결은 명시해야 한다 (포트)** | `EXPOSE`는 문서화일 뿐, 실제 접속은 `-p` 매핑이 있어야 가능 |
| **매핑과 내부 통신은 별개다** | `cache`는 포트 매핑이 없어 호스트에서 못 붙지만, `app`에서는 `nslookup cache`로 찾아졌다 |
| **상태는 컨테이너 밖에 둬야 한다 (볼륨)** | 컨테이너를 `rm`한 뒤에도 새 컨테이너에서 `data`를 다시 읽어냈다 |
| **주소가 아니라 이름으로 연결한다** | 재시작마다 바뀌는 IP 대신 서비스명 `web`, `cache`로 통신 성공 |
| **실행 조건도 파일로 고정한다** | 포트·빌드 경로를 `docker-compose.yml`에 적어 `up -d` 하나로 재현 |
| **설정은 이미지 밖으로 뺀다** | 같은 `alpine` 이미지에 환경변수만 주입해 동작을 바꿈 |
| **권한은 실제 보안 장치다** | 개인키 `600` / 공개키 `644` — SSH는 권한이 느슨하면 키 사용을 거부한다 |
| **환경 설정 자체가 버전 관리 대상이다** | Dockerfile · docker-compose.yml을 Git에 함께 두면 누구나 같은 환경을 재현 |


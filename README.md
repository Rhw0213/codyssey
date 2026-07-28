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
(컨테이너 ID 출력 — 채워넣기)

$ docker ps
CONTAINER ID   IMAGE      COMMAND   CREATED   STATUS   PORTS                  NAMES
(0.0.0.0:8080->80/tcp 확인 — 채워넣기)
```

**커스텀 페이지 접속 결과**

(여기에 스크린샷 URL 붙여넣기)

포트 매핑의 핵심: `-p 호스트포트:컨테이너포트` — `EXPOSE 80`은 문서화일 뿐이므로,
격리된 컨테이너의 80번 포트를 호스트의 8080번으로 **연결해야** 외부에서 접근할 수 있다.

> ⚠️ 위 콘솔 블록과 스크린샷은 원본 로그에서 비어 있던 구간입니다.

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
user.email=rhw0213@gmail.com
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

> ⚠️ **미기록 항목** — `git push` / `git status` 등 실제 연동 동작 출력이 원본에 없습니다.
> `git push -u origin main` 결과 로그를 추가하면 연동 검증이 완성됩니다.

---

## 7. 최종 정리 — 이 미션에서 얻은 원칙

| 원칙 | 근거가 된 실습 결과 |
|---|---|
| **이미지와 컨테이너는 분리된다** | 같은 `ubuntu` 이미지로 여러 컨테이너를 만들고 지워도 이미지는 `docker images` 에 그대로 남아 있었다 |
| **컨테이너는 격리된 실행 환경이다** | `docker run -it` 진입 시 호스트와 무관한 독립 파일시스템(`/bin`, `/etc` …)이 보였다 |
| **경로는 빌드 컨텍스트 기준이다** | `COPY index.html` 실패 → `COPY src/index.html` 성공으로 직접 확인 |
| **연결은 명시해야 한다 (포트)** | `EXPOSE` 는 문서화일 뿐, 실제 접속은 `-p` 매핑이 있어야 가능 |
| **상태는 컨테이너 밖에 둬야 한다 (볼륨)** | 컨테이너를 `rm` 한 뒤에도 새 컨테이너에서 `data` 를 다시 읽어냈다 |
| **환경 설정 자체가 버전 관리 대상이다** | Dockerfile을 Git 저장소에 함께 두면 누구나 같은 환경을 재현할 수 있다 |
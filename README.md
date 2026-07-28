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

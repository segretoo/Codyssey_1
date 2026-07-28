# 1번 미션 — 터미널/권한/Docker/Git 실습

## 0) 프로젝트 개요 (미션 목표 요약)
- 이 저장소는 터미널 기본 조작, 리눅스 파일 권한 실습, Docker 설치/운영, 커스텀 Dockerfile 제작,
  포트 매핑, 볼륨 영속성 검증, Git/GitHub 제출까지의 전 과정을 수행하고 기록한 결과물이다.
- (여기에 본인 과제 목표 1~2문장으로 요약해서 작성)

## 1) 실행 환경
- OS: (예: Windows 11 + WSL2 Ubuntu 22.04 / macOS 14 / Ubuntu 22.04)
- Shell: (예: bash / zsh)
- Docker: `docker --version` 결과 붙여넣기
- Git: `git --version` 결과 붙여넣기

```bash
$ docker --version
(출력 결과)

$ git --version
(출력 결과)
```

## 2) 수행 항목 체크리스트
- [ ] 터미널 기본 조작 (이동/목록/생성/복사/이동·이름변경/삭제/내용확인/빈파일생성)
- [ ] 권한 변경 실습 (파일 1개 + 디렉토리 1개, 변경 전/후 비교)
- [ ] Docker 설치/점검 (`docker --version`, `docker info`)
- [ ] Docker 기본 운영 명령 (images/ps/logs/stats)
- [ ] hello-world 실행
- [ ] ubuntu 컨테이너 진입 및 명령 수행 (ls, echo)
- [ ] 컨테이너 종료/유지(attach vs exec) 관찰 정리
- [ ] 커스텀 Dockerfile 작성 및 이미지 빌드/실행
- [ ] 포트 매핑 및 접속 증거 (2회 이상)
- [ ] 바인드 마운트 반영 확인
- [ ] Docker 볼륨 생성/연결/영속성 검증
- [ ] Git 설정 및 GitHub 업로드

## 3) 터미널 조작 로그

```bash
$ pwd
(출력)

$ ls -la
(출력)

$ mkdir -p ~/codyssey/mission1
$ cd ~/codyssey/mission1

$ touch memo.txt
$ cp memo.txt memo_copy.txt
$ mv memo_copy.txt memo_renamed.txt
$ cat memo.txt
(출력)

$ touch empty.txt
$ rm memo_renamed.txt
$ ls -la
(출력)
```

## 4) 권한 실습 (변경 전/후 비교)

### 파일 권한
```bash
$ touch perm_test.txt
$ ls -l perm_test.txt
(변경 전 출력)

$ chmod 600 perm_test.txt
$ ls -l perm_test.txt
(변경 후 출력)
```

### 디렉토리 권한
```bash
$ mkdir perm_dir
$ ls -ld perm_dir
(변경 전 출력)

$ chmod 700 perm_dir
$ ls -ld perm_dir
(변경 후 출력)
```

## 5) Docker 설치 및 기본 점검

```bash
$ docker --version
(출력)

$ docker info
(출력, 필요시 일부 생략 가능하나 정상 동작 여부가 드러나야 함)
```

## 6) Docker 기본 운영 명령

```bash
$ docker images
(출력)

$ docker pull nginx:alpine
(출력)

$ docker ps
(출력)

$ docker ps -a
(출력)

$ docker logs <container_name_or_id>
(출력)

$ docker stats --no-stream
(출력)
```

## 7) 컨테이너 실행 실습

```bash
$ docker run hello-world
(출력 — "Hello from Docker!" 확인)

$ docker run -it --name ubuntu-test ubuntu bash
root@...:/# ls
(출력)
root@...:/# echo "hello"
hello
root@...:/# exit
```

### attach vs exec 관찰
- `docker start -ai <container>` : 컨테이너의 메인 프로세스(PID 1)에 다시 연결(attach)한다.
  종료 시 컨테이너도 같이 정지되는 경우가 있다.
- `docker exec -it <container> bash` : 이미 실행 중인 컨테이너에 새로운 프로세스(bash)를
  추가로 실행해 접속한다. 이 셸을 종료해도 컨테이너 자체(메인 프로세스)는 계속 실행된다.
- (본인이 직접 관찰한 내용으로 다시 작성 권장)

## 8) 커스텀 Dockerfile 기반 이미지 제작

- 선택한 베이스: (예: nginx:alpine / ubuntu:22.04 등 — 본인이 선택한 것 기재)
- 선택 이유: (간단히)

```dockerfile
# Dockerfile
FROM nginx:alpine
LABEL org.opencontainers.image.title="my-custom-nginx"
ENV APP_ENV=dev
COPY site/ /usr/share/nginx/html/
```

### 적용한 커스텀 포인트와 목적
| 커스텀 포인트 | 목적 |
|---|---|
| LABEL 추가 | 이미지 메타데이터로 식별 용이 |
| ENV APP_ENV=dev | 환경 구분용 변수 설정 |
| COPY site/ | 정적 콘텐츠 교체 |

### 빌드/실행 명령 및 결과
```bash
$ docker build -t my-web:1.0 .
(출력 — Successfully built / tagged 확인)

$ docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
(출력)

$ curl http://localhost:8080
(출력 — 정상 응답 확인)
```

## 9) 포트 매핑 및 접속 증거

- 접속 URL: http://localhost:8080
- 증거 (스크린샷/캡처 파일 링크): `./evidence/port-mapping-1.png`
- 두 번째 접속 (다른 포트로 재실행 등): `./evidence/port-mapping-2.png`

```bash
$ docker run -d -p 8081:80 --name my-web-8081 my-web:1.0
$ curl http://localhost:8081
(출력)
```

## 10) 바인드 마운트 반영 확인

```bash
$ mkdir -p ~/codyssey/bindtest
$ echo "<h1>bind mount test</h1>" > ~/codyssey/bindtest/index.html
$ docker run -d -p 8082:80 -v ~/codyssey/bindtest:/usr/share/nginx/html --name bind-test my-web:1.0
$ curl http://localhost:8082
(출력 — bind mount test 확인)
```

## 11) Docker 볼륨 영속성 검증

```bash
$ docker volume create mydata
(출력)

$ docker run -d --name vol-test -v mydata:/data ubuntu sleep infinity
$ docker exec -it vol-test bash -lc "echo hi > /data/hello.txt && cat /data/hello.txt"
hi

$ docker rm -f vol-test
(컨테이너 삭제)

$ docker run -d --name vol-test2 -v mydata:/data ubuntu sleep infinity
$ docker exec -it vol-test2 bash -lc "cat /data/hello.txt"
hi
```
→ 컨테이너를 삭제한 뒤 새 컨테이너에서도 동일한 데이터(`hi`)가 확인되어
볼륨을 통한 데이터 영속성이 검증되었다.

## 12) Git / GitHub 제출

```bash
$ git init
$ git add .
$ git commit -m "1번 미션 제출: 터미널/권한/Docker 실습"
$ git branch -M main
$ git remote add origin <본인 저장소 URL>
$ git push -u origin main
```

- GitHub Repository 링크: (여기에 링크)

## 13) 검증 방법 요약

| 항목 | 검증 명령 | 결과 위치 |
|---|---|---|
| Docker 설치 | `docker --version`, `docker info` | 5번 섹션 |
| 이미지/컨테이너 | `docker images`, `docker ps -a` | 6번 섹션 |
| hello-world | `docker run hello-world` | 7번 섹션 |
| 커스텀 이미지 | `docker build`, `docker run` | 8번 섹션 |
| 포트 매핑 | `curl http://localhost:8080` | 9번 섹션 |
| 바인드 마운트 | `curl http://localhost:8082` | 10번 섹션 |
| 볼륨 영속성 | 삭제 전/후 `cat` 비교 | 11번 섹션 |

## 14) 주의사항 / 재현성 노트
- 본인 PC 환경에 종속된 경로(예: 홈 디렉토리 절대경로)가 있다면 여기에 대체 방법을 기재한다.
- 민감정보(비밀번호, 토큰, 개인정보 등)는 캡처/로그에서 마스킹 처리했다.

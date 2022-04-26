# 도커

## 이미지 & 컨테이너

- 이미지는 설정파일을 가지고 있고 이미지로 컨테이너를 생성함
  - 1️⃣ 시작시 실행될 명령어
  - 2️⃣ 파일 스냅샵

## 동작 방식

1. docker<도커 클라이언트 언급> run<도커 명령어> kakaotalk<이미지 이름>
2. 이미지안에 있는 스냅샷 파일이 컨테이너 하드디스크에 들어감
3. 명령어도 컨테이너로 들어감

## 2. 도커 클라이언트 명령어

### 작동 순서

- docker run alpine ls

1.  도커 클라이언트에 명령어 입력 하면 도커 서버로 전달됨
2.  입력된 이미지가 캐시에 존재하는지 확인
3.  없으면 허브에서 가져와서 컨테이너 생성

<도커 클라이언트 언급> <컨테이너 생성 및 실행> <이 컨테이너를 위한 이미지> <그 이미지가 가지고 있는 고유 명령어>

### 실행중인 컨테이너 나열

1.  docker ps(process status)
2.  docker run alpine ping localhost

### life cycle

1.  생성
    - docker create <이미지이름>
2.  시작
    - docker start <컨테이너 아이디/이름>

docker run = docker < create + start >

3.  실행

4.  중지

    - docker ps => process status 확인
    - docker stop < process id >
    - docker kill < process id >

5.  삭제
    - 중지시키고 삭제
    - docker rm < id >
    - docker rm \`docker ps -a -q\`
    - docker system prune (한번에 컨테이너, 이미지, 네트워크 모두 삭제)

### 실행중인 컨테이너에 명령어 전달

- docker exec <컨테이너 아이디> <명령어>

### redis 로 실습

- docker run redis

  - redis 컨테이너 생성

- docker ps -a

  - redis id 조회

- docker exec -it < redis id > redis-cli
  - -it === -i + -t
  - 내부에 계속 실행하도록 놔둬줌

### 컨테이너 내부에서 shell 사용

- docker exec -it < id > redis-cli, 이런 것처럼
- 계속 치기 번거러우니까 컨테이너 내부로 들어가서 shell 사용하는 방법

- docker exec -it < id > sh
- 나가는 방법은 ctl + D

## 3. 도커 이미지 직접 생성

### 이미지 생성 순서

1. dockerfile 생성
   - docker image를 만들기 위한 설정 파일
2. 도커 클라이언트로 dockerfile 전달
3. 도커 서버
   - 도커 클라이언트에 전달된 모든 중요한 작업들을 수행하는 곳
4. 이미지 생성

### 도커파일 만드는 순서

1. 베이스 이미지 명시 (파일 스냅샷에 해당)
2. 추가적으로 필요한 파일을 다운받기 위한 명령어를 명시(파일 스냅샷에 해당)
3. 컨테이너 시작시 실행 될 명령어를 명시(시작시 실행 될 명령어에 해당)

### docker file 실습

1. 도커 파일을 만들 폴더 생성 (dockerfile-folder)
2. 폴더 내부에 파일 생성 (dockerfile)
3. 기본토대를 명시

- FROM baseImage
- RUN command
- CMD [ "executable" ]

### docker file 을 도커 클라이언트 => 서버 => 생성

1. docker build ./
2. docker build -t <내 도커 아이디> / <저장소, 프로젝트 이름>:<버전>

- docker build -t siwonblue/nodejs:latest

## 4. node.js 앱 만들기

1. node.js 이미지 만들기
2. container 에서 실행시키기

### node.js 이미지 만들기

1. package.json 생성
2. server.js

### Dockerfile image 생성

1. FROM < package name >:< version >
2. RUN < npm install>
3. CMD ["node","server.js"]

### docker copy

1. FROM < package name >:< version >
2. COPY ./ ./
3. RUN < npm install>
4. CMD ["node","server.js"]

- copy 를 이용해서 ./ 에 있는 모든 파일을 컨테이너 안으로 copy

### docker run -p <브라우저 포트>:<컨테이너 내부 포트> <도커 이미지>

> 브라우저 포트와 컨테이너 내부 포트를 연결시켜 주기 위해 사용하는 명령어

ex . docker run -p 5000:3060 siwonblue/nodejs:latest

- http://localhost:5000 으로 접속하면 컨테이너로 mapping

### docker working directory 사용

> COPY 를 하면서 모든 파일이 root directory에 들어오게 되는 것을 방지

### 소스코드만 변경되었을 때

소스코드만 변경이 되었는데 패키지를 다시 다운받지 않도록 하기 위해
package.json 을 먼저 COPY 시켜줌

```docker
COPY package.json ./

RUN npm install

COPY ./ ./

```

### Volume 사용

- copy 는 컨테이너 내부에 복사해서 넣는 것
- volume 은 컨테이너에서 호스트 환경을 참조하기 위해 mapping

docker run -d -p 5000:8080 -v /usr/src/app/node_modules -v $(pwd):/usr/src/app siwonblue/nodejs
<도커 클라이언트 언급> <도커 실행> < 실행 후 컨테이거 나오기 > <포트 설정> <로컬에서 5000을 컨테이너 8008와 매핑> <호스트에 node_modules 없으니까 제외 시킴> <':'기준으로 왼쪽이 호스트환경, 오른쪽이 컨테이너>

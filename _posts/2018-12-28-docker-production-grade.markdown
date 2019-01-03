---
layout: post
title:  "Docker로 Production까지"
date:   2018-12-29 10:00:00 +09:00
categories: "docker"
published: false
---

## Production Grade Flow
* development -> testing -> deployment 의 과정

* Github Repo
  * Feature branch: 개발 서버
  * Master branch: 운용 서버

Feature branch로 Push를 하고, Pull Request를 통해 master로 merge가 된다.
master는 Travis CI로 연결된다.
결과가 통과면 AWS에 호스팅된다.

## 정리
1. Dev Phase
- 기능 추가/수정
- master가 아닌 branch에 변경 사항 기입

2. Github로 Push
3. master로 merge하기 위해 Pull Request 생성

4. Test Phase
- Travis CI로 코드 Push
- Test Run

5. Pull Request를 master와 merge

6. Prod Phase
- Travis CI로 코드 Push
- Test Run
- AWS에 배포 (Elastic Beanstalk)

## React Project 생성
`create-react-app frontend`

`Dockerfile.dev` 파일 생성
```Dockerfile
FROM node:alpine

WORKDIR '/app'

COPY package.json ./
RUN npm install

COPY ./ ./

CMD ["npm", "run", "start"]
```
* custom file사용하기: `docker build -f Dockerfile.dev .`

* `/node_modules` 폴더는 제거한 후에 `.Dockerignore` 파일에 추가하자

## 파일 변경사항 감지하기
Docker의 Volume을 사용하자
`docker run -v $(pwd):/app (image id)`

실행하면 아래와 같은 에러를 뿜으면서 실행이 안된다.
```
> frontend@0.1.0 start /app
> react-scripts start

sh: react-scripts: not found
```
볼륨을 전체로 놔뒀기 때문에 dependency가 전부 설치되어 있는 node_modules 폴더도 연결되어 있기 때문이다.
아까 node_modules폴더를 지웠기 때문에, container가 호스트에 있는 node_modules 폴더를 찾으려고 하면 데이터가 없기 때문에 오류가 난다.

이럴 경우, 매핑이 아닌, 별도의 북마킹을 이용하여 해당 폴더만 제외시키면 된다.
`docker run -v /app/node_modules -v $(pwd):/app (image id)`

위와 다른 점은, 콜론이 없고 경로도 하나만 있다는건데, 해당 경로는 container에서 사용해라 정도로 이해하면 될 것 같다.

이렇게 매핑이 끝나면, 로컬 코드 변화에 따라서 바로 변화가 반영된다.

## Docker Compose로 더 쉽게 하기
```yaml
version: '3'
services:
  web:
    build:
      context: ./
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - ./:/app
```
* context: 데이터가 위치한 곳. 만약에 다른 폴더에 있으면 `./another-folder`와 같이 지정해주면 된다.
* dockerfile: 별도의 dockerfile 지정

## Testing 진행하기
우선은 마지막에 있는 CMD를 오버라이딩 해주자
`docker run (image id) (CMD)`

여기서는 `docker run 4e28180ab9a2 npm run test`
하지만 확인해보면 테스트 중에 화면과 인터랙션이 안된다.

STDIN과 연결해야하기 때문에 -it 를 붙혀주도록 하자
`docker run -it 4e28180ab9a2 npm run test`

## Testing 문제 해결
컨테이너 생성시 전체에 대한 스냅샷을 떠서 컨테이너에 넣었기 때문에, 테스트 코드의 변경 사항이 반영되지 않는다.

1. 해결책: docker-compose volumes 변경
2. 기존의 container에 attach mode

* Attach mode
`docker-compose up`으로 서버를 띄운다

`docker exec -it (container id) npm run test`

이렇게 하면 변경사항이 정상적으로 반영된다.

* docker compose로 작업하기
```yaml
version: '3'
services:
  web:
    build:
      context: ./
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - ./:/app
  test-web:
    build:
      context: ./
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - ./:/app
    command: ["npm", "run", "test"]
```

이 방법대로 하면 다 좋지만, 기존에 사용하던 터미널 인터렉션이 사라졌다는 것을 알 수 있다.

docker-compose로 띄웠을 때 npm process가 start.js를 시작하는 형식으로 되어있다. Interactivity를 위해서는 해당 start.js파일의 STDIN을 연결해야하는데, 그렇게 할 수 없기 때문에 별도로 인터렉션을 발생시킬 수 없다. (결국 둘 다 장단점이 존재)

* 왜 STDIN 연결이 안될까?
`docker-compose up` 으로 서버를 띄운 뒤, `docker attach (해당 컨테이너 id)`를 통해 들어가보자. 아무런 화면도 안뜬다. 이유는 npm script가 실행되는 방식에 있다.
`docker exec -it (해당 컨테이너 id) sh`로 쉘에 들어간 다음 `ps`로 프로세스 목록을 확인해보자.

```shell
    1 root      0:00 npm
   19 root      0:00 node /app/node_modules/.bin/react-scripts test
   26 root      0:05 node /app/node_modules/react-scripts/scripts/test.js
```
보시다시피, npm과 node 스크립트들이 분리되어있는 것을 알 수 있다.
attach로 들어가면 현재 실행 중인 루트 프로세스 (여기서는 npm)를 기반으로 들어가게 되는데,
그럴 경우 npm으로 들어가지기 때문에 별도의 프로세스에 대해서는 attach가 불가하다.

결국 위에 언급한 방식으로 하고 싶으면 별도의 docker container를 만드는게 가장 낫다.

## Production 환경 구축
Dev Server는 프로덕션 환경에서 사용할 수 없기 때문에, 별도의 Web Server를 필요로 한다.
여기서는 NGINX를 이용하여 해당 환경을 구축해보도록 하자

### Multi-step build process 구축
```Dockerfile
FROM node:alpine as builder

WORKDIR /app
COPY ./package.json /app

RUN npm install
COPY ./ ./
RUN npm run build

FROM nginx
COPY --from=builder /app/build /usr/share/nginx/html
```
NGINX와 NodeJS는 별도의 패키지에 존재하고, 정말 필요한 데이터만 남기면 되기 때문에, multi-step build process 과정을 거쳐야한다. 

`npm run build`의 결과물 중에서는 /build 폴더에 있는 항목만 가지고 있으면 된다.
그렇기 때문에 하나의 Dockerfile 내에서 여러개의 이미지를 가져다가 사용할 수 있다.

각각의 이미지는 `FROM * as *` 로 나타내면 되고, 이렇게 될 경우 위에 나온 COPY와 같은 항목을 작성할 때 --from을 사용하여 해당 이미지로 제작된 파일에서 파일을 가져다 사용할 수 있다.

이미지 생성이 완료되면 아래와 같이 실행해 준다.
`docker build -t immigra1tion9/front-prod .` -> `docker run -p 8080:80 immigration9/front-prod`

완료된 후 `http://localhost:8080` 에 접속하여 결과를 확인한다.
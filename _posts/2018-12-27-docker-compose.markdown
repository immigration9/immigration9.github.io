---
layout: post
title:  "Docker Compose로 Node App과 Redis 띄우기"
date:   2018-12-27 22:00:00 +09:00
categories: "docker"
published: true
---


## 구성
Node App -> Redis Server

```javascript
// index.js
const express = require('express');
const redis = require('redis');

const app = express();
const client = redis.createClient();
client.set('visits', 0);

app.get('/', (req, res) => {
  client.get('visits', (err, visits) => {
    res.send('Number of visits is ' + visits);
    client.set('visits', parseInt(visits) + 1);
  });
});

app.listen(8081, () => {
  console.log('Listening on port 8081');
});
```

```json
// package.json
{
  "dependencies": {
    "express": "*",
    "redis": "2.8.0"
  },
  "scripts": {
    "start": "node index.js"
  }
}
```

```Dockerfile
FROM node:alpine

WORKDIR '/app'

COPY ./package.json ./
RUN npm install

COPY ./ ./

CMD ["npm", "start"]
```

Build script: `docker build -t immigration9/visits:latest .`
Run script: `docker run immigration9/visits`

## Docker-Compose 도입
이렇게 하면 무조건 오류가 난다. Redis 서버가 안떠있기 때문.
그렇지만 Redis server를 띄워도 마땅히 서로 연결할 방법이 없다.
이 과정을 쉽게 해결하기 위해 docker-compose를 사용한다.

1. CLI로부터의 독립
2. 동시에 여러대의 Docker 컨테이너 실행
3. `docker run`으로 실행시에 작성하던 긴 argument들을 자동화

docker-compose.yml 파일
```yaml
version: '3'
services: 
  redis-server:
    image: 'redis'
  node-app:
    build: ./
    ports:
      - "3000:8081"
```

이렇게 함으로써 이 둘은 같은 네트워크 안에 속하게 된다.
이제 파일을 아래와 같이 수정해주자
```javascript
const express = require('express');
const redis = require('redis');

const app = express();
const client = redis.createClient({
  host: 'redis-server',
  port: 6379
});
client.set('visits', 0);

app.get('/', (req, res) => {
  client.get('visits', (err, visits) => {
    res.send('Number of visits is ' + visits);
    client.set('visits', parseInt(visits) + 1);
  });
});

app.listen(8081, () => {
  console.log('Listening on port 8081');
});
```

놀랍게도 이제 'redis-server'를 Docker가 같이 찾을 수 있다. 이제 실행해주는 과정만 남았다.
`docker run (image)` -> `docker-compose up`
`docker build.` + `docker run (image)` -> `docker-compose up --build` (리빌딩시에 사용)

* `docker-compose up` 명령어를 통해 실행하자.

* `docker-compose up -d`: 백그라운드에서 실행하기

* `docker-compose down`로 중지할 수 있음 (기존이였다면 모든 컨테이너들에 대해 `docker stop (container id)`를 해줘야하던 것을 자동화)

## docker-compose를 이용하여 container 정비
우선 의도적으로 코드에 오류를 넣는다
```javascript
const express = require('express');
const redis = require('redis');
const process = require('process');

const app = express();
const client = redis.createClient({
  host: 'redis-server',
  port: 6379
});
client.set('visits', 0);

app.get('/', (req, res) => {
  process.exit(0) // 의도적인 오류
  client.get('visits', (err, visits) => {
    res.send('Number of visits is ' + visits);
    client.set('visits', parseInt(visits) + 1);
  });
});

app.listen(8081, () => {
  console.log('Listening on port 8081');
});

```
다시 빌드를 진행한다: `docker-compose up --build`
이제 `localhost:3000`로 접속하면, 다음과 같은 오류 메시지가 뜨면서 컨테이너가 죽는다: `docker-compose_node-app_1_8510497ff75d exited with code 0`

1. 재시작 정책
* "no": 컨테이너가 죽거나 충돌났을 때 재시작하지 말 것
* always: 컨테이너가 멈추면 이유여하 막론하고 재시작할 것
* on-failure: 에러코드를 출력하며 컨테이너가 멈출 경우 재시작
* unless-stopped: 강제로 멈춘 것이 아니라면 언제나 재시작

```yaml
version: '3'
services: 
  redis-server:
    image: 'redis'
  node-app:
    restart: always
    build: ./
    ports:
      - "3000:8081"
```
node-app 아래에 `restart: always`를 추가하면 죽지 않고 되살아난다.
`on-failure`가 성립하기 위해서는, 위에서 설정한 `process.exit(0)`를 다른 숫자로 바꿔줘야 한다. 0은 정상적으로 마침을 뜻한다.
위에 다른 것과 다르게 no만 "no"로 되어있는 이유는, yaml 파일에서 no는 false라는 뜻이기 때문이다.

## docker-compose를 사용하여 컨테이너 상태 확인
* `docker-compose ps`: 해당 디렉토리에서 실행해야 함.


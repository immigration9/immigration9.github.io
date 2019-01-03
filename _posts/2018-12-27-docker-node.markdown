---
layout: post
title:  "Docker로 Node App 띄우기"
date:   2018-12-27 22:00:00 +09:00
categories: "docker"
published: false
---


## 이미지 태깅
`docker build -t immigration9/redis:latest .`
(docker id)/(image name):(version)으로 구성된다.
뒤에 점은 build context라고 한다.

## 컨테이너로 이미지 만들기 
* 알고만 있으면 좋다. 쓸 일은 거의 없음
```Dockerfile
FROM alpine
RUN apk add --update redis
```

`docker commit -c 'CMD ["redis-server"]' (container id)`
commit은 진행 중인 컨테이너를 이미지로 만들 수 있다.

## NodeJS Webapp Dockerfile로 만들기

```Dockerfile
FROM node:alpine

COPY ./ ./
RUN npm install

EXPOSE 3000
CMD ["npm", "start"]
```

* COPY (source directory) (destination inside container)

## Docker port mapping
`docker run -p (Route incoming request port on local host) : (port inside the container) (image id)`

## Docker working directory
```Dockerfile
FROM node:alpine

WORKDIR /usr/app
COPY ./ ./
RUN npm install

EXPOSE 3000
CMD ["npm", "start"]
```
* WORKDIR 이후에 발생하는 모든 명령들은 컨테이너 내부의 해당 경로에 상대적으로 실행된다.

## 불필요한 Rebuilding 제거
파일 변경시
```javascript
const express = require('express');

const app = express();

app.get('/', (req, res) => {
  // res.send('Hello, World!');
  res.send('Bye, World!');
});

app.listen(3000, () => {
  console.log('Listening on port 3000');
});
```

```Dockerfile
FROM node:alpine

WORKDIR /usr/app

COPY ./package.json ./
RUN npm install

COPY ./ ./

EXPOSE 3000
CMD ["npm", "start"]
```
package.json 만 먼저 복사를 해서, npm install은 package.json 파일에 변화가 없기 때문에 기존에 캐시를 가져다 사용한다.
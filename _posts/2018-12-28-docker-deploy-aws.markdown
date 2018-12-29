---
layout: post
title:  "Docker로 Production까지"
date:   2018-12-29 11:00:00 +09:00
categories: "docker"
published: true
---

## Setting up
* Github repository 생성: docker-react
* Travis CI에 연동 후 설정
* AWS 계정 만든 후 Elastic Beanstalk 프로젝트 생성

feature branch 생성: `git checkout -b feature`

1. Dockerfile.dev (Development & Testing)
```Dockerfile
FROM node:alpine

WORKDIR '/app'

COPY package.json ./
RUN npm install

COPY ./ ./

CMD ["npm", "run", "start"]
```

2. Dockerfile (Production)
```Dockerfile
FROM node:alpine as builder

WORKDIR /app
COPY ./package.json /app

RUN npm install
COPY ./ ./
RUN npm run build

FROM nginx
EXPOSE 80
COPY --from=builder /app/build /usr/share/nginx/html
```
* Beanstalk이 어떤 포트가 노출되는지 인식하기 위해서는 별도로 EXPOSE 항목을 작성해줘야한다. 일반적인 환경에서는 전혀 필요 없음

```yaml
sudo: required
services:
  - docker

before_install:
  - docker build -t immigration9/docker-react -f Dockerfile.dev .

script:
  - docker run immigration9/docker-react npm run test -- --coverage

deploy:
  provider: elasticbeanstalk
  region: "ap-northeast-2"
  app: "docker-react"
  env: "DockerReact-env"
  bucket_name: "elasticbeanstalk-ap-northeast-2-878265607169"
  bucket_path: "docker-react" # same as app name
  on:
    branch: master
  access_key_id: "$AWS_ACCESS_KEY"
  secret_access_key:
    secure: "$AWS_SECRET_KEY"
```

* sudo / service: sudo는 docker를 위해 필요하고, service를 정의해주면 해당 서비스가 시작된다.
* before_install은 스크립트 실행 전에 돌아가야하는 항목
* script는 실질적으로 돌아가야하는 항목들이 기록된다.
* deploy 부분에서 실직적으로 elastic beanstalk에 등록된다.
* IAM에서 access_key와 secret_key를 발급 받은 다음에, Travis CI 설정에 등록해준다. 이 때 이름을 각각 설정 파일에서 사용할 변수명으로 설정해주면 된다. (ex. AWS_ACCESS_KEY)





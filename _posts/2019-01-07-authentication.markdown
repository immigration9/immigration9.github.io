---
layout: post
title: 'Authentication'
date: 2019-01-07 01:00:00 +09:00
categories: 'javascript'
published: true
---

## 인증 과정

1. Client: 아이디와 비밀번호 전송
2. Server: 아이디와 비밀번호 확인
3. Server: 인증 확인. **'토큰'과 같은 형식의 정보 전송 (향후 요청시에 사용)**
4. Client: 인증된 요청 전송 가능
5. Client: 인증이 필요한 정보 필요. 인증 정보 전송 (이전에 발급 받은 토큰과 같은 형식)
6. Server: 인증 정보를 기반으로 해당 정보 전송

- 향후 같은 절차 발생시 인증 정보를 기반으로 정보 전송

## 쿠키와 토큰

쿠키

- 모든 요청에 자동으로 삽입된다.
- 각각의 도메인별로 유니크하다
- 다른 도메인으로 전송이 불가능하다

토큰

- 수동으로 연결시켜줘야 한다.
- 다른 도메인으로 전송이 가능하다.

## 서버 세팅

```
mkdir server
cd server
npm install --save express mongoose morgan body-parser nodemon
```

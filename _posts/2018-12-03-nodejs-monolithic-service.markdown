---
layout: post
title:  "Practice MicroService Architecture in NodeJS"
date:   2018-12-04 00:47:00 +09:00
categories: "nodejs"
---

## Reference


## 마이크로서비스란
* 하나의 독립적인 프로세스 하나를 의미
* 개발과 배포에 상호 독립적
* 기술 독립성
* 독립적인 데이터 저장소 소유 가능
* 각 마이크로서비스는 각자 가진 네트워크 기능으로 통신할 수 있음.

## 단점
* 공유자원 접근의 어려움
* 배포와 실행 복잡
* 분산 시스템 구현이 어려움 (분산 네트워크)

주로 메시지-큐 기능을 이용하여 해결한다. I/O 작업을 별도로 처리하는 서버를 두고 API서버는 큐로 데이터를 전송한 후 I/O가 처리되면 다시 전달받아 클라이언트에 응답을 송신한다. -> Node.js가 내부적으로 어느정도 해결해준다.



The Aha moment
```javascript
function func(callback) {
  callback("callback!");
}

func((params) => {
  console.log(params);
});

function func((params) => { console.log(params) }) {
  ((params) => {
    console.log(params)
  })("callback")
}
```
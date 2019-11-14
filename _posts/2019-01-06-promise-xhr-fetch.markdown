---
layout: post
title: 'Promise / XMLHttpRequest / Fetch'
date: 2019-01-06 18:00:00 +09:00
categories: 'javascript'
published: true
---

## Promise란?

Promise 객체는 비동기 작업이 맞을 미래의 완료 또는 실패와 그 결과값을 나타냅니다 -> 쉽게 말해 비동기 작업 후 결과를 반환한다.

Promise의 상태는 3가지로 나뉜다.

- 대기(Pending): 작업 수행 전의 초기의 상태
- 이행(Fulfilled, Resolved): 작업이 성공적으로 완료됨
- 거부(Rejected): 작업에 실패함

## 예제

```javascript
const promise = new Promise((resolve, reject) => {
  let random = Math.random() * 10;
  setTimeout(() => {
    if (random > 5) {
      resolve();
    } else {
      reject();
    }
  }, 3000);
});
```

위 예제는 Promise 객체의 구현체이다. `promise`가 호출되었을 때, 결과값에 따라 랜덤으로 이행이나 거부가 반환되도록 설정되어있다. 아래의 항목을 통해 호출해보자.

```javascript
promise
  .then(() => {
    let number = 5;
    return number + 5;
  })
  .then(data => {
    console.log(data);
  })
  .catch(() => {
    console.log('rejected');
  });
```

`then`과 같은 경우. 체이닝을 통해 다음 행동을 같이 진행할 수 있다. 이전 항목의 데이터를 다음으로 넘기고자 할 경우, 리턴을 해주면 되고, 해당 값을 다음 `then`에서 받아서 사용하면 된다.

## Promise로 HTTP Request 보내기 (mdn)

MDN에 있는 Promise 기반의 XMLHttpRequest를 약간 변형시켜보자

```javascript
function myAsyncFunction(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    xhr.onload = () => {
      resolve(xhr.response);
    };
    xhr.onerror = () => {
      reject(xhr.response);
    };
    xhr.send();
  });
}
```

이제 URL을 넣으면, XMLHttpRequest response를 리턴하게 된다.
호출할 때는 다음과 같이 체이닝하여 호출해준다.

```javascript
myAsyncFunction('https://jsonplaceholder.typicode.com/posts/')
  .then(data => JSON.parse(data))
  .then(data => console.log(data));
```

첫 번째 `then`은 `XMLHttpRequest`의 `response`를 받은 뒤, JSON으로 파싱하여 반환한다.
두 번째 `then`은 해당 JSON값을 콘솔에 출력한다.

## XMLHttpRequest

XHR(XMLHttpRequest)는 서버와의 상호작용을 위한 객체이다.
이름 때문에 XML만 받아올 것 같지만 사실 HTTP 이외의 다른 프로토콜도 다 지원된다.

```javascript
let req = new XMLHttpRequest();
req.open('GET', 'https://https://jsonplaceholder.typicode.com/todos/1', false);
req.send();
if (req.status == 200) {
  console.log(req.responseText);
}
```

여기서 이 코드를 설명하는 이유는, 이 코드 자체는 동기적으로 동작하기 때문에, 브라우저에서 다른 행동들을 일시적으로 멈춘다. (이 과정이 끝날 때까지). 그렇기 때문에 `Promise`와 같은 비동기 형태의 객체와 같이 사용하는 것이 필수적이다.

`XMLHttpRequest`에도 역시나 비동기적으로 작동하도록 하는 함수가 존재한다.

```javascript
req.onreadystatechange = function(evt) {
  // ...todo
};
```

## XMLHttpRequest로 진행 상황 모니터링하기

```javascript
const onProgress = e => {
  if (e.lengthComputable) {
    let percentComplete = (e.loaded / e.total) * 100;
    console.log(percentComplete);
  }
};

const onLoad = e => {
  console.log('The progress has been finished!');
};

const onError = e => {
  alert('Error ' + e.target.status + ' occurred while receiving the document.');
};

const httpRequest = new XMLHttpRequest();
httpRequest.open('GET', 'https://jsonplaceholder.typicode.com/posts/', true);
httpRequest.onprogress = onProgress;
httpRequest.onload = onLoad;
httpRequest.onerror = onError;
httpRequest.send();
```

위와 같은 코드를 통해 Progress Bar를 만들 수도 있다.
다만 이 경우, `lengthComputable`이 `true`가 되기 위해서는 서버에서 전송할 때 `Content-Length` header를 포함하여 보내줘야 한다.

## Fetch

`fetch`는 `XMLHttpRequest`의 약간 더 강력한 버전이라고 생각하면 편할 것 같다. 같은 역할을 수행할 수 있으나, 좀 더 유연하고 쉽게 기능을 구현할 수 있다.

```javascript
const url = 'https://jsonplaceholder.typicode.com/posts/';

fetch(url).then(data => console.log(data));

fetch(url)
  .then(response => response.json())
  .then(data => console.log(data));
```

기본적인 로직 자체는 Promise와 상당히 유사하다. 이 내부에 있는 Interface나 Header, Body와 같은 별도로 사용할 수 있는 객체들이 다수 있지만, 인터넷 익스플로러에서는 전혀 지원하지 않기 때문에, 지원이 종료되는 그 순간을 기다려보자.

## Fetch가 가진 문제점

`fetch()` 함수의 경우 HTTP Error 상태가 reject와 연결되지 않는다.
즉, 404, 500같은 코드가 반환되더라도, `catch`로 빠지지 않고 `then`으로 빠지게 된다.
아래와 같은 예제를 한 번 보자.

```javascript
const url = 'https://jsonplaceholder.typicode.com/posts/';
const falseUrl = 'https://jsonplaceholder.typicode.com/wrongurl/';
const falseUrl2 = 'https://notevenaurl123123123123123.com/';

fetch(url).then(data => console.log(data));

fetch(url)
  .then(response => response.json())
  .then(data => console.log(data));

fetch(falseUrl)
  .then(response => console.log(response))
  .catch(error => console.log(error));

fetch(falseUrl2)
  .then(response => console.log(response))
  .catch(error => console.log(error));
```

`falseUrl`은 의도적으로 404 코드를 유도한 URL이고, `falseUrl2`는 존재하지 않는 도메인이다.
`fetch()`함수는 후자의 경우에만 `catch`로 유도되고, 그렇지 않을 경우에는 전부 `then`으로 들어간다.
만약에 이 함수를 쓰려고 한다면, `then`에서 조건문을 통해 분기해줘야 한다.

- 추가: 일반적으로 쿠키를 보내거나 받지 않는다. 쿠키 전송을 위해서는 별도의 자격증명 옵션(credentials)을 설정해줘야 한다.

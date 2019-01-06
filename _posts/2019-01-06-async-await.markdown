---
layout: post
title:  "Async Await"
date:   2019-01-06 19:00:00 +09:00
categories: "javascript"
published: true
---

## Async Await란?
`async/await`는 ES8 스펙에 포함되는 함수들이다. 
목표는 기존의 콜백이 가져올 수 있는 이른바 '콜백 지옥' 문제를 해결하는 역할을 수행하게 된다고 보면 될 것 같다.
앞장에서 설명했던 `Promise`나 `fetch`가 안에서 daisy chain 되어서 콜백 안에 콜백이 들어갈 경우 굉장히 코드가 지저분해 보일 수 있는 단점을 극복하게 된다.

## 기존의 콜백 형태
아래 코드를 통해 콜백이 작동하는 방식을 살펴보자

```javascript
const firstCb = () => {
  let promise = new Promise((res, rej) => {
    setTimeout(() => {
      res([1])
    }, 1000);
  });
  return promise;
}
const secondCb = (data) => {
  let promise = new Promise((res, rej) => {
    setTimeout(() => {
      res(data.concat(2))
    }, 1000);
  });
  return promise;
}
const thirdCb = (data) => {
  let promise = new Promise((res, rej) => {
    setTimeout(() => {
      res(data.concat(3))
    }, 1000);
  });
  return promise;
}

firstCb()
  .then(secondCb)
  .then(thirdCb)
  .then(data => console.log(data));
```

콜백을 여러개를 체이닝하였다.
코드 자체는 깔끔해졌지만, `catch`와 `deferred`와 같은 항목들도 고민 해봐야한다.

## async / await 사용
`async` 함수는 `AsyncFunction` 객체를 반환하는 비동기 함수를 반환한다.

`async` 함수가 호출되었을 때, `Promise` 객체를 반환한다. `async` 함수가 값을 반환하면, `Promise` 객체는 `resolved` 상태가 되며 값을 반환한다.
`async` 함수는 `await` 표현식을 가지고 있으며, `await`는 비동기 작업을 일시정지 시키고, `Promise` 객체의 종료를 기다린 뒤, 완료되면 작업을 속행시킨다.

아래 코드는 `Promise`에서 `async / await`로 변경된 코드이다.
```javascript
// 기존
function getData(url) {
  return downloadData(url)
    .then(v => processData(v))
    .then(v => downloadData(`${url}/${v[44].id}`))
    .then(v => console.log(v))
    .catch(e => fallbackData(e));
}

function downloadData(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);
    xhr.onload = () => {
      resolve(xhr.response)
    };
    xhr.onerror = () => {
      reject(xhr.response)
    };
    xhr.send();
  });
}

function fallbackData(e) {
  alert(e);
}
function processData(v) {
  return JSON.parse(v);
}
```

```javascript
// async await
async function getData(url) {
  let data;
  let final;
  try {
    data = await downloadData(url);
    final = await processData(data);
    final = await downloadData(`${url}/${processed[44].id}`)
  } catch (e) {
    final = await fallbackData(url);
  }
  return final;
}
```

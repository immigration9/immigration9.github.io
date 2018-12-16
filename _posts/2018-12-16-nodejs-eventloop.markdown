---
layout: post
title:  "NodeJS Event Loop, Timers, and `process.nextTick()`"
date:   2018-12-16 14:00:00 +09:00
categories: "nodejs"
---

## Reference
본 글은 Node.JS 공식 도큐먼트 글을 번역/요약/정리한 글입니다.
[The Node.js Event Loop, Timers, and process.nextTick()][reference-01]

## Event Loop이란 무엇인가?
Event Loop은 Javascript가 싱글 쓰레드로 구성되어있음에도 불구하고 Node.JS가 non-blocking I/O 작업을 할 수 있도록 해준다. 가능할 때 시스템 커널에서 진행중인 작업을 내리는 작업을 수행한다.
-> TODO: non-blocking I/O 설명 추가

대부분의 현대 커널들은 멀티 쓰레드를 지원하기 때문에, 다양한 작업들을 백그라운드에서 다룰 수 있다. 그 중 하나가 완료되면, 커널은 Node.js로 알려 poll queue에 적합한 콜백이 올라갈 수 있도록 해준다. (그렇게 되면 추후에 실행된다)

## Event Loop 구조
Node.js가 시작할 때, Event loop을 초기화하고, 비동기 API Call, 스캐쥴 타이머, 혹은 `process.nextTick()`을 호출하는 입력된 스크립트를 생성하고, Event loop 생성을 시작한다.

아래 다이어그램은 Event loop 실행 순서를 보여준다. 이 아래 항목들은 각각 phase
timers -> pending callbacks -> idle / prepare -> poll (connections) -> check -> close callbacks (다시 timers로)

각각의 페이즈는 실행될 콜백 목록이 FIFO 큐로 구성되어 있다. 각각의 페이즈는 개별적으로 용도와 목적을 가지고 있지만, 일반적으로 Event Loop이 하나의 페이즈에 진입하면, 해당하는 작업을 진행하고, 페이즈 큐 내에서 작업이 전부 소진되거나 최대 콜백 호출량(maximum number of callbacks)을 부를 때까지 진행된다. 페이즈 종료지점에 도달하면 Event loop은 다음 페이즈로 이동한다.

이 모든 작업들은 더 많은 작업들을 스케쥴링할 수도 있고, poll 페이즈에서 진행되는 새로운 이벤트들은 커널에 의해서 queued되기 때문에 poll 이벤트는 polling event가 진행되는 동안에도 queued 될 수 있다. (* 이 부분 명료화 필요). 결과적으로, 장기간 진행되는 콜백들은 poll 페이즈로 하여금 timer의 임계값을 넘어 진행되도록 할 수 있다.

## Phase Overview
* timers: `setTimeout()` 혹은 `setInterval()`로 스케쥴링된 콜백들을 실행
* pending callbacks: 루프 다음번 실행에 지연된 I/O 콜백을 실행
* idle, prepare: 내부적으로만 사용됨
* poll: 새로운 I/O 이벤트를 검색, I/O 관련 콜백 (close callbacks, 타이머에 의해 스케쥴된 콜백, setImmediate()을 제외하고) 거의 모든 콜백을 수행. 노드는 적절할 때 작업을 여기서 블록한다.
* check: `setImmediate()` 콜백이 호출됨
* close callbacks: `socket.on('close',...)`와 같은 close 콜백

Event loop 실행 간에, Node.js는 기다리고 있는 비동기 I/O나 타이머를 확인하고, 없을 경우 종료한다.

## Phases in Detail
### timers
타이머는 사용자가 실행되기 원하는 정확한 시간이 아닌, 제공된 콜백이 실행될 수 있는 임계 값을 지정한다. 타이머 콜백은 지정된 시간이 지난 후에 스케쥴링될 수 있는 가장 이른 시간에 실행된다. (운영체제 스케쥴링이나 다른 콜백으로 인해서 지연될 수는 있다)

예를들어 아래 코드를 보자. 100ms 후에 실행되는 타임아웃을 스케쥴링한 뒤에, 스크립트가 비동기로 파일을 읽는데 95ms가 걸린다고 가정하자.
```javascript
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);


// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```
Event loop가 poll 페이즈에 진입하면, `fs.readFile()`이 완료되지 않았기 때문에 빈 큐 상태이고, 타이머의 임계값 중 가장 빨리 도달하는 항목을 기다린다. 95ms 가 지난 후에, `fs.readFile()`이 파일 읽기를 완료하고, 완료하는데 10ms가 소요되는 콜백이 poll 큐에 추가되고 실행된다. 콜백이 완료되면, 큐에 남아있는 콜백이 더 이상 없기 때문에, Event loop은 임계값에 도달한 타이머가 있음을 확인하고 다시 타이머 페이즈로 돌아가서 콜백을 실행한다. 이 예제에서 타이머가 스케쥴링되고 실행되는 데에는 약 105ms의 딜레이가 존재하였다.

## pending callbacks
본 페이즈는 TCP 에러 타입과 같이 시스템 작업 콜백을 실행한다. 예를들어, TCP 소켓이 연결하고자 할 때 `ECONNREFUSED`를 전달받으면, 몇몇 *nix 시스템들은 이 에러를 보고할 때까지 기달리고자 한다. 해당 작업의 큐는 pending callbacks 페이즈에서 실행된다.

## poll
* 폴링(polling): CPU가 일정한 시간 간격을 두고 각 자원들의 상태를 주기적으로 확인하는 방식.
poll 페이즈는 두 개의 주요한 역할을 수행한다.
1. I/O 작업을 위해 얼마 동안 블록과 폴링을 해야하는지 계산
2. poll 큐에 있는 이벤트들을 실행

작업하다보니 한글 가이드도 있네... 한글 가이드 볼걸 (나머지는 한글 가이드에서)
https://nodejs.org/ko/docs/guides/event-loop-timers-and-nexttick/

## process.nextTick() 이해하기
`process.nextTick()`은 Event loop의 일부가 아니다. 대신, `process.nextTick()`은 이벤트 루프의 현재 단계와 관계없이 현재 작업이 완료된 후에 처리된다.

다이어그램에 표현된 단계 중 한 곳에서 `process.nextTick()`을 호출하면 `process.nextTick()`에 전달한 모든 콜백은 언제나 이벤트 루프를 계속 진행하기 전에 처리된다.

```javascript
let bar;

function someAsyncApiCall(callback) { callback(); }

someAsyncApiCall(() => {
  console.log('bar', bar); // undefined
});

bar = 1;
```
결국 위의 코드는, 마치 비동기처럼 생겼지만, 코드가 실행되었을 때 별도의 비동기 작업이 없기 때문에 사실상 동기 코드로 실행된다. 실행이 되었을 때 결국 `someAsyncApiCall()`에 할당된 콜백은 같은 Event loop 상에서 실행되게 된다. 결국, `bar`는 스크립트가 완료되지 않았기 때문에 할당되지 않는다.

콜백 내부에 `process.nextTick()`을 제공함으로써, 스크립트는 콜백이 실행되기 전에 완료단계까지 실행될 수 있고, 이로써 변수나 함수 할당이 가능하다. 이와 더불어 Event loop이 진행되지 않도록 하는 장점도 갖는다.

아래 코드를 하나 더 보자. 도큐멘테이션에서 권장하는 `setImmediate()`를 사용하였다.
```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  let that = this;
  EventEmitter.call(this);
  
  emitEvent(this);
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});

function emitEvent(self) {
  self.emit('event');
}
```
애석하게도, 해당 이벤트는 시작되지 않는다. `myEmitter.on()`에 대한 정의가 생성자의 단계에서는 성립되지 않았기 때문이다. 생성과 동시에 실행시키기 위해서는 생성자에 있는 `emitEvent`를 다음과 같이 `setImmediate()` 안에 넣어줘야 한다.

```javascript
function MyEmitter() {
  let that = this;
  EventEmitter.call(this);
  
  setImmediate(emitEvent, this)
}
```
`setImmediate`는 현재 poll 단계가 완료되면 스크립트가 실행되도록 설계되어있다. 이어진 순회나 Event loop의 'tick'에서 실행된다. (`process.nextTick()`과 이름이 반대라서 좀 헷갈린다)

[reference-01]:https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/

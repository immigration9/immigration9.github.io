---
layout: post
title:  "Observer Pattern (감시자 패턴)"
date:   2018-11-20 01:00
categories: "javascript"
---
## References
[자바스크립트 코딩 기법과 핵심 패턴][reference-01]

[THE OBSERVER PATTERN IN JAVASCRIPT EXPLAINED][reference-02]


## 감시자 패턴이란?
감시자 패턴은 어떤 객체가 다른 객체의 메서드를 호출하는 대신, 객체의 특별한 행동을 구독해 알림을 받는 구조를 갖는다. 예를들어, mouse, keyboard event와 같은 브라우저 이벤트는 감시자 패턴의 일종이라고 할 수 있다. 패턴이 가지고 있는 주요 목적은 객체들 상호간의 결합도를 낮추는 것이다.
구조는 Observing을 하는 Subscriber(구독자)와 Event emit을 하는 Publisher(발행자)로 나뉜다. 발행자는 이벤트 발생시 구독자들을 호출한다.

## Basic Code
```javascript
/**
 * Observable class
 */
class Observable {
  constructor() {
    this.observers = [];
  }

  subscribe(f) {
    this.observers.push(f);
  }

  unsubscribe(f) {
    this.observers = this.observers.filter(subscriber => subscriber !== f);
  }

  notify(data) {
    this.observers.forEach(observer => observer(data));
  }
}

/**
 * Implementation
 */
// some DOM references
const input = document.querySelector('.js-input');
const p1 = document.querySelector('.js-p1');
const p2 = document.querySelector('.js-p2');
const p3 = document.querySelector('.js-p3');

// some actions to add to the observers array
const updateP1 = text => p1.textContent = text;
const updateP2 = text => p2.textContent = text;
const updateP3 = text => p3.textContent = text;

// instantiate new Observer class
const headingsObserver = new Observable();

// subscribe to some observers
headingsObserver.subscribe(updateP1);
headingsObserver.subscribe(updateP2);
headingsObserver.subscribe(updateP3);

// notify all observers about new data on event
input.addEventListener('keyup', e => {
  headingsObserver.notify(e.target.value);
});

```



[reference-01]:https://book.naver.com/bookdb/book_detail.nhn?bid=6763510
[reference-02]:https://pawelgrzybek.com/the-observer-pattern-in-javascript-explained/
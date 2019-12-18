---
layout: post
title: 'Functional Programming: Generator & Iterator'
date: 2019-12-18 23:00:00 +09:00
categories: 'functionalprogramming'
published: false
---

## Generator / Iterator

- 제너레이터: 이터레이터이자 이터러블을 생성하는 함수. 즉, 이터레이터를 생성하는 함수

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}
let iter = gen();
console.log(iter.next()); // { value: 1, done: false }
console.log(iter.next()); // { value: 2, done: false }
console.log(iter.next()); // { value: 3, done: false }
console.log(iter.next()); // { value: undefined, done: true }
```

- `return` 값도 넣을 수 있긴한데, 그렇게 되면 프로토콜 상에서는 노출되지 않는다.

## Odds

```javascript
function* infinity(i = 0) {
  while (true) yield i++;
}
function* limit(l, iter) {
  for (const a of iter) {
    yield a;
    if (a == l) return;
  }
}

function* odds(l) {
  for (const a of limit(l, infinity(1))) {
    if (a % 2) yield a;
  }
}

let iter2 = odds(10);
```

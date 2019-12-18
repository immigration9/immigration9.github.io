---
layout: post
title: 'Functional Programming 101'
date: 2019-12-18 22:00:00 +09:00
categories: 'functionalprogramming'
published: false
---

## 고차 함수

### 함수를 인자로 받아서 실행하는 함수

```javascript
const apply1 = f => f(1);
const add2 = a => a + 2;

log(apply1(add2));
log(apply1(a => a - 1));

const times = (f, n) => {
  let i = -1;
  while (++i < n) f(i);
};

times(log, 3);
times(a => log(a + 10), 3);
```

### 함수를 만들어 리턴하는 함수 (클로저를 만들어 리턴하는 함수)

```javascript
const addMaker = a => b => a + b;
const add10 = addMaker(10); // a => a + 10
log(add10(5)); // 15
log(add10(10)); // 20
```

## ES6에서의 순회

```javascript
for (const a of list) {
  console.log(a);
}
```

### Array 순회

```javascript
const arr = [1, 2, 3];
for (const a of arr) console.log(a);
```

- 그렇다면 Array는 어떻게 Iterable하게 동작하는 것일까?
  Javascript Symbol.iterator를 통해 실험해본다.

```javascript
arr[Symbol.iterator] = null;
```

이 과정을 거치고 나면, 더 이상 Array는 순회가 되지 않는다.
즉, for - of 와 `Symbol.iterator`와 연관관계가 있다는 것을 알 수 있다.

### Set 순회

```javascript
const set = new Set([1, 2, 3]);
for (const a of set) console.log(a);
```

### Map 순회

```javascript
const map = new Map(['a', 1], ['b', 2], ['c', 3]);
for (const a of map) console.log(a);
```

## Iterator / Iterable Protocol

- Iterable: Iterator를 리턴하는 [Symbol.iterator]()를 가진 값
  - 즉, Array, Set, Map은 Iterable이라 할 수 있다.
- Iterator: `{ value, done }` 객체를 리턴하는 `next()`를 가진 값
- Iterable / Iterator Protocol: 이터러블을 for ... of, 전개 연산자 등과 함께 동작하다록 하는 규약

### Array를 통한 Iterable 확인

```javascript
const arr = [1, 2, 3];
const iter = arr[Symbol.iterator]();

iter.next(); // { value: 1, done: false }
iter.next(); // { value: 2, done: false }
iter.next(); // { value: 3, done: false }
iter.next(); // { value: undefined, done: true }
```

여기서 for ... of 문법도 같은 방식으로 동작한다는 것을 알 수 있다.

```javascript
const iter = arr[Symbol.iterator]();

iter.next();
for (const a of iter) console.log(a); // 이 경우, 2와 3만 리턴된다.
```

## 사용자 정의 이터러블

```javascript
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      }
    };
  }
};
let iterator = iterable[Symbol.iterator]();
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());

for (const a of iterable) console.log(a);
```

사용자 정의 이터러블은 `[Symbol.iterator]()`를 멤버로 가지는 객체다.

### Well-formed iterable

위에 언급한 iterable은 iterable protocol에 따라 정상적으로 동작하지만, iterator protocol에는 부합하지 않는다. 이게 뭔 소리일까 싶지만, 쉽게 말하자면 다음과 같다.

- iterable은 iterator를 반환한다.
- 문제는 iterator에는 `[Symbol.iterator]()` 함수가 존재하지 않기 때문에, iterator protocol에 따라 for ... of와 같은 항목에 넣어줄 수 없다.
- 이를 우리는 `non-well-formed iterable`이라고 한다.

이 상황을 회피하기 위해, iterator에도 동일하게 본인 스스로를 반환할 수 있도록 함수를 추가해줘야 한다.

```javascript
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
      [Symbol.iterator]() {
        return this;
      }
    };
  }
};
```

이제 이 과정을 통해, iterator를 for ... of syntax에 넣는 것으로 중간 지점부터 값을 반환할 수 있게 된다.

이는 단순히 Javascript 내장 Array, Set, Map 등에만 해당되는 것이 아니라, 기본적인 브라우저의 DOM을 컨트롤하는 항목들도 이를 포함하게 된다.

```javascript
const doms = document.querySelector('*');

for (const a of doms) console.log(a);
```

### 전개 연산자에서도 사용되는 iterable protocol

```javascript
[...a, ...b];
```

이런 전개 연산자들도 실제로는 iterable protocol을 준수한다.

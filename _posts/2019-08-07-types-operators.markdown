---
layout: post
title: 'Javascript - Types & Operators'
date: 2019-08-07 14:30:00 +09:00
categories: 'javascript'
published: true
---

## Javascript에서의 Typing

- Dynamic Typing: 타입을 런타임에 결정
- Primitive Types(원시 타입): 하나의 '값'을 나타내는 데이터 종류. 즉, 객체가 아니다.

* undefined
* null
* Number: floating point number라는 점이 중요하다.
* String
* Boolean
* Symbol: ES6에 새롭게 추가된 Spec

## Operators (연산자)

- Operator: 문법적으로 다르게 쓰여진 특별한 함수. 일반적으로 2개의 매개변수를 받아서 한 개의 결과값을 반환한다.
- 예를들어, + - x / 등이 있다.
- in-fix notation 방식으로 작동하여, 일반 함수와 달리, ()걸 넣지 않고, 중간에 연산자를 넣어 계산할 수 있다.

### Operator Precedence and Associativity

- Operator Precedence: 연산자 함수의 순서 (어떤 것이 먼저 불리는지)
  https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence

- Operator Associativity: 연산자 함수의 방향 (우향인지 좌향인지)

```javascript
var a = 2,
  b = 3,
  c = 4;

a = b = c;

console.log(a); // 4를 반환
console.log(b); // 4를 반환
console.log(c); // 4를 반환
```

이 예제에서 전부 4가 반환되는 이유는, assignment는 right-to-left 방향성을 가지고 있기 때문이다. 아래와 같은 순서로 할당 받는다.

1. b = c 값을 갖는다.
2. a = b 값을 갖는다.

## Coercion (형변환)

- Coercion: 하나의 타입으로 다른 타입으로 변환하는 것

```javascript
var a = 1 + '2';
console.log(a);
```

Javascript Engine이 형 변환을 진행한다.

## Comparison Operators

Number Native 함수를 함부로 쓰면 안되는 이유

```javascript
Number(undefined); // NaN를 반환
Number(null); // 0을 반환
```

보시다시피, 결과값이 일정하지 않다.

- 항상 `===`을 쓰자

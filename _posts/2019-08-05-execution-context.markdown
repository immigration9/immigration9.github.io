---
layout: post
title:  "Javascript Execution Contexts"
date:   2019-08-05 14:00:00 +09:00
categories: "javascript"
published: true
---

## Javascript Conceptual Stuff
* Syntax Parsers
* Lexical Environments
* Execution Contexts (실행 컨텍스트): Javascript 코드 실행시 Execution Context (eg. Javascript Engine)은 코드를 Wrapping한다.
가장 기초가 되는 Execution Context는 Global이다. Execution Context는 코드 실행 시 아래와 같은 두 개를 생성한다.
- Global Object
- 'this': this가 Global Object를 가르키는 것은, Function 안에 있지 않다는 것.

- Outer Environment

## Execution Context: Creation and Hoisting
```javascript
b();
console.log(a);

var a = 'Hello, World';

function b() {
  console.log('Called b');
}
```

b는 실행이 되고, a는 되지만, `undefined`가 반환된다.
이걸 Hoisting이라고 한다.
그런데 이 Hoisting이 단순히 선언이 먼저 된다! 라는 식으로 말하기엔 맞지 않다.

Execution Context (Javascript Engine의 작동)는 2개의 Phase를 거치게 된다.
1. Creation Phase
- Global Object, 'this', Outer Environment (if exists)가 생성된다.
- 변수와 함수들을 위해 메모리 공간 할당: 이를 Hoisting이라고 한다.
  - 함수는 메모리에 올라갔고, 변수의 값은 다음 단계에서 실행되기 때문에 Placeholder격으로 `undefined`값이 들어간다.
* 코딩시에 왠만해선 undefined로 직접적으로 선언하지 말자. 
```javascript
var a;
a = undefined; // 이런걸 하지 말자
```

2. Execution Phase

```javascript
function b() {
  console.log("Called b!");
}
b();

console.log(a);

var a = "Hello, World!";

console.log(a);
```

위와 같이 구성할 경우, `var a = "Hello, World!";`에서 a 변수에 할당된다.

## Single Threaded, Synchronous Execution
* Synchronous Execution: One at a time

## Function Invocation and The Execution Stack
* Function Invocation: Running a function, Javascript에서는 ()를 쓴다.
```javascript
function b() {}

function a() {
  b();
}

a();
```
이 코드를 실행하면
1. Global Execution Context가 실행된다. -> this, Global Object(window)를 만들고, b와 a 함수들을 메모리에 상주시킨다.
2. Execution Context 생성 & 실행: a 가 실행이 된다.
3. Execution Context 생성 & 실행: b 가 실행이 된다.

이 코드들이 Execution Stack에 쌓인다 (LIFO). 3번이 실행이 완료되면 Execution Stack에서 Pop되고, 2번이 Pop되는 순으로 진행된다.

즉, 함수(function)가 실행될 때마다, 각 함수를 위해 Execution Context가 생성된다.

## Functions, Context, and Variable Environments
* Variable Environment: 변수가 어디 위치한 것인지. (Scope)
```javascript
function b() {
  var myVar;
  console.log(myVar); // undefined를 반환
}
function a() {
  var myVar = 2;
  console.log(myVar); // 2를 반환
  b();
}
var myVar = 1;
console.log(myVar); // 1을 반환
a();
console.log(myVar); // 1을 반환한다
```

## The Scope Chain
```javascript
function b() {
  console.log(myVar); // 1을 반환한다
}
function a() {
  var myVar = 2;
  b();
}
var myVar = 1;
a();
```
b의 `myVar`는 Outer Environment를 참고하는데, 그것이 Global Execution Context이다.
여기서 Lexical Environment가 중요한데,
b의 Lexical Environment는 Global Execution Context기 때문이다. (thus, `myVar`가 없기 때문에, Outer Environment에서 찾는 것).
a도 동일하게 Global Execution Context로 Lexical Environment를 갖는다.

* 이 과정을 Scope Chain이라고 한다. 첫 번째 장소에서 찾지 못하면, 더 아래의 Scope Chain을 찾아서 값을 찾아낸다.

위 코드를 아래와 같이 수정해본다.
```javascript
function a() {

  function b() {
    console.log(myVar); // 2를 반환한다
  }

  var myVar = 2;
  b();
}
var myVar = 1;
a();
```
이 상황에서 b의 Lexical Environment는 a 안이기 때문에, Outer Environment인 a에 속한 `myVar` 값을 참고하게 된다.
만약 a 안에 myVar가 선언되어 있지 않다면, Global Execution Context 단계로 넘어가 1을 반환한다.

## Scope, ES6, and `let`
* Scope: 코드 상에 존재하는 변수의 위치
* `let`: Block Scoping이 가능하게 한다.

## What about Asynchronous Callbacks?
* Browser에는 Javascript Engine이외에도 Rendering Engine, HTTP Request 등이 있다.
* 다른 것과 조합하여 Asynchronous하게 작동할 수 있지만, Javascript Engine만 놓고 봤을 때는 Synchronous하게 작동한다.

### Event Queue
Stack이 비어 있으면, Javascript Engine은 Event Queue를 확인하고 있을 경우 실행시킨다.
예를들어, `click` 이벤트가 Queue에 있으면, clickHandler Execution Context가 실행된다.

* 실제로 Asynchronous라기 보다는, Browser가 Event Queue에 비동기적으로 작업을 넣는다고 생각하면 된다.

```javascript
function waitThreeSeconds() {
  var ms = 3000 + new Date().getTime();
  while (new Date() < ms) {}
  console.log("finished function");
}

function clickHandler() {
  console.log('clicked!');
}

document.addEventListener('click!', clickHandler)

waitThreeSeconds();
console.log("finished execution");
```



## 참고할 것
[https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0](Understanding Execution Context and Execution Stack in Javascript)

---
layout: post
title:  "Javascript Basics"
date:   2019-08-10 18:30:00 +09:00
categories: "javascript"
published: true
---

## Functional Programming이란

함수형 프로그래밍은 프로그램을 함수의 계산으로 바라보는 프로그래밍 패러다임이다. 아래와 같은 개념들이 보장되어야 한다.
- 1급 객체 (고차 함수)
- 불변성
- 순수 함수
- 합성 함수

## Mutable vs Immutable (비불변성 vs 불변성)

데이터가 Immutable일 때, 생성 후 상태는 변경되지 않는다. 불변성이 있는 객체의 경우, 변경 사항이 있을 때는 새로운 객체를 만든 후 새로운 변수에 할당한다.

아래와 같은 코드에서는, `for` 반복문을 돌면서, 함수의 매개변수가 아닌 값을 변화시킨다.

```javascript
var values = [ 1, 2, 3, 4, 5 ];
var sumOfValues = 0;

for (var i = 0; i < values.length; i++) { // 여기서는 i의 값이 계속 변이된다.
  sumOfValues += values[i]; // 이 부분에서 sumOfValues 값을 변화시킨다.
}

sumOfValues
```

함수형 프로그래밍에서는, 불변성을 보장하기 위해 아래와 같이 작성할 수 있다. 아래의 예제는, 재귀 형식을 사용하여 함수 외부에 존재하는 변수나 요소에 영향을 받지 않고, 오로지 매개변수에 따른 결과값을 반환할 수 있게 해준다.

```javascript
let list = [ 1, 2, 3, 4, 5 ];

function sum(list, accumulator) {
  if (list.length === 0) {
    return accumulator;
  }
  return sum(list.slice(1), accumulator + list[0]);
}

let total = sum(list, 0);

```

## First-class Object

함수를 다른 변수와 동일하게 다루는 언어를 '일급 함수'를 가졌다고 표현한다. 즉, 언어가 함수를 값으로 다룰 수 있다면 일급 함수를 갖는다고 할 수 있다.

* Javascript는 Function 자체가 Object의 인스턴스기 때문에 일급함수를 갖는다.
  
- 함수를 인자로 전달 (콜백 함수)
- 함수를 반환

## High-Order Function

고차 함수란, 함수를 인자로 전달받거나 함수를 결과로 반환하는 함수를 말한다.
Javascript의 함수가 일급 객체임으로 해당된다.

## Pure Javascript Function

순수 함수란, 함수에 들어온 매개변수만을 기반으로 값을 도출해내는 것을 의미한다.

## Promise / PromiseAll

Promise 객체는 비동기 작업이 맞이할 미래의 완료 또는 실패와 그 결과 값을 나타낸다.

## Event Delegation / Event Bubbling

이벤트 위임은 특정 노드로 Event Listener를 등록하지 않고, 하나의 부모 노드에 등록하는 것을 의미한다. 

예를들어, 아래와 같은 코드가 있다고 가정하자.

```javascript
<ul id="parent-list">
	<li id="post-1">Item 1</li>
	<li id="post-2">Item 2</li>
	<li id="post-3">Item 3</li>
</ul>
```

만약 여기서 내부에 있는 `li` 요소에 Click Event를 넣어야 하는데, 주기적으로 노드가 추가 / 제거 될 경우 어떻게 처리해야할 것인가?
아래와 같이, Event Bubble을 `ul` 요소로 올려주고, 실제로 클릭 된 노드를 찾아가도록 하면 된다.

```javascript
document.getElementById("parent-id").addEventListener("click", function(evt) {
  if (evt.target && evt.target.nodeName === "LI") {
    console.log("Selected Item: " + e.target.id + " was clicked!");
  }
});
```

그렇다면, `ul li`와 같은 관계까 아닌, 다른 `div` 안에 속한 특정 태그에 클래스를 갖는 경우 어떻게 해야할까?
`matches` API를 사용하면 된다.

```javascript
document.getElementById("someDiv").addEventListener("click", function(evt) {
  if (evt.target && e.target.matches("a.someClass")) {
    console.log("Selected some class: " + e.target.className + "" )
  }
});
```

## References

[https://davidwalsh.name/event-delegate](How JavaScript Event Delegation Works)

[https://www.freecodecamp.org/news/functional-programming-principles-in-javascript-1b8fc6c3563f/](Functional Programming Principles in Javascript)
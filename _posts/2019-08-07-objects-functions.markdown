---
layout: post
title: 'Javascript - Objects & Functions'
date: 2019-08-07 15:30:00 +09:00
categories: 'javascript'
published: true
---

## Objects and the dot

- Javascript에서의 Object(객체): Name-Value로 이루어진 페어의 집합. 아래의 값들의 주소를 가질 수 있다.

* 원시타입
* 다른 객체
* 함수 (메소드)

```javascript
var person = new Object();
person['firstname'] = 'John';
person['lastname'] = 'Doe';

var fnProperty = 'firstname';

console.log(person[fnProperty]);
console.log(person.firstname);
```

## 객체와 객체 리터럴 (Objects & Object Literals)

```javascript
var person = {
  firstname: 'John',
  lastname: 'Doe'
}; // new Object() 와 동일
```

## By Value & By Reference

- By Value: 원시값의 경우, 다른 변수에 할당할 경우, 복사 값을 할당한다

```javascript
// By Value
var a = 3;
var b = a;

b = 5;
console.log(a); // 3을 반환
console.log(b); // 5를 반환
```

- By Reference: 객체의 경우, 다른 변수에 할당할 경우, 해당 값의 주소값을 할당한다. (mutate가 된다)

```javascript
// By Reference 1
var objA = { index: 5 };
var objB = objA;

objB.index = 3; // mutate 진행

console.log(objA.index); // 3을 반환
console.log(objA.index); // 3을 반환

// By Reference 2 (매개변수로 전달 시)
function changeIndex(obj) {
  obj.index = 10;
}

changeIndex(objA);

console.log(objA.index); // 10을 반환
console.log(objA.index); // 10을 반환
```

물론, 아예 할당 연산자를 사용할 경우 새로운 메모리 공간을 할당 받는다

```javascript
objB = { newIndex: 3 };

// 이제 objB를 변경하여도, objA에 연결되지 않는다.
```

- 결론: 모든 원시타입은 전부 By Value로 복제된 값이 할당되고, 객체타입은 전부 By Reference로 주소값이 할당된다.

## Objects, Functions, and 'this'

```javascript
function a() {
  console.log(this);
}

var b = function() {
  console.log(this);
};

a(); // window가 console에 나타난다.
b(); // window가 console에 나타난다.

var c = {
  name: 'This Object',
  log: function() {
    this.name = 'hello object';
    console.log(this);

    /**
     * 약간의 버그로 보여지는 부분. 이렇게 하면, 여기서 실행 컨텍스트가 window로 향하게 된다.
     * 이런 문제 때문에, self를 놓고, 안에서 self를 쓰도록 하도록 되었다.
     */
    var setName = function(newname) {
      this.name = newname;
      /**
       * self.name 으로 들어가면, 우선, 이 항목에서 self를 확인해보고, 없기 때문에 한 단계 위의 Execution Stack에서 self를 찾는다.
       */
    };

    setName('Updated Object Name');
    console.log(this);
  }
};

c.log(); // 해당 Object가 console에 나타난다.
```

## arguments and spread

- arguements: 원 뜻은 함수로 전달된 매개변수들. Execution Context 생성시에 arguments 도 같이 생성된다. 이 내부에는 함수로 전달된 매개변수들이 전부 포함된다.
- Javascript에서는 arguments를 사용할 경우, 전달된 매개변수들이 행렬 형식으로 전달된다. (묘하게 다르다. 행렬은 아니라는 점을 얘기한다)
- ...spread가 대체할 것이라 이 정도로 하고 넘어간다

## Immediately Invoked Function Expressions (IIFE)

## Understanding Closures

```javascript
function greet(whattosay) {
  return function(name) {
    console.log(whattosay + ' ' + name);
  };
}

var sayHi = greet('Hi');
sayHi('immigration9');
```

여기서 물음표가 생길점은, 분명, `whattosay` 항목은 함수 실행 후 사라졌을 텐데, 어떻게 `sayHi`를 호출하였을 때, `whattosay`를 인지하고 있는 것일까?

- 여기서 Closure가 등장한다.
  기존의 Execution Stack이 종료되면, 안에 있는 변수들은 같이 GC에 의해 사라졌어야 하나, Closure로 인하여 `whattosay`가 메모리에 상주하게 된다.
  그렇기 때문에, `sayHi`가 호출되었을 때, `whattosay`를 기존에 상주해 있는 곳에서 찾게 되고, 사용할 수 있게 된다.

```javascript
function buildFunctions() {
  var arr = [];

  for (var i = 0; i < 3; i++) {
    arr.push(function() {
      console.log(i);
    });
  }

  return arr;
}

var fs = buildFunctions();

fs[0](); // 3을 console에 반환
fs[1](); // 3을 console에 반환
fs[2](); // 3을 console에 반환
```

- 위 예제와 똑같다. 각각의 `fs`의 요소들이 실행 될 당시, `i`의 값은 3이 된 상태이다. 만약에, 0, 1, 2를 반환하게 싶다면 아래와 같이 해줘야 한다.

```javascript
function buildFunctionsNew() {
  var arr = [];

  for (var i = 0; i < 3; i++) {
    arr.push(
      (function(j) {
        return function() {
          console.log(j);
        };
      })(i)
    );
  }

  return arr;
}

var fs = buildFunctionsNew();

fs[0](); // 0을 console에 반환
fs[1](); // 1을 console에 반환
fs[2](); // 2을 console에 반환
```

ES6 스펙을 사용한다고 하면, `arr.push()` 위에 `let j = i;`를 선언해 놓고, `j`를 사용하면 훨씬 더 쉽게 해결할 수 있다.

## Closures and Callbacks

```javascript
function sayHiLater() {
  var greeting = 'Hi';

  setTimeout(function() {
    console.log(greeting);
  }, 3000);
}

sayHiLater();
```

- `setTimeout`과 같은 Callback에 해당하는 함수를 사용한 적이 있다면, 전부 Closure의 영향을 받았음을 알 수 있다. `sayHiLater`가 종료된 후에 3000 밀리초 후에 `setTimeout`에 있는 함수가 호출되고, 그 안에서 `sayHiLater` 안에 있는 `greeting`을 사용할 수 있게 되기 때문이다.

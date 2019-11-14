---
layout: post
title: 'Javascript OOP'
date: 2019-06-13 17:00:00 +09:00
categories: 'javascript'
published: false
---

## Jvascript 객체지향 프로그래밍

닷노테이션

객체 접근 - 프로퍼티 삭제

```javascript
var obj2 = {
  name: 'Tom'
};

// null 값을 대입해도 프로퍼티는 지워진게 아니다.
obj2.name = null;
'name' in obj2 === true;

// delete으로 완전히 삭제
delete obj2.name; //정상적으로 지워지면 true 리턴

obj2.name === undefined;
'name' in obj2 === false;
```

### Object.defineProperty

- 닷노테이션으로 객체 생성시, writable / enumerable / configurable은 모두 default로 true로 생성됨

```javascript
var obj = {};

Object.defineProperty(obj, 'name', {
  value: 'awerff',
  configurable: false, // Object.freeze가 있지
  writable: true,
  enumerable: true
});

Object.defineProperty(obj, 'age', {
  value: 29,
  configurable: true, // Object.freeze가 있지
  writable: false,
  enumerable: true
});

Object.defineProperty(obj, 'id', {
  value: 12,
  configurable: true, // Object.freeze가 있지
  writable: false,
  enumerable: false
});
```

### getter / setter

- getter, setter는 계산형 프로퍼티 만들거나 프로퍼티의 데이터를 외부에 간접적으로 노출할 때 사용한다.
- 함수형태로 정의한다. 그래서 setter에서 동명의 프로퍼티에 값을 할당하면 무한 루프 생김

```javascript
var obj = {};
var ageNum = 0;

Object.defineProperty(obj, 'age', {
  set: function(value) {
    ageNum = value;
  },
  get: function() {
    return ageNum + '살';
  }
});

Object.defineProperty(obj, 'name', {
  set: function(value) {
    this.newName = value;
  },
  get: function() {
    return this.newName;
  }
});

obj.age = 10;
obj.name = 'hello, world!';

console.log(obj.age); // 10 '살'
console.log(ageNum);

console.log(obj.name);
```

```javascript
var todo = {
  complete: false,
  title: '',
  getTitle: function() {},
  setComplete: function(complete) {}
};

Object.defineProperty(todo, 'state', {
  set: function(value) {
    this.complete = value;
  },
  get: function() {
    if (this.complete === true) {
      return '완료';
    } else {
      return '미완료';
    }
  }
});
```

## this

- this는 메서드가 정의될때가 아닌 실행될 때 결정 (함수를 실행하기 전에 엔진에서 준비해줌, 실행 컨텍스트에 포함)

* 메서드안에서의 this(2)

- 같은 함수라도 다른 객체와 연결되어 실행되면 this가 변경됨

```
nottodo.getTitle = todo.getTitle;
nottodo.getTitle();
```

- nottodo안에 getTitle 메서드가 필요로하는 프로퍼티가 없다면 에러발생

```javascript
var todo = {
  complete: false,
  title: '',
  getTitle: function() {},
  setComplete: function(complete) {}
};

Object.defineProperty(todo, 'getTitle', {
  get: function() {
    return this.title;
  }
});

Object.defineProperty(todo, 'setComplete', {
  set: function(value) {
    this.complete = value;
  }
});
```

## this 변조

- 자바스크립트의 함수는 Function 타입의 인스턴스 so 인스턴스 메서드 사용 가능(Callable Object)
- 함수를 실행할때 this를 임의로 정해서 실행 -> call(), apply()
- 함수의 this가 변경되지 않도록 고정 -> bind()

```javascript
var todo = {
  complete: false,
  title: '자바스크립트 공부하기'
};

function setComplete(complete) {
  this.complete = complete;
}

/**
 * this가 todo로 할당됨
 * 두 번째 인자부터 파라미터 전달
 */
setComplete.call(todo, true);
```

- `bind` example

```javascript
var todo = {
  complete: false,
  title: '자바스크립트 공부하기'
};

function setComplete(complete) {
  this.complete = complete;
}

var setCompleteOfTodo = setComplete.bind(todo);

setCompleteOfTodo(true);

todo.complete === true;
```

```javascript
var todo = {
  complete: false,
  title: '자바스크립트 공부하기'
};

function printTodo() {
  var indicator;
  this.complete ? (indicator = '[완료] ') : (indicator = '[미완료] ');
  console.log(indicator + this.title);
}

/** Call 이용하여 */
printTodo.call(todo);

/** Bind 이용하여 */
todo.complete = true;
var printTodoTodo = printTodo.bind(todo);
printTodoTodo();
```

- `call` 메소드를 이용해 `bind` 함수를 직접 구현

```javascript
var person = {
  name: 'Steve Jobs'
};

function printPersonName(description, korean) {
  console.log(this.name + ' / ' + description + ' / ' + korean);
}

function myBind(func, context, ...args) {
  return function(...args) {
    func.apply(context, args);
  };
}

var printPersonNameBinded = myBind(printPersonName, person);
printPersonNameBinded('good man', '스티브');
```

## 객체 생성 패턴: 팩터리 함수

```javascript
function createTodo(title) {
  var newTodo = {};

  newTodo.title = title;
  newTodo.complete = false;
  newTodo.getTitle = function() {
    return this.title;
  };
  newTodo.setComplete = function(complete) {
    this.complete = complete;
  };
  return todo;
}

var todo1 = createTodo('출근하기');
var todo2 = createTodo('퇴근하기');

todo1.getTitle() === '출근하기';
todo2.getTitle() === '퇴근하기';
```

## 생성자

- 자바스크립트에서 클래스의 역할
- 일반 함수를 new 키워드와 함께 실행하면 생성자로 동작

```javascript
function Person(name) {
  this.name = name;
}
var person = new Person('Steve Jobs');
```

## 객체 타입 비교

- 생성자 통해 객체 생성시 생성자 타입 가지게 됨
- `constructor`와 `instanceof` 연산자를 이용할 수 있음.

```javascript
function Human() {}

var man = new Human();

typeof man; // 'object'
man.constructor === Human; // true
man instanceof Human; // true
```

### 팩토리 패턴을 생성자로

```javascript
function Todo(title) {
  this.title = title;
  this.complete = false;
  this.getTitle = function() {
    return this.title;
  };
  this.setComplete = function(complete) {
    this.complete = complete;
  };
}

var todo = new Todo('hello, world!');
```

## Prototype

- 위에 나온 것 중에 `getTitle` / `setComplete` 들은 생성자로 매번 새로운걸 만들 필요가 없음. 메모리 낭비.

```javascript
function Todo(title) {
  this.title = title;
  this.complete = false;
}
Todo.prototype.getTitle = function() {
  return this.title;
};
Todo.prototype.setComplete = function(complete) {
  this.complete = complete;
};
```

```javascript
/** 물론 이렇게 쓰면 안됨 */
var parent = {
  pMethod: () => {}
};
var child = {
  __proto__: parent,
  cMethod: function() {}
};
var grandChild = {
  __proto__: child,
  gcMethod: function() {}
};

/** 할거면 이런 방식으로 */
var parent = {
  pMethod: function() {}
};
var child = Object.create(parent); // child.__proto__ --> parent
child.cMethod = function() {};

/** 실습 */
var parent = {
  pMethod: () => {}
};
var child = Object.create(parent);
child.cMethod = () => {};

var grandChild = Object.create(child);
grandChild.gcMethod = () => {};
```

## 생성자 기반 프로토타입 체인 상속

```javascript
function Parent() {}
Parent.prototype.pMethod = function() {};

function Child() {}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child; // Object.create가 덮어씌운 것을 수정 해야함
Child.prototype.cMethod = function() {};

var p2 = new Child(); // p2.__proto__ === Child.prototype

function GrandChild() {}
GrandChild.prototype = Object.create(Child.prototype);
GrandChild.prototype.constructor = GrandChild;
GrandChild.prototype.gcMethod = function() {};

var p3 = new GrandChild();

Parent.prototype.newMethod = function() {};
```

### Basic 한 상속의 형식

```javascript
function Parent(name) {
  this.name = name;
}
Parent.prototype.pMethod = function() {};

function Child(name) {
  Parent.call(this, name);

  this.someChildNeeds = 'asdfaasdfasdf';
}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child; // Object.create가 덮어씌운 것을 수정 해야함
Child.prototype.cMethod = function() {};
```

### 메소드 오버라이드

- 프로토타입 체인에선 실행 객체와 가장 가까운 메소드가 우선 순위를 갖는다. (Child면 Child에 있는 것)

예제

```javascript
function Todo(title) {
  this.title = title;
  this.complete = false;
}

Todo.prototype.getDisplayText = function() {
  var indicator = this.complete ? '[완료]' : '[미완료]';
  return indicator + ' ' + this.title;
};
Todo.prototype.print = function() {
  console.log(this.getDisplayText());
};

function OfficeTodo(title) {
  Todo.call(this, title);
}
function HomeTodo(title) {
  Todo.call(this, title);
}

OfficeTodo.prototype = Object.create(Todo.prototype);
/**
 * Object.create returns the object below
 * OfficeTodo.prototype = {
 *   __proto__: Todo.prototype
 * }
 }
 */
/**
 * 이걸 안해주면, Object.create 가 새로운 객체를 만들어주는 것이기 때문에
 * constructor == Object 가 된다.
 */
// OfficeTodo.prototype.constructor = OfficeTodo;
ObjectTodo.prototype.constructor === Object;

HomeTodo.prototype = Object.create(Todo.prototype);
HomeTodo.prototype.constructor = HomeTodo;

OfficeTodo.prototype.getDisplayText = function() {
  return '-Office-' + Todo.prototype.getDisplayText.call(this);
};
HomeTodo.prototype.getDisplayText = function() {
  return '-Home-' + Todo.prototype.getDisplayText.call(this);
};
```

김성호 FE개발랩 `shiren@nhn.com`

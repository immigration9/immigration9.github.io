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
감시자 패턴은 감시자 역할을 하는 `Observable` 객체와 감시 대상인 `Observer` 객체로 이루어진다. 위에 언급된 Subscriber/Publisher Pattern은 비교를 위해 언급하였지만, 패턴 자체는 다르다. 

* 아래 코드는 모두 ES6로 작성된 뒤, Webpack으로 번들링되었다. 
* 코드는 다음 Repo에서 확인 가능: https://github.com/immigration9/javascript-patterns

## Observer Pattern 예제 1
아래 예제는 input text field에 들어온 값에 대하여 여러개의 감시 대상에 동시에 알림을 보내는 예제이다.

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

## Observer Pattern 예제 2
아래 예제는 잡지 구독의 형태로, 감시자(Observable, Publisher)인 잡지사와 감시 대상(Observer, Subscriber)로 이루어진다. 

```javascript
// Publisher.js
class Publisher {
  subscribers = { any: [] }

  /**
   * `subscribe` method takes function & type to register.
   * `unsubscribe` method takes functon & type to deregister
   */
  subscribe = (fn, type) => {
    type = type || 'any';
    /**
     * Initialization of subscriber field.
     */
    if (typeof this.subscribers[type] === 'undefined') {
      this.subscribers[type] = [];
    }
    this.subscribers[type].push(fn);
  }
  unsubscribe = (fn, type) => {
    this.visitSubscribers('unsubscribe', fn, type);
  }
  /**
   * `publish` method invokes the `visitSubscribers` method.
   * `visitSubscriber` method determines the action, and if it is set to 'publish', the trigger the functions registered as observers.
   */
  publish = (publication, type) => {
    this.visitSubscribers('publish', publication, type);
  }
  visitSubscribers = (action, arg, type) => {
    let pubType = type || 'any';
    let subscribers = this.subscribers[pubType];
    let max = subscribers.length;

    for (let i = 0; i < max; i++ ) {
      if (action === 'publish') {
        subscribers[i](arg);
      } else {
        if (subscribers[i] === arg) {
          subscribers.splice(i, 1);
        }
      }
    }
  }
}

export default Publisher;
```

```javascript
// Paper.js
import Publisher from './Publisher';

/**
 * `Paper` is a singleton class which exists throughout the application.
 * When either `daily` or `monthly` function is invoked, it notifies the `Publisher`.
 * by invoking the publish function, `Publisher` notifies the observers within the scope.
 */
class Paper extends Publisher {
  constructor() {
    super();
  }
  /**
   * Because the type in `daily` function was not defined,
   * The default would be `any`, which means the users registered to the default category,
   * they will be notified.
   */
  daily = () => {
    this.publish("[Washington Post]");
  }
  weekly = () => {
    this.publish("[Hustler]", "weekly");
  }
  monthly = () => {
    this.publish("[Horse & Hounds]", "monthly");
  }
}

let paper = new Paper();
export default paper;
```

```javascript
// Subscriber.js
/**
 * `Subscriber` is a class which entails two things.
 * the name of the subscriber, and a method which later on will be registered to the Observable.
 * In this example, `readNews` function will be registered
 */
class Subscriber {
  constructor(name) {
    this.name = name;
  }
  /**
   * `readNews`: function to be registered when the Observable is ready
   */
  readNews = (news) => {
    console.log(`${this.name}: I was reading ${news}`);
  }
}

export default Subscriber
```

```javascript
// index.js
import paper from './Paper';
import Subscriber from './Subscriber'

let joe = new Subscriber('Joe');
let jane = new Subscriber('Jane');
let john = new Subscriber('John');

/**
 * Registers the both users' `readNews` method.
 */
paper.subscribe(joe.readNews);
paper.subscribe(jane.readNews, 'monthly');
paper.subscribe(john.readNews, 'weekly');

/**
 * Invoke functions within `paper` object.
 * Only registered Observers react to this situation.
 */
paper.daily();
paper.daily();
paper.daily();
paper.daily();
paper.weekly();
paper.monthly();
```

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>Observer Pattern</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <div>
    # Observer Pattern example.<br/>
    This example was created to help understanding of observer pattern.<br/>
    Codes were written in ES6 and was compiled using Webpack.<br/>
    <br/>
    Original reference from the book `Javascript Patterns` by Stoyan Stefanov.<br/><br/>
  </div>
  <div id="results"></div>
</body>
<script src="observer.js"></script>
</html>
```

[reference-01]:https://book.naver.com/bookdb/book_detail.nhn?bid=6763510
[reference-02]:https://pawelgrzybek.com/the-observer-pattern-in-javascript-explained/
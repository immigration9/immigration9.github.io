---
layout: post
title: "(번역) React를 사용하기 위해 알아야 하는 JavaScript (원제: Javascript to Know for React)"
date: "2020-07-26 17:00:00 +09:00"
categories: "react,javascript"
published: true
---

![Photo by Benjamin Voros](https://kentcdodds.com/static/3d283a7e351a639fbc71b810eee15eab/d6099/banner.webp)
Photo By Benjamin Voros

## React를 배우고 사용하기 위해 친숙해져야 하는 JavaScript 기능들

원본: [JavaScript to Know for React](https://kentcdodds.com/blog/javascript-to-know-for-react)

그 동안 사용했던 다른 프레임워크들과 비교하였을 때 내가 React에 대해 가장 좋아하는 점은 JavaScript에 굉장히 많이 노출되어 있다는 점이다. 해당 프레임워크에 귀속되는 템플릿 작성법이 따로 없고 (JSX가 JavaScript로 컴파일을 해준다), React Hook 등장 이후로 컴포넌트 API는 사용하기 한 단계 더 쉬워졌으며, 프레임워크 자체가 해결하고자 했던 core UI 밖으로는 굉장히 적은 양의 추상화를 제공하기 때문이다.

그렇기 때문에, React를 이용하여 애플리케이션을 제작할 때 효율적이기 위해서는 JavaScript의 기능들을 배우는 것이 권장된다. 여기 소개되는 기능들은 React를 최대한 효율적으로 이용하기 위해 공부하는데 시간을 좀 들이길 바라는 JavaScript 기능들이다.

### Template Literals

템플릿 리터럴은 특별한 힘을 가진 string이라 할 수 있다:

```javascript
const greeting = 'Hello';
const subject = 'World';

console.log(`${greeting} ${subject}!`); // Hello World!
// 위 아래 console.log는 같은 결과를 도출해낸다.
console.log(greeting + ' ' + subject + '!');

// in React:
function Box({ className, ...props }) {
  return <div className={`box ${className}`} {...props} />;
}
```

[MDN: Template Literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

### Shorthand property names

이건 너무 효과적이라 이젠 거의 일상이 되었다.

```javascript
const a = 'hello';
const b = 42;
const c = { d: [true, false] };

console.log({ a, b, c });
// 위 아래 console.log는 같은 결과를 도출해낸다.
console.log({ a: a, b: b, c: c });

// in React:
function Counter({ initialCount, step }) {
  const [count, setCount] = useCounter({ initialCount, step });
  return <button onClick={setCount}>{count}</button>;
}
```

[MDN: Object initializer New notations in ECMAScript 2015](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_2015)

### 화살표 함수: Arrow functions

화살표 함수는 JavaScript 함수를 사용하는 하나의 방법 중 하나지만, 약간의 의미상 차이는 존재한다. 다행히도 React의 세상에서는, Hook으로 프로젝트를 작성한다면 `this`에 대해 고민을 하지 않아도 된다. 화살표 함수는 간결한 익명 함수와 암시적 반환(implicit return)을 가능하게 해주기 때문에, 앞으로 굉장히 많이 사용하게 될 것이다.

```javascript
const getFive = () => 5;
const addFive = (a) => a + 5;
const divide = (a, b) => a / b;
// 위와 아래는 같은 의미를 갖는다
function getFive() {
  return 5;
}
function addFive(a) {
  return a + 5;
}
function divide(a, b) {
  return a / b;
}
// in React:
function TeddyBearList({ teddyBears }) {
  return (
    <ul>
      {teddyBears.map((
        teddyBear // 암시적 반환 (return을 사용하지 않았음)
      ) => (
        <li key={teddyBear.id}>
          <span>{teddyBear.name}</span>
        </li>
      ))}
    </ul>
  );
}
```

> 위의 예제에서 열고 닫는 소괄호 `(`를 주목해보자. JSX를 작성할 때 화살표 함수의 암시적 반환을 사용하는 일반적인 방법이다.

[MDN: Arrow Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

### 비구조화 할당: Destructuring

비구조화 할당은 내가 가장 좋아하는 JavaScript 기능이다. 나는 객체와 배열들을 언제나 비구조화 할당한다 (당신도 `useState`를 사용한다면 [이렇게](https://kentcdodds.com/blog/react-hooks-array-destructuring-fundamentals) 사용하고 있을 것이다).

```javascript
// const obj = {x: 3.6, y: 7.8}
// makeCalculation(obj)
function makeCalculation({ x, y: d, z = 4 }) {
  return Math.floor((x + d + z) / 3);
}

// 위와 아래는 같은 기능을 수행한다
function makeCalculation(obj) {
  const { x, y: d, z = 4 } = obj;
  return Math.floor((x + d + z) / 3);
}

// 아래 함수도 위 두 함수들과 같은 의미를 갖는다
function makeCalculation(obj) {
  const x = obj.x;
  const d = obj.y;
  const z = obj.z === undefined ? 4 : obj.z;
  return Math.floor((x + d + z) / 3);
}
// in React:
function UserGitHubImg({ username = 'ghost', ...props }) {
  return <img src={`https://github.com/${username}.png`} {...props} />;
}
```

[MDN: Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

> 꼭 MDN에서 저 내용을 읽어보도록. 분명 새롭게 배우는 것이 있을 것이다. 다 보고난 뒤에는, 아래 코드를 비구조화 할당을 이용하여 리팩토링 해보자.

```javascript
function nestedArrayAndObject() {
  // 아래 코드를 비구조화 할당을 이용하여 한 줄로 바꿔보자
  const info = {
    title: 'Once Upon a Time',
    protagonist: {
      name: 'Emma Swan',
      enemies: [
        { name: 'Regina Mills', title: 'Evil Queen' },
        { name: 'Cora Mills', title: 'Queen of Hearts' },
        { name: 'Peter Pan', title: `The boy who wouldn't grow up` },
        { name: 'Zelena', title: 'The Wicked Witch' },
      ],
    },
  };
  // const {} = info // <-- 아래 `const`가 들어간 코드들을 이 한 줄로 압축해보자.
  const title = info.title;
  const protagonistName = info.protagonist.name;
  const enemy = info.protagonist.enemies[3];
  const enemyTitle = enemy.title;
  const enemyName = enemy.name;
  return `${enemyName} (${enemyTitle}) is an enemy to ${protagonistName} in "${title}"`;
}
```

### 기본 매개변수: Parameter defaults

내가 항상 사용하는 또 하나의 기능이다. 함수 매개변수의 기본값을 선언적으로 표현하는 굉장히 강력한 방법이다.

```javascript
// add(1)
// add(1, 2)
function add(a, b = 0) {
  return a + b;
}
// 위와 아래 코드는 같다
const add = (a, b = 0) => a + b;

// 기본 매개변수가 없을 경우 아래와 같이 쓰여질 것이다
function add(a, b) {
  b = b === undefined ? 0 : b;
  return a + b;
}

// in React:
function useLocalStorageState({
  key,
  initialValue,
  serialize = (v) => v,
  deserialize = (v) => v,
}) {
  const [state, setState] = React.useState(
    () => deserialize(window.localStorage.getItem(key)) || initialValue
  );
  const serializedState = serialize(state);
  React.useEffect(() => {
    window.localStorage.setItem(key, serializedState);
  }, [key, serializedState]);
  return [state, setState];
}
```

[MDN: Default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)

### Rest/Spread

`...` 문법은 값들의 모음을 가지고 동작하는 문법 "모음집"이라 할 수 있다. 나는 언제나 이걸 사용하고 어떻게, 그리고 어디서 이게 사용될 수 있는지 공부하는 것을 강력히 추천한다. 실제로 문맥에 따라 다른 의미를 갖기 때문에, 사용되는 뉘양스를 공부하는 것이 도움을 줄 것이다.

```javascript
const arr = [5, 6, 8, 4, 9];
Math.max(...arr);
// 위 아래 Math.max는 같은 결과를 도출해낸다.
Math.max.apply(null, arr);
const obj1 = {
  a: 'a from obj1',
  b: 'b from obj1',
  c: 'c from obj1',
  d: {
    e: 'e from obj1',
    f: 'f from obj1',
  },
};
const obj2 = {
  b: 'b from obj2',
  c: 'c from obj2',
  d: {
    g: 'g from obj2',
    h: 'g from obj2',
  },
};
console.log({ ...obj1, ...obj2 });
// 위 아래 console.log는 같은 결과를 도출해낸다.
console.log(Object.assign({}, obj1, obj2));

function add(first, ...rest) {
  return rest.reduce((sum, next) => sum + next, first);
}
// 위 아래 add 함수는 같은 결과를 반환한다.
function add() {
  const first = arguments[0];
  const rest = Array.from(arguments).slice(1);
  return rest.reduce((sum, next) => sum + next, first);
}

// in React:
function Box({ className, ...restOfTheProps }) {
  const defaultProps = {
    className: `box ${className}`,
    children: 'Empty box',
  };
  return <div {...defaultProps} {...restOfTheProps} />;
}
```

[MDN: Spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)

[MDN: Rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)

### ESModules

최신 툴을 사용하여 애플리케이션을 개발한다면, 높은 확률로 모듈을 지원할 것이다. 모듈에 있는 코드가 어떻게 돌아가는지에 대해 알아놓는 것이 좋은 아이디어인게, 굉장히 작은 사이즈의 애플리케이션도 코드 재사용과 정리를 위해 모듈을 필요로할 것이기 때문이다.

```javascript
export default function add(a, b) {
  return a + b;
}
/*
 * import add from './add'
 * console.assert(add(3, 2) === 5)
 */
export const foo = 'bar';
/*
 * import {foo} from './foo'
 * console.assert(foo === 'bar')
 */
export function subtract(a, b) {
  return a - b;
}
export const now = new Date();
/*
 * import {subtract, now} from './stuff'
 * console.assert(subtract(4, 2) === 2)
 * console.assert(now instanceof Date)
 */

// dynamic imports
import('./some-module').then(
  (allModuleExports) => {
    // allModuleExports 객체는 import * as allModuleExports from './some-module'
    // 로 했을 때와 같은 객체를 반환한다.
    // 유일한 차이점은 이 내용이 비동기적으로 로드되기 때문에
    // 몇몇 케이스들에 있어 성능적 이점을 갖는다.
  },
  (error) => {
    // 에러 핸들링
    // 로딩이나 모듈 실행에 있어서 에러가 있을 경우 발생한다.
  }
);

// in React:
import React, { Suspense, Fragment } from 'react';
// 동적으로 리액트 컴포넌트 로딩하기
const BigComponent = React.lazy(() => import('./big-component'));
// 위 코드가 동작하기 위해 big-component.js는 "export default BigComponent"를 해줘야 한다
```

[MDN: import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)

[MDN: export](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)

> 또 다른 리소스로, 이 문법에 대한 발표가 있다. [유튜브 링크](https://www.youtube.com/watch?v=kTlcu16rSLc&list=PLV5CVI1eNcJgNqzNwcs4UKrlJdhfDjshf)

### 삼항 조건 연산자: Ternaries

삼항 조건 연산자는 정말 최고다. 특히 JSX 사용시에 선언적인 측면은 아름답다.

```javascript
const message = bottle.fullOfSoda
  ? 'The bottle has soda!'
  : 'The bottle may not have soda :-(';

// 아래 코드는 위 코드와 같다
let message;
if (bottle.fullOfSoda) {
  message = 'The bottle has soda!';
} else {
  message = 'The bottle may not have soda :-(';
}

// in React:
function TeddyBearList({ teddyBears }) {
  return (
    <React.Fragment>
      {/* JSX에서 사용하는 삼항 연산자 */}
      {teddyBears.length ? (
        <ul>
          {teddyBears.map((teddyBear) => (
            <li key={teddyBear.id}>
              <span>{teddyBear.name}</span>
            </li>
          ))}
        </ul>
      ) : (
        <div>There are no teddy bears. The sadness.</div>
      )}
    </React.Fragment>
  );
}
```

> 삼항 조건 연산자를 소개할 때 [prettier](https://prettier.io/) 이전 시대에 직접 코드 청소를 해야했던 사람들로부터 이 문법에 대한 자동반사적 거부감을 불러올 수 있다는 것을 안다. prettier를 아직 사용해보지 않았다면, 꼭 해보길 바란다. Prettier는 삼항 조건 연산자의 가독성을 굉장히 높혀준다.

[MDN: Conditional (ternary) operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)

### Array Methods

배열은 환상적이고 나는 배열 메소드를 언제나 사용한다! 아래에 있는 메소드들은 내가 가장 자주 사용하는 것들이다.

- find
- some
- every
- includes
- map
- filter
- reduce

```javascript
const dogs = [
  {
    id: 'dog-1',
    name: 'Poodle',
    temperament: [
      'Intelligent',
      'Active',
      'Alert',
      'Faithful',
      'Trainable',
      'Instinctual',
    ],
  },
  {
    id: 'dog-2',
    name: 'Bernese Mountain Dog',
    temperament: ['Affectionate', 'Intelligent', 'Loyal', 'Faithful'],
  },
  {
    id: 'dog-3',
    name: 'Labrador Retriever',
    temperament: [
      'Intelligent',
      'Even Tempered',
      'Kind',
      'Agile',
      'Outgoing',
      'Trusting',
      'Gentle',
    ],
  },
];

dogs.find((dog) => dog.name === 'Bernese Mountain Dog');
// {id: 'dog-2', name: 'Bernese Mountain Dog', ...etc}

dogs.some((dog) => dog.temperament.includes('Aggressive'));
// false

dogs.some((dog) => dog.temperament.includes('Trusting'));
// true

dogs.every((dog) => dog.temperament.includes('Trusting'));
// false

dogs.every((dog) => dog.temperament.includes('Intelligent'));
// true

dogs.map((dog) => dog.name);
// ['Poodle', 'Bernese Mountain Dog', 'Labrador Retriever']

dogs.filter((dog) => dog.temperament.includes('Faithful'));
// [{id: 'dog-1', ..etc}, {id: 'dog-2', ...etc}]

dogs.reduce((allTemperaments, dog) => {
  return [...allTemperaments, ...dog.temperaments];
}, []);
// [ 'Intelligent', 'Active', 'Alert', ...etc ]

// in React:
function RepositoryList({ repositories, owner }) {
  return (
    <ul>
      {repositories
        .filter((repo) => repo.owner === owner)
        .map((repo) => (
          <li key={repo.id}>{repo.name}</li>
        ))}
    </ul>
  );
}
```

[MDN: Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

### null류 병합 연산자: Nullish coalescing operator

값이 `null`이나 `undefined`면, 기본값을 보여주도록 할 수 있다.

```javascript
// 우리가 자주 사용하던 방식:
x = x || 'some default';

// 하지만 위 방식은, '0'이나 'false' 정상적인 값으로 인식되는 경우에 문제를 발생시켰다.
// 그렇기 때문에 이 방식을 사용하기 위해:
add(null, 3);
// 이전에는 이렇게 별도로 작성을 해줬어야 했다.
function add(a, b) {
  a = a == null ? 0 : a;
  b = b == null ? 0 : b;
  return a + b;
}

// 이제는 이렇게 해주면 된다.
function add(a, b) {
  a = a ?? 0;
  b = b ?? 0;
  return a + b;
}

// in React:
function DisplayContactName({ contact }) {
  return <div>{contact.name ?? 'Unknown'}</div>;
}
```

[MDN: Nullish coalescing operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator)

### Optional Chaining

"Elvis 연산자"로도 알려졌으며, 존재하거나 존재하지 않는 요소와 함수들을 안전하게 접근할 수 있도록 해준다. Optional Chaining 이전에는 해당 항목의 존재 유무를 판단하기 위해 편법적인 방법을 사용했어야 했다.

```javascript
// Optional Chaining 이전에 사용해야 했던 방법
const streetName = user && user.address && user.address.street.name;

// 이제 사용할 수 있는 방법
const streetName = user?.address?.street?.name;

// 아래 코드는 options 항목이 undefined여도 실행할 수 있도록 해준다 (물론, undefined가 반환될 것이다. 존재하지 않기 때문에)
const onSuccess = options?.onSuccess;

// 아래 코드는 onSuccess가 undefined여도 동작할 수 있도록 해준다 (이 경우, 어떤 함수도 호출되지 않을 것이다.)
onSuccess?.({ data: 'yay' });

// 위 두 케이스를 하나로 합칠 수도 있다.
options?.onSuccess?.({ data: 'yay' });

// 만약에 options가 존재하면 onSuccess도 존재한다는 100% 확신이 있다면
// 호출 전에 추가적인 ?. 를 붙히지 않아도 된다.
// 오로지 왼쪽에 있는 항목이 존재하지 않을 수도 있는 경우에만 ?. 을 사용하도록 하자
options?.onSuccess({ data: 'yay' });

// in React:
function UserProfile({ user }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <strong>{user.bio?.slice(0, 50)}...</strong>
    </div>
  );
}
```

이 기능에 대해 주의를 하자면, 코드 내에서 `?.`을 굉장히 많이 사용한다면, 값의 원류로 돌아가 값을 일정하게 반환하도록 처리하는 방법을 생각해야한다.

[MDN: Optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)

### Promises and async/await

이건 굉장히 큰 주제이고 잘 사용하기 위해서는 어느정도의 연습과 시간을 필요로 한다. Promise는 JavaScript 생태계 뿐만 아니라, 그 생태계 속에 뿌리깊게 내린 React에도 어디에나 존재한다. (React 스스로도 내부적으로 Promise를 사용한다).

Promise는 비동기 코드를 관리할 수 있도록 도와주고, 많은 써드파티 라이브러리나 DOM API들에서 반환된다. Async/await 문법은 Promise를 다루는 특별한 문법이다. 그 둘은 같이 사용된다.

```javascript
function promises() {
  const successfulPromise = timeout(100).then((result) => `success: ${result}`);
  const failingPromise = timeout(200, true).then(null, (error) =>
    Promise.reject(`failure: ${error}`)
  );
  const recoveredPromise = timeout(300, true).then(null, (error) =>
    Promise.resolve(`failed and recovered: ${error}`)
  );
  successfulPromise.then(log, logError);
  failingPromise.then(log, logError);
  recoveredPromise.then(log, logError);
}

function asyncAwaits() {
  async function successfulAsyncAwait() {
    const result = await timeout(100);
    return `success: ${result}`;
  }
  async function failedAsyncAwait() {
    const result = await timeout(200, true);
    return `failed: ${result}`;
  }
  async function recoveredAsyncAwait() {
    let result;
    try {
      result = await timeout(300, true);
      return `failed: ${result}`; // this would not be executed
    } catch (error) {
      return `failed and recovered: ${error}`;
    }
  }
  successfulAsyncAwait().then(log, logError);
  failedAsyncAwait().then(log, logError);
  recoveredAsyncAwait().then(log, logError);
}

function log(...args) {
  console.log(...args);
}

function logError(...args) {
  console.error(...args);
}

// 이 코드는 현 예제에서 사용되는 모든 비동기 코드의 모선이다
function timeout(duration = 0, shouldReject = false) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (shouldReject) {
        reject(`rejected after ${duration}ms`);
      } else {
        resolve(`resolved after ${duration}ms`);
      }
    }, duration);
  });
}

// in React:
function GetGreetingForSubject({ subject }) {
  const [isLoading, setIsLoading] = React.useState(false);
  const [error, setError] = React.useState(null);
  const [greeting, setGreeting] = React.useState(null);
  React.useEffect(() => {
    async function fetchGreeting() {
      try {
        const response = await window.fetch('https://example.com/api/greeting');
        const data = await response.json();
        setGreeting(data.greeting);
      } catch (error) {
        setError(error);
      } finally {
        setIsLoading(false);
      }
    }
    setIsLoading(true);
    fetchGreeting();
  }, []);
  return isLoading ? (
    'loading...'
  ) : error ? (
    'ERROR!'
  ) : greeting ? (
    <div>
      {greeting} {subject}
    </div>
  ) : null;
}
```

[MDN: Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

[MDN: async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)

[MDN: await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)

### Conclusion

물론 React 애플리케이션을 개발할 때 정말 유용한 JavaScript 기능들이 정말 많다. 위에 언급된 내용들은 주로 내가 계속해서 사용하게 되는 기능들입니다. 공부하는데 도움이 되었기를 바랍니다.

위 내용들을 더 깊게 공부해보고 싶다면, 제가 PayPal에서 일하는 동안 촬영한 JavaScript workshop이 도움이 될겁니다: [ES6 and Beyond Workshop at PayPal](https://www.youtube.com/playlist?list=PLV5CVI1eNcJgUA2ziIML3-7sMbS7utie5)

항상 건승하시길!

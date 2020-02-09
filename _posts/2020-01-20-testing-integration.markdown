---
layout: post
title: 'Jest + React Testing Library 도입'
date: 2020-01-20 22:15:00 +09:00
categories: 'react,testing'
published: true
---

| 작성한 테스트 케이스가 실제 소프트웨어를 잘 반영할 수록 코드에 대한 더 많은 자신감을 가질 수 있다. - Kent. C. Dodds

## 시나리오 구성: 무얼 테스팅할 것인가?

1. Router 변화에 대한 Testing

2. 내부 State 변경에 대한 Testing

- 무엇을 mock할 것인가? 어떻게 mock할 것인가?

3. ContextAPI에 대한 Testing

Mocking의 목적은 우리가 컨트롤하지 않는 것을 컨트롤 하도록 만드는데 있다

- Call caputre
- Return 값 설정
- implementation 자체를 변경

## spyOn을 사용하여 그리는 시나리오

- toBeCalled
- toBeCalledWith
- toBe

```javascript
```

### 특정 함수의 중복 호출 방지

- toBeCalled를 이용한 함수 호출 횟수 모니터링

### 모의 구현체로 바꾸기 (mock implementation)

모듈 전체를 mock하고 싶을 때 사용하면 된다.

```javascript
/**
 * Basic Calculation example
 * ./calculator.js
 */

export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
export const multiply = (a, b) => a * b;
export const divide = (a, b) => a / b;

export default calculation = (a, b, func) => func(a, b);
```

위와 같은 calculator 모듈 전체를 mock하고 싶다면, 아래와 같이 전체 module을 mock에 넣어주면 된다.

```javascript
jest.mock('./calculator');
```

이렇게 되면, jest는 자동으로 해당 모듈을 mocked module로 변환시키며,
다른 곳에서 해당 모듈을 사용할 때도 jest의 mocked module을 바라보게 된다.

예를들어, 다른 곳에서 `calculator.js` 파일을 사용한다고 가정해보자.

```javascript
/** computer.js */
import { add, subtract } from './calculator';

export const doubleSubtraction = (a, b) =>
  subtract(subtract(a, b), subtract(a, b));
export const doubleAddition = (a, b) => add(add(a, b), add(a, b));
```

```javascript
/** computer.test.js */
import * as computer from '../computer';

jest.mock('../calculator.js');

expect(computer.doubleAddition(1, 2)).toBe(6);
```

이러면 6이 예상되지만 그렇지 않다. 왜냐면 모든 calculator의 항목들은 `jest.fn()` 으로 대체되었기 때문이다.
`jest.mock()`을 실행하면 아래와 같은 결과를 기대할 수 있다.

```javascript
export const add = jest.fn();
export const subtract = jest.fn();
export const multiply = jest.fn();
export const divide = jest.fn();
```

그렇다면 `jest.mock()`은 언제 써야하는 것일까? /\*_ @todo 여기서부터 _/

### API Call만 intercept하기

## jest.mock 을 사용하여 불필요한 모듈 mocking하기

## 중복된 함수들을 utils로 빼서 코드 중복 방지하기

- 항상 사용하는 Router의 경우, 별도의 entry를 만들어서 컴포넌트로 빼둔다.
- 여기서 주의할 점은, render는 react의 render가 아닌, `@testing-library/react`의 render라는 점이다.

```javascript
import { render } from '@testing-library/react';

function renderWithRouter(
  ui,
  {
    route = '/',
    history = createMemoryHistory({ initialEntries: [route] })
  } = {}
) {
  const Wrapper = ({ children }) => (
    <Router history={history}>{children}</Router>
  );

  return {
    ...render(ui, { wrapper: Wrapper }),
    history
  };
}
```

- ContextAPI 사용시 연관되는 Provider들은 모두 묶어서 선언해준다.

```javascript
function renderWithProvider(ui) {
  return <Provider>{ui}</Provider>;
}
```

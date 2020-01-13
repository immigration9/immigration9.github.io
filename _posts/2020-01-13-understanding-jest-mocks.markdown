---
layout: post
title: '(번역) Jest Mock 이해하기 (원제: Understanding Jest Mocks)'
date: 2020-01-13 10:15:00 +09:00
categories: 'react,testing'
published: true
---

본 글은 Jest maintainer Rick Hanlon의 글 Understanding Jest Mocks를 번역한 글입니다.
[Understanding Jest Mocks](https://medium.com/@rickhanlonii/understanding-jest-mocks-f0046c68e53c)

Mocking, 한글로 '속이다'로 번역할 수 있는 본 단어는 의존성 있는 항목들을 개발자가 관리할 수 있는 객체로 바꿈으로써 테스트 대상을 고립시키는 기술을 말한다. (Mocking 단어 자체는 '가짜'로 해석 할 수 있으나, 여기서는 Jest 모듈의 함수로도 칭할 수 있으니 mock을 그대로 사용하겠다). 의존성은 테스트 대상이 의존하는 그 무엇이던 될 수 있으나, 일반적으로 대상이 import 하는 모듈을 의미한다.

자바스크립트의 경우 testdouble, sinon 같은 훌륭한 mocking 라이브러리가 있으며, Jest는 이 기능을 내장하고 있다.

Jest에서의 mocking을 얘기할 때, 일반적으로 우리는 [Mock Function](https://jestjs.io/docs/en/mock-function-api.html)을 가지고 의존성을 교체하는 것을 의미한다. 이 글에서는 Mock 함수를 리뷰하고, 의존성을 교체할 수 있는 다양한 방법들에 대해 알아보도록 한다.

## Mock Function

Mocking의 목적은 우리가 컨트롤하지 않는 것을 컨트롤 할 수 있는 것으로 교체하는 것이기 때문에 교체 대상이 우리가 필요로 하는 모든 기능들이 있는 것이 중요하다.

Jest의 Mock Function은 아래와 같은 기능들을 제공한다.

- 호출 캡쳐링 (Capture calls)
- 반환 값 설정 (Set return values)
- 구현 자체를 변경 (Change the implementation)

Mock function 인스턴스를 생성하는 가장 쉬운 방법은 `jest.fn()`을 사용하는 것이다.

이것과 [Jest Expect](https://jestjs.io/docs/en/expect.html)를 사용하면, 호출을 캡쳐링하는 것이 간단하다.

```javascript
test('returns undefined by default', () => {
  const mock = jest.fn();

  let result = mock('foo');

  expect(result).toBeUndefined();
  expect(mock).toHaveBeenCalled();
  expect(mock).toHaveBeenCalledTimes(1);
  expect(mock).toHaveBeenCalledWith('foo');
});
```

(역자주) 윗 사례에서 볼 수 있듯이, `jest.fn()`으로 생성된 함수는 호출된 횟수와 전달된 매개변수, 호출 여부를 탐지할 수 있다.

이와 더불어 반환 값, 구현 내용, 혹은 Promise 결과값을 변경할 수 있다.

```javascript
test('mock implementation', () => {
  const mock = jest.fn(() => 'bar');

  expect(mock('foo')).toBe('bar');
  expect(mock).toHaveBeenCalledWith('foo');
});

test('also mock implementation', () => {
  const mock = jest.fn().mockImplementation(() => 'bar');

  expect(mock('foo')).toBe('bar');
  expect(mock).toHaveBeenCalledWith('foo');
});

test('mock implementation one time', () => {
  const mock = jest.fn().mockImplementationOnce(() => 'bar');

  expect(mock('foo')).toBe('bar');
  expect(mock).toHaveBeenCalledWith('foo');

  expect(mock('baz')).toBe(undefined);
  expect(mock).toHaveBeenCalledWith('baz');
});

test('mock return value', () => {
  const mock = jest.fn();
  mock.mockReturnValue('bar');

  expect(mock('foo')).toBe('bar');
  expect(mock).toHaveBeenCalledWith('foo');
});

test('mock promise resolution', () => {
  const mock = jest.fn();
  mock.mockResolvedValue('bar');

  expect(mock('foo')).resolves.toBe('bar');
  expect(mock).toHaveBeenCalledWith('foo');
});
```

## 의존성 주입 (Dependency Injection)

Mock function을 사용하는 일반적인 방법 중 하나로는, 테스트하는 함수의 매개변수 중 하나로 직접 전달하는 방법이다. 이 방법을 통해 테스트 대상을 실행하고, mock이 어떻게 실행되었으며 어떤 매개변수를 갖고 실행되었는지 확인할 수 있다.

```javascript
const doAdd = (a, b, callback) => {
  callback(a + b);
};

test('calls callback with arguments added', () => {
  const mockCallback = jest.fn();
  doAdd(1, 2, mockCallback);
  expect(mockCallback).toHaveBeenCalledWith(3);
});
```

이 방법은 굉장히 견고하지만, 이 방법은 코드의 의존성 주입 지원 여부를 요구한다. 일반적으로 그렇지 않기 때문에, 기존에 존재하는 모듈과 함수 mock 할 수 있는 도구를 필요로 한다.

## Mocking Modules & Functions

Jest에는 세 가지의 주 모듈 / 함수 mocking 방법이 존재한다.

- `jest.fn`: 함수를 mock한다
- `jest.mock`: 모듈을 mock한다
- `jest.spyOn`: 함수를 spy하거나 mock한다.

위 방법들은 각각의 방법으로 Mock function을 생성한다. 각자가 어떻게 동작하는지 보기 위해, 아래 프로젝트 구조를 살펴보자.

```md
├ example/
| └── app.js
| └── app.test.js
| └── math.js
```

이 구성에서, 일반적으로 `app.js`를 테스트하며 실제 `math.js` 함수들을 호출하지 않거나, 그들이 기대한 대로 호출되었는지 spy하고 싶을 수 있다. 해당 예제는 진부하지만, 한 번 `math.js`가 원하지 않는 IO와 같은 과정을 필요로 하는 굉장히 복잡한 연산이라 상상해보자.

```javascript
// math.js
export const add = (a, b) => a + b;
export const subtract = (a, b) => b - a;
export const multiply = (a, b) => a * b;
export const divide = (a, b) => b / a;
```

```javascript
// app.js
import * as math from './math.js';

export const doAdd = (a, b) => math.add(a, b);
export const doSubtract = (a, b) => math.subtract(a, b);
export const doMultiply = (a, b) => math.multiply(a, b);
export const doDivide = (a, b) => math.divide(a, b);
```

## jest.fn을 이용하여 함수 mock 하기

Mocking을 위한 가장 일반적인 전략은 해당 함수를 Mock function으로 재지정하는 것이다. 이 방법을 통해 재지정된 함수가 사용되는 곳 어디에서나 mock이 원 함수 대신에 호출될 것이다.

```javascript
// mock_jest_fn.test
import * as app from './app';
import * as math from './math';

math.add = jest.fn();
math.subtract = jest.fn();

test('calls math.add', () => {
  app.doAdd(1, 2);
  expect(math.add).toHaveBeenCalledWith(1, 2);
});

test('calls math.subtract', () => {
  app.doSubtract(1, 2);
  expect(math.subtract).toHaveBeenCalledWith(1, 2);
});
```

이 방식의 mocking은 아래와 같은 이유들로 일반적으로 많이 사용되지는 않는다.

- `jest.mock`은 모듈 안에 있는 모든 함수들을 대상으로 이 작업을 자동으로 한다.
- `jest.spyOn`은 동일하게 작동하지만 원 함수 내용을 유지한다.

## jest.mock을 이용하여 모듈 mock 하기

더 일반적인 방법은 `jest.mock`을 사용하여 자동으로 모듈의 모든 export 대상들을 Mock function으로 설정하는 것이다. 이 경우, `jest.mock('./math.js);`를 호출하는 것은 아래와 같이 `math.js` 파일을 정의하는 것과 같다.

```javascript
export const add = jest.fn();
export const subtract = jest.fn();
export const multiply = jest.fn();
export const divide = jest.fn();
```

이 지점부터, 우리는 해당 모듈의 모든 export되는 함수들에 대해 Mock function의 기능들을 사용할 수 있다.

```javascript
// mock_jest_mock.test.js
import * as app from './app';
import * as math from './math';

// Set all module functions to jest.fn
jest.mock('./math.js');

test('calls math.add', () => {
  app.doAdd(1, 2);
  expect(math.add).toHaveBeenCalledWith(1, 2);
});

test('calls math.subtract', () => {
  app.doSubtract(1, 2);
  expect(math.subtract).toHaveBeenCalledWith(1, 2);
});
```

이 방법은 가장 간단하고 일반적인 mocking 방법이라 할 수 있다.

이 전략의 유일한 단점은 모듈의 원본 함수 구현체를 접근하기 어렵다는 것이다. 이러한 사용자 케이스를 위해 `spyOn`을 사용한다.

## jest.spyOn을 이용하여 Spy하거나 Mock하기

간혹 작업하다보면 메소드의 호출 여부는 확인하고 싶지만, 원본 구현 내용을 유지하고 싶을 수 있다. 다른 경우, 구현 내용을 mock하지만 나중에 테스트 중에는 복원하고 싶을 수도 있다.

이러한 경우에, 우리는 `jest.spyOn`을 사용할 수 있다.

이 방법은 단순히 math 함수 호출을 "spy" 하지만, 원본 구현 내용은 그대로 놔둔다.

```javascript
// mock_jest_spyOn.test
import * as app from './app';
import * as math from './math';

test('calls math.add', () => {
  const addMock = jest.spyOn(math, 'add');

  // calls the original implementation
  expect(app.doAdd(1, 2)).toEqual(3);

  // and the spy stores the calls to add
  expect(addMock).toHaveBeenCalledWith(1, 2);
});
```

이 방법은 사이드 이펙트의 발생 여부는 확인하지만, 실제로 교체하고 싶지는 않은 시나리오들에서 유용하게 사용될 수 있다.

다른 경우에서는, 함수를 mock하고 싶지만, 원본 구현체를 복원하고 싶을 때 사용될 수 있다.

```javascript
// mock_jest_spyOn_restore.test.js
import * as app from './app';
import * as math from './math';

test('calls math.add', () => {
  const addMock = jest.spyOn(math, 'add');

  // 구현 내용을 오버라이딩 한다
  addMock.mockImplementation(() => 'mock');
  expect(app.doAdd(1, 2)).toEqual('mock');

  // 원본 구현 내용을 복원한다
  addMock.mockRestore();
  expect(app.doAdd(1, 2)).toEqual(3);
});
```

이 방법은 동일한 파일 내에서의 테스트에서는 유용하지만, 각각의 Jest 테스트 파일들은 각각의 샌드박스 안에 존재하게 됨으로 `afterAll` 훅에 추가하지 않아도 된다.

`jest.spyOn`에 대해 기억해야할 중요한 내용은 이것이 `jest.fn()` 사용법의 문법적 설탕이라는 점이다. 아래와 같이 원본을 저장해놓고, mock 구현 내용을 원본에 할당한 뒤, 다시 저장해두었던 원본 구현 내용을 재지정함으로써 동일한 결과물을 얻을 수 있다.

```javascript
// mock_jest_spyOn_sugar.js

import * as app from './app';
import * as math from './math';

test('calls math.add', () => {
  // store the original implementation
  const originalAdd = math.add;

  // mock add with the original implementation
  math.add = jest.fn(originalAdd);

  // spy the calls to add
  expect(app.doAdd(1, 2)).toEqual(3);
  expect(math.add).toHaveBeenCalledWith(1, 2);

  // override the implementation
  math.add.mockImplementation(() => 'mock');
  expect(app.doAdd(1, 2)).toEqual('mock');
  expect(math.add).toHaveBeenCalledWith(1, 2);

  // restore the original implementation
  math.add = originalAdd;
  expect(app.doAdd(1, 2)).toEqual(3);
});
```

실제로, 위 방법은 `jest.spyOn`이 [구현된 방식](https://github.com/facebook/jest/blob/e9aa321e0587d0990bd2b5ca5065e84a1aecb2fa/packages/jest-mock/src/index.js#L674-L708)과 동일하다.

## 결론

이 글에서, 우리는 함수 호출 추적, 구현 내용 변경, 그리고 반환 값 설정을 위해 모듈과 함수를 재지정하는 Mock function과 다양한 전략들을 알아보았다.

이 글이 당신이 어려움 없이 테스트 코드를 작성하는데 많은 도움이 되었길 바란다. :)

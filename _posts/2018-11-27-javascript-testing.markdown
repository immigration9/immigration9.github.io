---
layout: post
title: 'Delightful Javascript Testing - Jest'
date: 2018-11-27 01:30
categories: 'javascript'
---

## 사용 시작

디자인 컴포넌트 라이브러리를 만들다보니 슬슬 규모가 커지는게 느껴졌다. 그 전에는 그냥 `assert` 라이브러리로 작업을 했는데, 아래와 같은 문제점이 발생하였다.

```javascript
console.log("QuickSort Test");
console.log("#. isEqual test");
assert.strictEqual(
  qs.sort(testArray1, { isClone: true }).toString(),
  [0, 1, 2, 3, 4, 5, 7].toString()
)

assert.strictEqual(
  qs.sort(testArray2, { isClone: true }).toString(),
  [1, 2, 3, 4, 5, 6, 11].toString()
)

assert.strictEqual(
  qs.sort(testArray_test, { isClone: false }).toString(),
  testArray_test.toString()
)
console.log("#. isNotEqual test");
console.log("#. cloning test");
console.log("#. option test");
...
```

바로 관리가 너무 어렵다는 것이었다. 매번 테스트 진행할 때마다 `console.log`로 명시해줘야하고, 나중에 모듈에 변화하더라도, 쉽게 캐치할 수 없다는 점이 크게 다가웠다. 무엇보다 저런 파일들이 여러개 생기다보니 관리하는 것이 오히려 코드를 작성하는 것보다 많은 공수가 들어가는 것 같아, Javascript용 테스팅 프레임워크를 사용하기로 결정하였다. 여러 후보들이 있었으나, 가장 많이 쓰이는 것으로 보이는 'Delightful Javascript Testing' 프레임워크, Jest로 결정하였다.

Jest를 결정한 이유는, React / Typescript integration이 용이하고, 전반적으로 설정이 간편해보여서 결정하였다. 사실 Testing을 하는건 좋지만, 거기에 코드를 작성하는 것보다 많은 시간을 투자하고 싶지는 않았기 때문이다.

- Jest: https://jestjs.io/en/
  이 블로그 글이 작성될 당시 Jest의 버전은 23.6이다.

## Starting with Jest

- 처음 시작하는 경우 다음 절차를 따르면 된다: https://jestjs.io/docs/en/getting-started.html

나와 같은 경우, `Babel`을 사용하여 빌드를 하고 있기 때문에, 별도의 Babel용 모듈들을 다운받아줘야 한다.

```javascript
npm install --save-dev jest babel-jest babel-core@^7.0.0-bridge.0 @babel/core regenerator-runtime
```

슬프게도, 저 버전의 `babel-core`를 제외하고 정식 버전에서는 따로 실행이 되지 않는 것 같았다... 불편하지만 Jest를 사용해야하니까 굳이 시간을 더 투자하지 않고 그냥 버전을 갈아 끼운다.

`.babelrc` 파일은 다음과 같이 ES6와 React를 사용할 수 있도록 설정되어있다.

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": ["@babel/plugin-proposal-class-properties"]
}
```

## Matchers

가장 기본적인 테스팅은 `test`와 `expect` api를 사용하여 진행한다.

```javascript
test('object assignment', () => {
  const data = { one: 1 };
  data['two'] = 2;
  expect(data).toEqual({ one: 1, two: 2 });
});
```

`test` 함수는 타이틀과 콜백함수를 받는다. 콜백함수에는 `expect` 함수가 호출되었는데, 호출 후 뒤에 오는 값 (`toBe`, `toEqual`, `not.toBe`) 등과의 기댓값을 비교한다.

### 신뢰성 향상

테스트에서 올바른 답변이 아닌 경우를 확인해보고 싶을 경우, 별도로 처리를 해줄 수 있다. 일반적으로 `undefined`, `null`, 그리고 `false` 셋중 하나가 쓰이는데, 셋 모두 별도로 커버할 수 있다.

- `toBeNull`: `null` 확인
- `toBeUndefined`: `undefined` 확인
- `toBeDefined`: `toBeUndefined`의 반대
- `toBeTruthy`: `if` 문에서 `true`
- `toBeFalsy`: `if` 문에서 `false`

아래 예제를 통해 `null`과 0이 어떤 항목에 속하는지 알 수 있다.

```javascript
test('null test', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});

test('zero test', () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
});
```

### Numbers

숫자 값을 비교함에 있어서는 아래와 같은 함수들을 사용할 수 있다.

- `toBe` / `toEqual`: 숫자에 있어서 둘은 '같음'을 뜻한다. (동일)
- `toBeGreaterThan`: (비교대상)보다 크다
- `toBeGreaterThanOrEqual`: (비교대상)보다 크거나 같다
- `toBeLessThan`: (비교대상)보다 작다
- `toBelessThanOrEqual`: (비교대상)보다 작거나 같다

- `toBeCloseTo`: 소수값에 대해서 테스트를 진행할 경우 근사값을 기반으로 테스팅할 수 있다.

### Strings

- `toMatch`: 정규식과의 비교값을 도출한다.

```javascript
test('there is no I in team', () => {
  expect('team').not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/);
});
```

### Arrays

- `toContain`: 행렬 안에 특정 값이 있는지를 확인할 수 있다.

```javascript
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'beer'
];

test('shopping list test', () => {
  expect(shoppingList).toContain('beer');
});
```

## 비동기 코드 테스팅

자바스크립트에서는 코드가 비동기로 실행되는 것이 일반적이다. Jest에서는 다양한 방법을 이용하여 비동기 코드를 테스팅할 수 있다.

### 콜백함수를 통한 테스팅

가장 일반적인 방법으로 콜백함수를 호출할 수 있다. 일반적으로, Jest 테스팅은 실행을 마치면 종료된다. 그 뜻은 아래와 같이 구성된 코드는 동작하지 않는다는 것이다.

```javascript
test('the data is peanut butter', () => {
  function callback(data) {
    expect(data).toBe('peanut butter');
  }

  fetchData(callback);
});
```

여기서 문제는 `fetchData`를 호출한 다음에 콜백 함수를 호출하지 않고 테스트가 바로 끝나버린다는 것이다.
종료 지점을 지정해줘야 하기 때문에, 아래와 같이 코드를 수정한다.

```javascript
test('the data is peanut butter', done => {
  function callback(data) {
    expect(data).toBe('peanut butter');
    done();
  }

  fetchData(callback);
});
```

`done` 함수를 호출함으로써 Jest에게 직접 종료 지점을 알려줄 수 있다.

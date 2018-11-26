---
layout: post
title:  "Delightful Javascript Testing - Jest"
date:   2018-11-27 01:30
categories: "javascript"
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

* Jest: https://jestjs.io/en/
이 블로그 글이 작성될 당시 Jest의 버전은 23.6이다.

## Starting with Jest
* 처음 시작하는 경우 다음 절차를 따르면 된다: https://jestjs.io/docs/en/getting-started.html

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
  expect(data).toEqual({one: 1, two: 2});
})
```
`test` 함수는 타이틀과 콜백함수를 받는다. 콜백함수에는 

// TODO from Here
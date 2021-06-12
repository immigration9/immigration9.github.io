---
layout: post
title: "React 18: 렌더링 최적화를 위한 자동 배칭"
date: 2021-06-12 13:00:00 +09:00
categories: "react"
published: true
---

## Introduction

2021년 6월 8일 React 팀에서 새로운 React 18 버전을 발표하며 오랫동안 기다리던 (한편으로는 Vaporware가 되는 것은 아닌가 우려했던) Concurrent React를 다시 한 번 강조했습니다. 사실 이미 16 때부터 나온다 나온다 하던 것이다 보니 이미 어느정도 친숙했지만, 사실 운영환경에서는 사용하기 어려웠다보니 이번에 명시적으로 Concurrent React가 확인된건 크게 기대되는 바입니다.

이번 기회에 React 팀에서 발표한 Concurrent React에 대한 내용과, 그 동안 어렴풋이 알지만, 정확히 어떤 것을 하는 것인지에 대해서는 많은 자료가 없던 Concurrent Mode + Suspense에 대한 내용도 여러편에 나누어 다뤄보고자 합니다.

이번에는 첫 번째 주제로 항상 헷갈리고, 또 명확한 문서가 찾기 어려워 항상 혼선을 주던 'React는 state를 언제 batching하는가'에 대해 React core팀의 Dan Abramov가 작성한 글을 번역해보았습니다.

원글은 Automatic batching for fewer renders in React 18 (React 18: 렌더링 최적화를 위한 자동 배칭)이며, React 18 변경점을 소개하며 이전 버전에서의 배칭 메커니즘에 대해 소개하기에 다뤄보고자 합니다.

- Batching 자체는 일괄 처리로 번역할 수 있으나, 배칭이란 용어 자체가 생소한 개념이 아니기 때문에 직역하여 한글로 옮겼습니다.

원글: [Automatic batching for fewer renders in React 18](https://github.com/reactwg/react-18/discussions/21)

## Overview

React 18은 더 많은 배칭을 통해 별도의 수동 배칭을 하지 않고도 성능 개선을 바로 누릴 수 있다. 이 번 포스팅에서는 배칭이 어떤 것이고, 이전에 어떻게 작동하였으며, 어떻게 변화할 것인지 다뤄보고자 한다.

## 배칭이란? (What is batching?)

배칭은 React가 더 나은 성능을 위해 여러 개의 state 업데이트를 하나의 리렌더링 (re-render)로 묶는 것을 의미한다.

예를들어, 하나의 클릭 이벤트 안에 두 개의 state 업데이트를 가지고 있다면, React는 언제나 이 작업을 배칭하여 하나의 리렌더링으로 만들었다. 다음과 같은 코드를 실행해보면, 매 번 누를 때마다, state를 두 번 변경하였지만, React가 단 한 번의 렌더링만 수행한 것을 볼 수 있다.

```javascript
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    setCount((c) => c + 1); // 아직 리렌더링 하지 않는다
    setFlag((f) => !f); // 아직 리렌더링 하지 않는다
    // React는 이 함수가 끝나면 리렌더링을 한다 (이것이 배칭이다!)
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```

- [데모: 이벤트 핸들러 내부에서 React 17의 배칭](https://codesandbox.io/s/spring-water-929i6?file=/src/index.js) (클릭시 렌더가 한 번만 콘솔에 찍히는 것을 보자)

이 과정은 불필요한 리렌더링을 줄이기 때문에 성능에 굉장히 좋다. 또한, 컴포넌트가 "반만 완료된" state를 렌더링하는 것을 방지한다. (이 것 또한 버그를 발생시킬 수 있다). 레스토랑 웨이터에 비유를 하면 더 쉽게 와닿을 수 있는데, 주문을 할 때 하나 고를 때마다 주방으로 달려가지 않고, 오더를 완성시킬 때까지 대기하는 것과 같다.

하지만, React는 그 동안 업데이트에 대한 배칭을 언제할 것인지 일관적이지 못했다. 예를들어, 데이터를 외부 소스로부터 가져와서 아래 보이는 `handleClick` 함수 내부에서 state를 업데이트를 하고자 하면, React는 업데이트를 배칭하지 않고, 두 개의 독립적인 업데이트를 수행하였다.

일관적이지 않은 이유는 React가 클릭과 같은 브라우저 이벤트의 업데이트만 배칭을 해왔기 때문이고, 이 경우에 fetch 콜백에서 이벤트가 핸들링이 완료된 이후에 state를 업데이트하기 때문에 배칭이 적용되지 않는 것이다.

```javascript
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 17과 이전 버전에서는 이 업데이트들이
      // 이벤트가 *진행되는 중*이 아닌, *완료된 후의* 콜백에서 실행되기 때문에
      // 배칭되지 않았다.
      setCount((c) => c + 1); // 리렌더링을 발생시킨다.
      setFlag((f) => !f); // 리렌더링을 발생시킨다.
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```

- [데모: React 17은 이벤트 핸들러 밖에서 발생하는 업데이트를 배칭하지 않는다](https://codesandbox.io/s/trusting-khayyam-cn5ct?file=/src/index.js) (콘솔에 렌더가 두 번 찍히는 것을 보자)

React 18 이전까지, React 이벤트 핸들러 내부에서 발생하는 업데이트만 배칭을 하였다. Promise, setTimeout, native 이벤트 핸들러, 그리고 여타 모든 이벤트 내부에서 발생하는 업데이트들은 React에서 배칭되지 않았다.

## 자동 배칭이란 무엇인가? (What is automatic batching?)

React 18의 `createRoot`를 통해, 모든 업데이트들은 어디서 왔는가와 무관하게 자동으로 배칭되게 된다.

이 뜻은, timeout, promise, native 이벤트 핸들러와 모든 여타 이벤트는 React에서 제공하는 이벤트와 동일하게 state 업데이트를 배칭할 수 있다. 이를 통해 우리는 렌더링을 최소화하고, 나아가 애플리케이션에서 더 나은 성능을 기대한다.

```javascript
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 18과 이후 버전에서는 아래 항목들을 배칭한다.
      setCount((c) => c + 1);
      setFlag((f) => !f);
      // React는 이 콜백이 끝났을 때만 리렌더링을 하게 된다 (이제 여기도 배칭이 들어간다!)
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```

- [✅ 데모: React 18 + `createRoot`는 외부 이벤트 핸들러도 배칭해준다!](https://codesandbox.io/s/morning-sun-lgz88?file=/src/index.js) (콘솔에 렌더가 한 번만 찍히는 것을 주목하자!)
- [🟡 데모: React 18 + 레거시 `render`는 이전 방식을 유지한다](https://codesandbox.io/s/jolly-benz-hb1zx?file=/src/index.js) (콘솔에 렌더가 두 번 찍히는 것을 확인할 수 있다)

> Note: React 18을 도입할 때 `createRoot`로 업그레이드 하는 것이 권장된다. `render`를 통해 확인 가능하도록 한 유일한 이유는 프로덕션 환경에서 테스팅이 용이하기 때문이다.

React는 업데이트의 발생지점과 무관하게 자동으로 업데이트를 배칭한다. 그렇기에 아래의 예제:

```javascript
function handleClick() {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // React는 이 함수가 끝날 때만 리렌더링을 한다 (배칭이다!)
}
```

는 이 예제와 동일하게 동작하고

```javascript
setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // React는 이 함수가 끝날 때만 리렌더링을 한다 (배칭이다!)
}, 1000);
```

또한, 아래 항목과 동일하게 동작하고

```javascript
fetch(/*...*/).then(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // React는 이 함수가 끝날 때만 리렌더링을 한다 (배칭이다!)
});
```

또한, 아래 항목과도 동일하게 동작한다!

```javascript
elm.addEventListener("click", () => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // React는 이 함수가 끝날 때만 리렌더링을 한다 (배칭이다!)
});
```

> Note: React는 업데이트 배칭이 '안전할 때만' 수행한다. 예를들어, React는 click이나 keypress와 같은 유저가 실행한 이벤트의 경우 다음 이벤트 수행 이전에 DOM이 완벽히 업데이트 되도록 보장한다. 이 과정을 통해 제출(submit) 버튼을 눌렀을 때 폼(form)을 비활성화 시킴으로써 폼이 두 번 전송되는 것을 막아준다.

## 배칭을 하고 싶지 않다면?

대부분의 경우 배칭은 안전한 절차지만, 몇몇 코드는 state 변경 후 즉시 DOM으로부터 값을 가져오는 것에 의존한다. 이런 경우, `ReactDOM.flushSync()`를 사용함으로써 배칭을 하지 않을 수 있다.

```javascript
import { flushSync } from "react-dom"; // Note: react가 아닌 react-dom이다

function handleClick() {
  flushSync(() => {
    setCounter((c) => c + 1);
  });
  // 이 과정이 끝났을 때 React는 DOM을 업데이트한 상태이다
  flushSync(() => {
    setFlag((f) => !f);
  });
  // 이 과정이 끝났을 때 React는 DOM을 업데이트한 상태이다
}
```

이 과정이 일반적이리라 생각하진 않는다.

## Hook을 사용할 때 문제가 생길 수 있을까?

Hook을 사용하고 있다면, 거의 모든 경우에 있어 자동 배칭은 아무 문제 없이 동작할 것이다 (만약에 그렇지 않다면 알려달라)

## Class를 사용할 때 문제가 생길 수 있을까?

React의 이벤트 핸들러는 언제든 배칭이 되고 있었기에, 이 부분에 있어 변화는 없다.

Class component를 사용할 때는 문제가 생길 수 있는 예외 케이스가 존재한다.

Class component에는 이벤트 내부에서 state 업데이트된 값을 동기적으로 읽을 수 있는 구현 특성이 있었다. 이 뜻은, `setState` 호출 사이에 `this.state`의 변화 값을 읽을 수 있었다는 것이다.

```javascript
handleClick = () => {
  setTimeout(() => {
    this.setState(({ count }) => ({ count: count + 1 }));

    // { count: 1, flag: false }
    console.log(this.state);

    this.setState(({ flag }) => ({ flag: !flag }));
  });
};
```

React 18에서 이건 더 이상 동작하지 않는다. `setTimeout` 안에 있는 모든 업데이트도 배칭되기 때문에, React는 더 이상 첫 번째 `setState`의 결과를 동기적으로 렌더링하지 않는다. 렌더링은 다음 브라우저 tick상에서 발생하게 되기에 렌더가 아직 수행되지 않은 상태로 남는다.

```javascript
handleClick = () => {
  setTimeout(() => {
    this.setState(({ count }) => ({ count: count + 1 }));

    // { count: 0, flag: false }
    console.log(this.state);

    this.setState(({ flag }) => ({ flag: !flag }));
  });
};
```

[Sandbox](https://codesandbox.io/s/interesting-rain-hkjqw?file=/src/App.js)를 확인해보자.

만약에 이 케이스가 React 18로의 업그레이드를 막는 원인이 된다면, `ReactDOM.flushSync`를 이용하여 업데이트를 강제할 수 있지만, 최대한 사용하지 않는 것을 추천한다.

```javascript
handleClick = () => {
  setTimeout(() => {
    ReactDOM.flushSync(() => {
      this.setState(({ count }) => ({ count: count + 1 }));
    });

    // { count: 1, flag: false }
    console.log(this.state);

    this.setState(({ flag }) => ({ flag: !flag }));
  });
};
```

[Snadbox](https://codesandbox.io/s/hopeful-minsky-99m7u?file=/src/App.js)를 통해 확인해보자.

`useState`에서 state 변경은 기존 값을 업데이트하지 않기에 Hooks를 가진 함수형 컴포넌트는 이 이슈에 영향을 받지 않는다.

```javascript
function handleClick() {
  setTimeout(() => {
    console.log(count); // 0
    setCount(c => c + 1);
    setCount(c => c + 1);
    setCount(c => c + 1);
    console.log(count); // 0
  }, 1000)
```

이러한 방식이 Hooks를 처음 도입하였을 때 어색했겠지만, 이 방법을 통해 자동 배칭이 진행될 수 있었다.

## unstable_batchedUpdates의 경우 어떻게 될까?

몇몇 React 라이브러리들으 이벤트 핸들러 밖의 `setState`가 배칭되는 것을 강제하기 위해 이 도큐먼트에도 없는 API를 사용하고 있다.

```javascript
import { unstable_batchedUpdates } from "react-dom";

unstable_batchedUpdates(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
});
```

이 API는 React 18에서도 존재할 것이지만, 배칭이 자동으로 동작하기에 사실 더 이상 필요는 없다. 18 버전에서 없앨 예정은 아니고, 향후에 메이저 라이브러리들이 이 API 사용을 지우고난 뒤에 주요 버전 업데이트에서 없앨 예정이다.

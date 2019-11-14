---
layout: post
title: 'React Fundamentals'
date: 2019-08-10 02:30:00 +09:00
categories: 'javascript,react'
published: true
---

## Lifecycle In Order

### Mount Phase

constructor -> getDerivedStateFromProps -> render -> componentDidMount

- 놀랍게도 한 가지 잘못 알고 있던 것이, render가 componentDidMount 전에 온다는 것이었다.
- componentDidMount에서는 setState를 해도 괜찮다.

* 이 과정에서 render가 두 번 호출되긴 하지만, Browser가 화면을 Update하기 전에 행해지기 때문에 그 중간의 간극을 발견할 수는 없다. 일반적으로 굳이 State를 변경하고 싶다면 constructor에서 실행하는 것이 맞지만, 이 경우는 주로 DOM이 Mount된 이후, Modal이나 Tooltip과 같은 요소들의 사이즈를 측정 할 때 사용 될 수 있다.

### Update Phase

getDerivedStateFromProps -> shouldComponentUpdate -> render -> getSnapshotBeforeUpdate -> componentDidUpdate

- componentDidUpdate에서 Side effect를 실행하도록 하자.

## Smart Component & Dumb Component

React Component를 정의하는 패턴 중 하나로, 컴포넌트 분류 방법 패턴이다. Dumb Component는 Presentational Component로 불리기도 하며, Smart Component는 Container Component, Stateful Component로도 불린다.

- React Hook 등장 이후로는 원 저자인 Dan Abramov가 더 이상 추천하지 않는다.

### Presentational Component

Presentational Component (aka. Dumb Component)는 DOM에 그려지는 것만을 담당한다. 로직이 들어가지 않기 때문에 Dumb Component로도 불리는 것이며, 가장 효과적으로 사용하는 방법은 '재사용성이 요구되는' 컴포넌트에 사용하는 것이다.

이외에도 Dan Abramov는 다음과 같은 항목들에 해당하는 것을 Presentational Component라 한다.

- Flux Action, Redux Store와 같은 것에 의존성을 갖지 않는다.
- 데이터 로딩, 변이 같은 과정에 간섭하지 않는다.
- Prop을 통해 데이터와 Callback을 전달받는다.
- 거의 State를 사용하지 않는다. (사용한다면 UI와 관련된 것만 사용한다)
- 일반적으로 lifecycle hook 사용이나, 성능 개선, 혹은 내부 State를 필요로 하지 않는다면 functional component로 작성된다.
- 예를들어, Page, Sidebar, Story, List 등이 있다.

### Container Component

Container Component (aka. Smart Component)는 내부 State와 로직을 가지고 있다. Presentational Component와 동일하게 Prop을 전달 받아 다른 Smart / Dumb Component에 전달할 수다. Class 형식으로 작성된다.

- 다른 Component에 데이터나 행동방향(콜백등)을 제공한다.
- Flux, Redux 등을 호출하여 사용한다.
- 주로 내부 State를 가지며, Data Source로 사용되곤 한다.

### 이 방법의 이점

무엇보다, 가장 큰 것은 SOC (Separation of Concern)을 할 수 있다는 점일 것이다.
재사용성을 보장하며 사용할 수 있어, Presentational Component는 정말 App UI를 그리는 데에만 사용하고, 수정 과정에서도 애플리케이션의 로직을 건드리지 않을 수 있다.

- UI Testing에도 굉장히 유용할 수 있다.

## Reconciliation (재조정)

리액트의 재조정은 무수히 많은 엘리먼트를 그릴 때 최소한의 비교 연산 `O(n)`만을 수행하도록 하는 알고리즘을 수반한다. 해당 알고리즘은 아래 두 항목의 가정을 기반한다.

1. 서로 다른 타입의 두 엘리먼트는 서로 다른 트리를 만들어 낸다.

- 두 루트 엘리먼트의 타입이 다르면, 이전 DOM 노드를 전부 파괴하고 (`componentWillUnmount` 실행), 새로운 DOM 노드들이 상위 루트 DOM에 삽입된다. (이 과정에서 `componentDidMount` 실행).
- 컴포넌트의 엘리먼트 타입은 같을 경우, 내부에 있는 속성만 갱신된다.
- 같은 타입의 컴포넌트 엘리먼트일 경우, 인스턴스는 동일하게 유지되며, 각 인스턴스의 `componentWillReceiveProps`에 해당하는 함수가 호출되게 된다. (현재는 다를 것)
- 그 후 `render`가 호출된다.

2. 개발자가 key prop을 통해, 여러 렌더링 사이에서 어떤 자식 엘리먼트가 변경되지 않아야 할지 표시해 줄 수 있다.

- 자식 엘리먼트는 각각 key 속성을 가지고 있어, key를 기반으로 색인하기 때문에, 그냥 행렬의 형식으로 가지고 있는 것보다 훨씬 빠르게 동작한다.
- key는 형제 사이에서만 유일하면 되지만, 항목 재배열 문제 때문에 배열의 index를 key로 사용하는 것은 권장되지 않는다.
  - [https://codepen.io/pen?editors=0010](index 값을 key로 선택했을 때 발생하는 문제)
  - 확인해보면, `<input />`안에 있는 값이 정상적으로 바뀌지 않는 것을 발견할 수 있다.

## High Order Components

High Order Component (HOC, 고차 컴포넌트)는 컴포넌트 로직 재사용을 위해 사용된다. 리액트 자체의 API에서 온다기 보단, 리액트가 가지고 있는 Composition(합성) 방식의 컴포넌트 구조에서 비롯된다.

요약하자면, Component를 받아서 새로운 컴포넌트를 반환하는 함수라 할 수 있다.

- 주로, 중복되는 로직을 가지고 있는 경우, HOC하나에서 해당 로직을 수행하고, 렌더되는 컴포넌트를 반환하도록 해준다.

## Render Props Pattern

Render Prop은, React 컴포넌트 간에 코드를 중복하지 않고 데이터와 기능을 공유하도록 한다.
가장 큰 사용법 중 하나는, 횡단 관심사 (Cross-Cutting Concerns) 상황에서 코드 재사용성을 보장하기 위해 사용될 수 있다.

컴포넌트 내부에서 `render` 함수를 선언해 놓음으로써, 하단(Child로)에 추가될 코드들은 해당 Prop을 받아서 사용할 수 있게 된다. 이 방법을 통해, 같은 행동을 하는 코드에 대해서 중복 작성을 하지 않아도 되고, Logic과 UI를 쉽게 구분할 수도 있다.

## References

[https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0](Presentational and Container Components)

[https://ko.reactjs.org/docs/render-props.html](Render Props)

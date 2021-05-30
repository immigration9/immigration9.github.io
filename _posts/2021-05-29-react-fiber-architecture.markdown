---
layout: post
title: 'React Fiber Architecture'
date: 2021-05-29 18:00:00 +09:00
categories: 'react'
published: true
---

본 글은 React Core Team에 근무 중인 Andrew Clark (aka. acdlite)가 Facebook에 합류하기 전에 본인이 스스로 정리해본 React Fiber Architecture에 대한 글을 번역한 글입니다.

원문은 아래와 같습니다.
[React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture)

## Introduction

React Fiber(리액트 파이버로 읽으면 될듯하다. 여기서는 원문 그대로 Fiber를 사용하겠다)는 현재 진행 중인 React core 알고리즘 재구성이다. React 팀의 2년간의 연구 결과이다.

Fiber의 목적은 animation, layout, gesture (애니메이션, 레이아웃, 제스처)와 같은 영역들에 있어서 React의 적합성을 확보하기 위함이다. 주요 주제는 점증적 렌더링 (incremental rendering)으로, 렌더링 작업을 chunk 단위로 나눈뒤 여러 프레임에 수행하는 것을 의미한다.

이와 더불어 새로운 업데이트가 들어올 때 기존의 작업을 멈추거나, 정지하거나, 재사용하는 기능들을 포함한다. 이외에도 다른 종류에 업데이트에 우선순위를 부여하거나, 새로운 동시성 모드를 위한 초기 작업들이 포함된다.

### About this document

Fiber는 단순히 코드만 봐서는 이해하기 어려운 여러가지 기발한 컨셉들이 등장한다. 이 문서는 React 프로젝트에서 Fiber의 도입 방식을 추적하며 남겨온 메모들로부터 시작하였다. 문서의 양이 증가하며 나는 다른 이 문서가 다른이들에게 도움이 될만한 리소스가 될 수 있으리라 생각하게 되었다.

문서를 진행하며 최대한 이해 가능한 언어를 사용하고, 중요 어휘들은 명시적으로 의미를 밝혀 이해가 최대한 용이하도록 하겠다. 또한, 중간 중간 가능할 때마다 외부 리소스 링크를 최대한 남겨놓겠다.

다만 문서 읽기에 앞서, 나는 React 팀에 소속되어 있지도 않고, 어떠한 권한을 갖고 말하지 않음을 알았으면 한다. **이건 공식문서가 아니다.** 정확성을 위해 React 팀에 소속된 구성원들에게 리뷰를 요청해둔 상태이다.

동시에 이 작업은 현재 진행형이다. **Fiber는 완성되기 전에 굉장히 많은 리팩토링을 거칠 가능성이 높은 프로젝트다.** 동시에 진행 중인건 내가 Fiber의 디자인을 문서화 하고 있는 것이다. 개선사항이나 제안사항은 언제나 환영이다.

당신이 이 문서를 읽은 뒤 당신이 충분히 Fiber를 이해하여 그 개발 과정을 충분히 따라갈 수 있도록 되고, 나아가 React에 다시 기여할 수 있는 것이 나의 목표다.

### 사전 필요 항목

더 읽기 전에 아래 리소스들에 충분히 익숙해지길 추천한다.

[React Components, Elements, and Instances](https://facebook.github.io/react/blog/2015/12/18/react-components-elements-and-instances.html): "Component"는 지나치게 다양하게 사용되는 단어다. 이런 용어에 대한 확고한 이해는 매우 중요하다.

[Reconciliation](https://facebook.github.io/react/docs/reconciliation.html): React 재조정 알고리즘에 대한 고레벨 수준의 설명

[React Basic Theoretical Concepts](https://github.com/reactjs/react-basic): React의 개념적 모델에 대한 설명. 처음 읽을 때는 몇가지 항목들이 이해가 가지 않을 수 있다. 괜찮다, 읽으면 읽을 수록 이해가 된다.

[React Design Principles](https://facebook.github.io/react/contributing/design-principles.html): 특히 스케쥴링 항목에 대해 집중해볼 것.

## 리뷰

아직 사전 필요 항목을 보기 전이라면 꼭 먼저 보길 바란다.
새로운걸 배우기 전에, 몇 가지 주요 컨셉들을 짚고 넘어가자.

### 재조정 (Reconciliation) 이란?

_재조정(reconciliation)_

- React에서 어떤 부분들이 변해야하는지 서로 다른 두 개의 트리를 비교하는 데 사용하는 알고리즘

_업데이트(update)_

- React 애플리케이션을 렌더하기 위해 사용되는 데이터의 변화. 주로 `setState`와 같은 함수 결과로 나타난다. 결론적으로 리렌더링(re-rendering)의 결과물이다.

React API의 중심 아이디어는 업데이트로 하여금 전체 애플리케이션을 리렌더링을 유발하는 것으로 생각하는 것이다. 이 방향은 개발자로 하여금 특정 상태로부터 다른 상태로 애플리케이션을 어떻게 하면 효율적으로 전이시킬 수 있는지에 대한 고민에서 벗어나 선언적으로 추론할 수 있게 된다.

사실 전체 애플리케이션을 매 변화가 있을 때마다 리렌더링 하는 것은 정말 정말 작은(trivial) 앱에서나 가능한 일이다. 실제 애플리케이션에서는 이럴 경우 퍼포먼스 상에서 금지될 정도로 무거운 작업이다. React는 최적화를 통해 리렌더링 속에 전체 앱을 렌더하면서도 동시에 굉장한 성능을 유지할 수 있다. 이런 최적화의 많은 부분들은 **재조정**이란 과정의 일부이다.

재조정은 "virtual dom"을 책임지는 알고리즘이다. 설명하자면 다음과 같다: React 애플리케이션을 렌더링하면, 앱을 나타내는 노드들이 달린 트리가 생성되고 메모리에 저장된다. 이 트리는 렌더링 환경에 플러시된다. (예를들어, 브라우저 앱의 경우, DOM 작업의 형태로 번역된다). `setState`와 같은 방법을 통해 앱이 업데이트 되면, 새로운 트리가 생성된다. 새로운 트리는 렌더된 앱에 있어 변화가 필요한 부분들을 발견하기 위해 이전 트리와 비교된다.

비록 Fiber는 이 조정기(reconciler)를 밑바닥부터 다시 작성한 것이지만, 고레벨에 있어서 알고리즘은 [React 문서에 설명된 것과 같이](https://facebook.github.io/react/docs/reconciliation.html) 대부분 동일할 것이다. 여기서 키포인트는 다음과 같다:

- 다른 컴포넌트 타입은 상당히 다른 트리를 생성할 것이라 예상된다. React는 이런 경우에 둘을 비교하지 않고, 이전 트리를 완전히 교체할 것이다.
- 리스트(lists)는 key를 이용하여 비교된다. 여기서 key는 안정적이고, 예측 가능하며, 유일해야한다. (stable, predictable, and unique)

### 재조정 vs 렌더링 (Reconciliation vs Rendering)

DOM은 React가 렌더할 수 있는 환경들 중 하나이며, 이외에도 네이티브 iOS나 Android view를 대상으로 하는 React Native도 있다. (그렇기 때문에 "virtual DOM"이란 표현은 다소 잘못된 호칭이다).

다양한 타겟을 타게팅할 수 있는 이유는 React가 재조정과 렌더링을 다른 단계에서 진행하도록 디자인되었기 때문이다. 조정기(reconciler)는 트리의 어떤 부분들이 변화해야하는지를 계산하고, 렌더러(renderer)는 그 정보를 바탕으로 렌더된 앱을 업데이트한다.

이 단계들의 분리는 React DOM과 React Native가 서로 다른 렌더러를 사용하지만, React core에서 제공되는 같은 조정기를 사용할 수 있음을 의미한다.

Fiber는 이 조정기의 새로운 버전이라 볼 수 있다. 렌더링 과정에는 크게 관여하지 않지만, 렌더러들은 새로운 아키텍처에 맞춰 변화할 필요는 있다.

### 스케쥴링 (Scheduling)

_스케쥴링(scheduling)_
작업이 언제 수행되어야 하는지를 결정하는 과정

_작업(work)_
수행되어야 하는 어떠한 계산의 형태. 여기서 작업은 주로 `setState`와 같은 업데이트의 결과이다.

React의 [디자인 원칙](https://facebook.github.io/react/contributing/design-principles.html#scheduling) 문서는 이 부분에 있어 너무 잘 쓰여져있어 그대로 인용하도록 하겠다.

> 현재 도입방식에서 React는 한 번의 Tick 동안 재귀적으로 트리를 탐색하고 업데이트된 트리 전체의 렌더 함수들을 호출한다. 하지만 미래에는 프레임 드롭을 방지하기 위해 몇몇 업데이트들은 지연시킬 수 있다.

> 이것은 React 디자인에 있어 공통되는 주제이다. 몇몇 인기 라이브러리들은 새로운 데이터가 제공되었을 때 연산을 수행하는 "Push" 방식을 도입하였다. 반대로 React는 연산이 필요시까지 지연될 수 있는 "Pull" 방식을 고수한다.

> React는 제네릭(generic)한 데이터 처리 라이브러리가 아니다. React는 유저 인터페이스를 만드는 라이브러리다. 우리가 생각했을 때 React는 어떤 연산이 현재 적합하고, 어떤건 적합하지 않은지를 판별하기 위해 애플리케이션 내부에 굉장히 고유하게 위치한다고 생각한다.

> 어떤 항목이 스크린에서 벗어나면, 우리는 그것과 관련된 로직을 지연시킬 수 있다. 만약 데이터가 프레임 주사율보다 빠르게 도착하면, 업데이트를 합치고 배칭시킬 수 있다. 우리는 프레임 드롭을 방지하기 위해 유저 상호작용에서 오는 작업(버튼 클릭으로 발생하는 애니메이션과 같은)에 덜 중요한 백그라운드 작업(네트워크로부터 방금 막 도착하여 새롭게 렌더링되는 컨텐츠) 보다 높은 우선순위를 줄 수도 있다.

여기서 키포인트는 다음과 같다:

- UI에 있어 모든 업데이트가 꼭 즉각적으로 반영되어야 하는 것은 아니다. 오히려 그렇게 함으로써 프레임 드롭과 유저 경험 저하 등의 상황을 불러올 수 있다.
- 다른 종류의 업데이트들은 다른 종류의 우선순위를 갖는다. 예를들어 애니메이션 업데이트는 데이터 스토어 업데이트에 비해 빨리 완료되어야 한다.
- Push 기반 접근은 애플리케이션 (당신, 프로그래머)으로 하여금 스케쥴링이 어떻게 작동할 것인지 결정하도록 요구한다. Pull 기반 접근은 프레임워크 (이 경우 React)로 하여금 똑똑하게 그 결정사항들을 대신 결정할 수 있도록 해준다.

React는 아직 중요한 항목에 있어 이런 스케쥴링 장점을 살리지 못한다. 업데이트는 내부에 하위트리 전체로 하여금 즉각적으로 리렌더링 되도록 한다. React core 알고리즘을 정비하여 스케쥴링에 있어 강점을 갖는 것이 Fiber의 주요 쟁점이다.

---

자 이제 우리는 Fiber 제작에 뛰어들 준비가 되었다. 다음 섹션은 여태까지 우리가 논의한 항목들에 비해 더 기술적이다. 더 나아가기 전에 꼭 이전 항목들에 익숙해지길 권장한다.

## Fiber란 무엇인가?

우리는 이제 React Fiber 아키텍쳐의 심장에 대해 얘기해보고자 한다. Fiber는 애플리케이션 개발자들이 생각하는 것보다 훨씬 더 저레벨의 추상화된 형태로 존재한다. 이해하는 과정에서 지친 본인의 모습을 발견한다면, 낙담하지 않길 바란다. 계속해서 보다보면 어느순간 이해가 될 것이다. (정말 끝까지 이해가 안된다면, 어떻게하면 이 항목을 개선할 수 있을지도 제안해주길 바란다).

Here we go!

우리는 Fiber의 주요 목표가 React로 하여금 스케쥴링에 있어 강점을 갖도록 하는 것이라고 확립하였다. 특히 우리는 아래 항목들을 할 수 있어야 한다고 논의하였다.

- 작업을 중단하고 나중에 다시 돌아올 수 있어야 한다.
- 다른 종류의 작업에 우선순위를 부여할 수 있어야 한다.
- 이전에 완료된 작업을 재사용할 수 있어야 한다.
- 더 이상 필요 없어지면 작업을 중단할 수 있어야 한다.

위 항목들 중 하나라도 할 수 있으려면, 우리는 작업을 유닛 단위로 나눌 수 있어야 한다. 어떤 의미에서 그것 자체가 곧 Fiber라 할 수 있다. 하나의 fiber는 작업의 작은 단위라고 볼 수 있다.

더 나아가기 위해, [데이터 함수로서의 React 컴포넌트, 원제: React - 기본 이론적 개념](https://github.com/reactjs/react-basic#transformation) 개념으로 돌아가 아래를 살펴보자.

```
v = f(d)
```

이 개념은 React 애플리케이션에 있어 렌더링은 함수 내부에서 다른 함수 호출을 포함하고 있는 것과 유사하다는 점을 보여준다. 이러한 유추 방식은 fiber에 대해 생각함에 있어 유용하다.

일반적으로 컴퓨터가 프로그램의 실행을 추적하는 것은 [콜스택(call stack)](https://en.wikipedia.org/wiki/Call_stack)을 통해서다. 함수가 실행되면, 새로운 스택 프레임(stack frame)은 스택 위에 쌓이게 된다. 해당 스택 프레임은 해당 함수에 의해 실행되는 작업을 나타낸다.

UI를 다룰 때 문제는 너무 많은 작업이 동시에 수행되면 애니메이션 프레임 드롭이 생기고 전반적으로 뚝뚝 끊기는 느낌을 주게 된다. 더 나아가, 그 중 몇몇 작업들은 더 나중에 발생하는 작업들로 인해 불필요해질 수도 있다. 이 지점에서 UI 컴포넌트와 함수(function)의 비교가 세분화되는데, 일반적으로 컴포넌트는 함수 대비하여 더 구체적으로 고려해야할 사항들이 더 많기 때문이다.

새로운 브라우저들은 (React Native도 포함하여) 동일한 문제를 해결하는 것을 돕기위한 API들을 가지고 있다. `requestIdleCallback`은 유후 기간 동안 낮은 우선순위를 갖는 함수들이 스케쥴링될 수 있도록 하고, `requestAnimationFrame`은 다음 애니메이션 프레임에 높은 우선순위를 갖는 함수가 호출될 수 있도록 해준다. 여기서 문제점은, 이러한 API들을 사용하기 위해, 작업을 점증적인 단위로 나눌 방법이 필요하다. 콜스택에만 기대하게 되면, 브라우저는 스택이 비워질 때까지 작업을 진행할 것이다.

UI 렌더링을 최적화하기 위해 콜스택의 작업 방식이 커스터마이징할 수 있다면 굉장하지 않을까? 원할 때 콜스택에 인터럽트를 걸 수 있고 수동으로 스택프레임을 조정할 수 있다면 멋지지 않을까?

이것이 React Fiber의 목적이다. Fiber는 특별히 React component를 위한 스택의 재구현이다. 단일 fiber는 가상의 스택프레임(virtual stack frame)이라 볼 수 있다.

스택을 재구현하는 것의 장점은 [스택 프레임을 메모리에 저장하고](https://www.facebook.com/groups/2003630259862046/permalink/2054053404819731/) 원할 때 실행할 수 있다는 것이다. 이것은 스케쥴링 목표를 달성하기 위해 중요하다.

스케쥴링 이외에도, 스택 프레임을 수동으로 다룰 수 있는 것은 동시성(concurrency)이나 에러 범위(error boundaries)와 같은 기능들을 사용할 수 있게 해준다. 이러한 주제들은 다른 섹션에서 다뤄보도록 하겠다.

다음섹션에서, fiber의 구조에 대해 더 살펴보자.

### Fiber의 구조

구체적인 용어로 Fiber는 컴포넌트에 대한 입력과 출력에 대한 정보들을 가지고 있는 JavaScript 객체다.

하나의 fiber는 스택 프레임에 해당하며, 동시에 Component의 인스턴스에 해당되기도 한다

Fiber에 해당하는 중요한 항목들을 좀 리스트업 해보았다.

`type`과 `key`

Fiber의 type과 key는 React 요소에 있어 그것과 같은 목적을 갖는다. (실제로 React 요소로부터 fiber가 생성될 때, 이 두 필드는 복사되어 전달된다).

Fiber의 type은 그것이 상응하는 컴포넌트를 가리킨다. 합성 컴포넌트(Composite component)의 type은 함수 또는 클래스 컴포넌트 자체다. `div`나 `span`과 같은 호스트 컴포넌트(Host component)의 type은 문자열(string)이다.

개념상으로 type은 실행이 스택 프레임에의해 추적되는 함수이다 (`v = f(d)`의 사례에서 볼 수 있듯이)

type과 함께, key는 fiber가 재사용될 수 있는지를 판별하기 위해 재조정 중에 사용된다.

`child`와 `sibling`

이 필드들은 다른 fiber를 대상으로 하며, fiber의 재귀적 트리 구조를 가리킨다.

자식 fiber(a child fiber)는 컴포넌트의 `render` 함수가 반환하는 값에 해당한다.

```javascript
function Parent() {
  return <Child />;
}
```

그렇기에 위 예제에서 `Parent`의 자식 fiber는 `Child`에 해당된다.

Sibling field는 `render` 함수가 여러개의 자식을 반환할 때 설명된다:

```javascript
function Parent() {
  return [<Child1 />, <Child2 />];
}
```

자식 fiber들은 head가 첫번째 자식을 가리키고 있는 싱글 LinkedList를 구성한다. 위 예제에서, `Parent`의 자식은 `Child1`이고, `Child2`는 `Child1`의 형제 자매가 된다.

함수 유추로 돌아가서 설명하자면, 자식 fiber는 [꼬리 호출 함수(tail-called function)](https://en.wikipedia.org/wiki/Tail_call)라 생각할 수 있다.

`return`

return fiber는 현재 항목을 수행한 후 프로그램이 반환해야하는 정보를 담고 있는 fiber다. 개념상으로 스택 프레임의 반환 주소값과 같다. 또한 부모 fiber로 생각될 수도 있다.

fiber가 다수의 자식 fiber들을 갖고 있다면, 각각의 자식 fiber의 return fiber는 그 부모가 된다. 이전 섹션에서 본 예제를 통해 설명하자면, `Child1`과 `Child2`의 return fiber는 `Parent`가 된다.

`pendingProps`와 `memoizedProps`

개념상으로 props는 함수의 인수이다. Fiber의 `pendingProps`는 그것의 실행 처음에 설정되고, `memoizedProps`는 실행 마지막에 설정된다.

들어오는 `pendingProps`가 `memoizedProps`와 같으면, 이걸 통해 fiber에게 이전 출력물이 재사용될 수 있음을 알리고, 불필요한 작업을 방지할 수 있다.

`pendingWorkPriority`

Fiber 작업의 우선 순위를 나타내는 숫자. [ReactPriorityLevel (관련 설명 현재 삭제됨)](https://github.com/facebook/react/blob/master/src/renderers/shared/fiber/ReactPriorityLevel.js) 모듈은 다른 우선 순위 수준과 각각의 의미를 나열한다.

0값을 갖는 `NoWork`의 경우를 제외하고, 높은 숫자는 낮은 우선순위를 가리킨다. 예를들어, 아래 함수를 사용하여 fiber의 우선순위가 최소한 주어진 레벨만큼 높은지를 확인할 수 있다.

```javascript
function matchesPriority(fiber, priority) {
  return (
    fiber.pendingWorkPriority !== 0 && fiber.pendingWorkPriority <= priority
  );
}
```

_이 함수는 실제로 코드베이스에 있는 코드가 아니다. 보여주기 위함이다._

스케쥴러는 다음으로 수행할 작업 단위를 찾기 위해 우선 순위 필드를 사용한다.

`alternate`

_flush_

- fiber를 flush하는 것은 그 결과를 화면에 나타내는 것이다.

_work-in-progress_

- 아직 완료되지 않은 fiber. 개념상으로 아직 반환되지 않은 스택 프레임을 의미한다.

언제든지 컴포넌트 인스턴스는 여기에 부합하는 최대 두 개의 fiber를 갖는다. 현재 fiber(current fiber), flush된 fiber(flushed fiber), 그리고 work-in-progress fiber.

현재 fiber의 대체재는 work-in-progress 이며, 반대 역시 성립한다.

fiber의 대체재는 `cloneFiber`라는 함수를 통해 지연 생성(created lazily)된다. 항상 새로운 객체를 생성하는 대신에, `cloneFiber`는 존재할 경우 기존 fiber의 대체재를 사용하여 할당을 최소화시킨다.

`alternate` 필드를 구현 세부 사항으로 생각해야하지만, 여기에서 논의하는 것이 가치가 있을만큼 코드 상에서 자주 등장한다.

`output`

_host component_

- React 애플리케이션의 리프 노드. 렌더링 환경에 특정지어진다. (Browser 애플리케이션의 경우 `div`, `span`, etc와 같은 항목들이다. JSX에서 전체 소문자를 사용하여 표시된다.

개념상으로, fiber의 결과물은 함수의 반환 값이다.

모든 fiber는 결국 출력값을 갖지만 출력값은 host component를 통해 리프 노드에서만 생성된다. 출력값은 그런 뒤 트리의 위로 올라간다.

출력값은 rendering 환경에서 변경사항들을 flush 할 수 있게 렌더러에 전달되는 것이다. 출력물이 어떻게 생성되고 업데이트될지 결정하는 것은 렌더러의 역할이다.

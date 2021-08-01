---
layout: post
title: "Recoil 0.4 - 패치노트 읽기"
date: 2021-08-01 15:00:00 +09:00
categories: "react,recoil"
published: true
---

## 패치노트 소개

이번 업데이트는 Recoil이 가지고 있는 이슈 중 하나인 다량의 selector를 사용하거나, 하나의 selector가 지나치게 많은 의존성을 가지고 있을 때 발생하는 메모리 관련 이슈를 해결하고자 하는 대규모 업데이트의 시작점이라 생각됩니다.

기존 Recoil은 무엇보다 Snapshot을 통해 모든 Recoil 상태를 가지고 있었기 때문에 이런 메모리 누수가 발생하기 굉장히 쉬운 구조로 되어 있었는데요, 이번 업데이트를 통해 정말 필요한 항목만 유지시키고, 그렇지 않은 항목은 자연스럽게 드랍할 수 있게 될 수 있으리라 생각합니다.

다만, 여전히 UNSTABLE을 달고 나온 API들이라 바로 실사용하기는 어려워보이고, Recoil이 생각하는 방향에 대해 확인하는 업데이트 정도로 봐야할듯합니다. 구체적인 방향성 확립까지는 아직 시간이 조금 더 필요해보이네요.

## TL;DR

- selector 캐시 설정 옵션을 고를 수 있게 됩니다.
- 트랜잭션의 형태로 여러개의 atom을 동시에 업데이트할 수 있게 됩니다.
- HMR을 사용했을 때 발생하는 정말 이상한 오류 ‘this is fine, but odd’가 없어졌습니다.

## 설정 가능한 selector 캐시

selector와 selectorFamily 에서 사용할 수 있는 새로운 cachePolicy_UNSTABLE 속성은 selector 내부의 캐시 동작 방식을 직접 설정할 수 있게 해줍니다. 이 속성을 통해 많은 양의 selector를 가지고 있는 애플리케이션이나 다량의 의존성을 가지고 있는 selector 환경에서 메모리 사용을 줄일 수 있게 해줍니다.

아래는 새로운 속성을 사용하는 예제입니다.

```javascript
const clockState = selector({
  key: "clockState",
  get: ({ get }) => {
    const hour = get(hourState);
    const minute = get(minuteState);
    const second = get(secondState); // will re-run every second

    return `${hour}:${minute}:${second}`;
  },
  cachePolicy_UNSTABLE: {
    // Only store the most recent set of dependencies and their values
    eviction: "most-recent",
  },
});
```

위 예제에서 `clockState` 는 매 초마다 재계산되어 새로운 의존성 값을 내부 캐시에 저장하게 되는데, 이는 시간이 지남에 따라 내부 캐시가 무제한으로 증가할 수 있어 메모리 문제가 발생할 수 있습니다. `most-recent` (가장 최근) 캐쉬 제거 정책을 사용하면 내부 캐시는 가장 최근 의존성 값과 그 의존성 값을 토대로 만들어진 실제 selector 값만을 보관하여 메모리 이슈를 해결할 수 있습니다.

캐쉬 제거 옵션은 다음과 같습니다:

- `lru` - maxSize 로 정의된 사이즈를 초과하였을 때 가장 예전에 사용한 캐시를 제거합니다
- `most-recent` - 가장 최근 값만을 보관합니다
- `keep-all` - (기본값) 모든 엔트리를 보관하고 캐쉬를 제거하지 않습니다.

| 주의할 점은, 현재 제거 정책인 `keep-all`은 미래에 바뀔 수 있습니다.

### (역자 해석) 어떻게 작동하는 걸까?

selector나 atom에서 초기화 될 때 옵션에 따라 사용하는 Map이 조금씩 다릅니다.
`keep-all`과 같은 경우 단순히 Javascript Map을 wrapping한 `MapCache`의 구현체를 사용하는 반면 `lru와` `most-recent`는 Map에서 별도의 기능이 추가된 `LRUCache`를 사용합니다.
`LRUCache`는 size 항목을 가지고 있어, size가 초과된 경우 가장 예전에 사용된 항목을 지우도록 설계되어 있습니다.
`most-recent`는 `lru와` 같지만, size가 1로 고정됩니다.

## Atom Transaction

여러 개의 atom을 단일 트랜잭션을 통해 업데이트할 수 있는 새로운 API를 소개합니다. 새로운 `useRecoilTransaction_UNSTABLE()` hook은 이전 버전보다 더 간단하고, 실용적이며, 안전합니다. 이 hook은 향후 `useRecoilCallback()` hook이 사용되는 대부분의 케이스를 대체할 것이지만, 현재로서 이번 릴리즈는 최초 도입으로 향후 릴리즈에서 해결될 몇몇 한계점들을 포함하고 있습니다.

### 예제

`positionState`와 `headingState` 두 atom을 가지고 있고 이 두 값을 하나의 액션을 통해 `positonState`의 새로운 값을 현재 `positionState` 값과 `headingState` 값을 기반으로 만들고자 한다고 가정해보자. 이 액션은 side-effect가 없는 순수 함수를 기반의 트랜잭션으로 만들 수 있습니다.

```javascript
const goForward = useRecoilTransaction_UNSTABLE(
  ({ get, set }) =>
    (distance) => {
      const heading = get(headingState);
      const position = get(positionState);
      set(positionAtom, {
        x: position.x + cos(heading) * distance,
        y: position.y + sin(heading) * distance,
      });
    }
);
```

위와 같이 구성되면 `goForward(distance)`를 이벤트 핸들러 안에서 호출하는 것 만으로 트랜잭션을 수행할 수 있습니다. 이 트랜잭션은 상태값을 컴포넌트가 렌더되었을 때의 상태가 아닌 ‘현재 값'을 기준으로 업데이트해줍니다. 트랜잭션 중에는 이전 값도 읽을 수 있고, 트랜잭션 진행 중에는 다른 업데이트가 실행되지 않기 때문에 일정한 상태값을 기대할 수 있습니다.

만약 위 코드를 useRecoilCallback()을 이용하여 구현했다면 아래와 같이 나타날 것입니다:

```javascript
const goForward = useRecoilCallback(
  ({ snapshot, gotoSnapshot }) =>
    (distance) => {
      const mutatedSnapshot = snapshot.map(({ get, set }) => {
        const heading = get(headingState);
        const position = get(positionState);
        set(positionState, {
          x: position.x + cos(heading) * distance,
          y: position.y + sin(heading) * distance,
        });
      });
      gotoSnapshot(mutatedSnapshot);
    }
);
```

이 방식은 아래와 같은 문제점을 갖습니다:

- 전체 스냅샷을 확인해야하기 때문에 성능상 오버헤드가 발생합니다
- 버그가 발생할 지점이 많다: 스냅샷이 유지되고 미래에 사용될 수도 있습니다. 스냅샷은 Recoil 상태 변경점이 아닌, 전체를 보유하고 있기 때문에 스냅샷 생성과 트리 커밋 중간에 발생한 변경점을 의도치않게 되돌릴 수도 있습니다.

### Reducer 예제

이 hook을 이용하여 여러개의 atom을 변경하는 reducer 패턴을 구현할 수도 있습니다:

```javascript
const reducer = useRecoilTransaction_UNSTABLE(({get, set}) => action => {
  switch(action.type) {
    case 'goForward':
      const heading = get(headingState);
      set(positionState, position => {
        x: position.x + cos(heading) * action.distance,
        y: position.y + sin(heading) * action.distance,
      });
      break;

    case 'turn':
      set(headingState, action.heading);
      break;
  }
});
```

## 픽스와 최적화

- `selectorFamily`, `getCallback`, `useGetRecoilValueInfo`, `Sanpshot#getNodes` 가 정상적으로 typing되지 않았던 문제가 해결되었습니다.
- 이제 selector에서 mutable 값을 사용할 수 있게 해주는 `dangerouslyAllowMutability` 옵션이 `waitForAll()`과 같은 `waitFor*()` API를 사용할 수 있습니다.
- Atom Effects 수정사항
  - `onSet()` 핸들러가에서 atom이 리셋되거나 resolve되는 비동기 default Promise를 갖는 경우 올바른 새로운 값을 가져올 수 있도록 수정되었습니다.
  - 여러개의 Atom Effects 클린업 핸들러 문제가 수정되었습니다.
  - effect가 있는 atom이 Snapshot을 통해 초기화되었을 때 seletor 구독 문제가 수정되었습니다.
- 의존하는 항목이 cache된 값을 resolve하는 비동기 selector에 대한 최적화가 이뤄졌습니다.
- **불필요한 경고 메시지가 삭제되었습니다.**

---
layout: post
title: "React Redux의 동작 원리"
date: 2020-08-24 23:00:00 +09:00
categories: "react,redux"
published: true
---

## Redux의 동작 원리

Redux는 기본적으로 createStore로 이뤄진다. createStore는 dispatch, getState, 그리고 subscribe로 이뤄지며,
각각 다음과 같은 역할을 수행한다.

- dispatch: Action을 dispatch한다
- getState: 현재 state를 반환한다.
- subscribe: Action이 dispatch 되었을 때 실행할 액션을 정의한다.

```javascript
const counter = (state = 0, action) => {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    default:
      return state;
  }
};

const { createStore } = Redux;
const store = createStore(counter);

const render = () => {
  document.body.innerText = store.getState();
};

/**
 * register a callback that the redux will call
 * anytime an action has been dispatched
 */
store.subscribe(render);
render();

document.addEventListener("click", () => {
  store.dispatch({ type: "INCREMENT" });
});
```

## react-redux의 동작 원리

react-redux의 core는 Context API로 이루어진 Provider.js 파일로 이루어져있다.
얼핏 보기엔 결국 Context API로 구현한게 아닌가? 라고 생각할 수 있지만, 사실 Context API는 전역적으로 createStore의 함수들을 전달하기 위해서만 사용된다.

- 실질적으로 Provider에는 아래와 같은 코드가 있지만, store 그 자체가 변화하는 경우가 아니면 바뀌지 않는다.

```javascript
const contextValue = useMemo(() => {
  const subscription = new Subscription(store);
  subscription.onStateChange = subscription.notifyNestedSubs;
  return {
    store,
    subscription,
  };
}, [store]);
```

### Subscription

Subscription은 action이 dispatch되었을 때 변화를 기다리고 있을 listener들을 등록하는 역할을 한다.
listener들을 담고 있는 collection factory는 연결 리스트 형태로 listener들을 저장한다.

### useSelector는 어떻게 동작하는 것일까?

useSelector는 기본적으로 createSelectorHook 함수 결과를 반환하는데,
createSelectorHook은 함수 팩토리 역할을 수행한다

```javascript
export function createSelectorHook(context = ReactReduxContext) {
  const useReduxContext = useDefaultReduxContext;

  return function useSelector(selector, equalityFn = refEquality) {
    /**
     * 현재 Redux Context 안에는 store와 subscription이 전달되어 있다.
     * Provider에서 value로 그 둘을 넣은 contextValue를 전달하고 있기 때문이다.
     */
    const { store, subscription: contextSub } = useReduxContext();

    const selectedState = useSelectorWithStoreAndSubscription(
      selector,
      equalityFn,
      store,
      contextSub
    );

    return selectedState;
  };
}
```

여기서 `useSelectorWithStoreAndSubscription`이 어떤 역할을 수행하는지 확인해야한다.
해당 함수는 현재 store에서 특정 항목만 가져오는 `selector` 함수와 이전 ref값과 비교하는 `equalityFn`, 현재 `store`, 그리고 Root `subscription`이 전달된다.

```javascript
function useSelectorWithStoreAndSubscription(
  selector,
  equalityFn,
  store,
  contextSub
) {
  const [, forceRender] = useReducer((s) => s + 1, 0);

  const subscription = useMemo(() => new Subscription(store, contextSub), [
    store,
    contextSub,
  ]);

  const latestSelector = useRef();
  const latestStoreState = useRef();
  const latestSelectedState = useRef();

  const storeState = store.getState();
  let selectedState;

  if (
    selector !== latestSelector.current ||
    storeState !== latestStoreState.current
  ) {
    selectedState = selector(storeState);
  } else {
    selectedState = latestSelectedState.current;
  }

  useEffect(() => {
    latestSelector.current = selector;
    latestStoreState.current = storeState;
    latestSelectedState.current = selectedState;
  });

  useEffect(() => {
    function checkForUpdates() {
      const newSelectedState = latestSelector.current(store.getState());
      if (equalityFn(newSelectedState, latestSelectedState.current)) {
        return;
      }
      latestSelectedState.current = newSelectedState;

      forceRender();
    }

    subscription.onStateChange = checkForUpdates;
    subscription.trySubscribe();

    checkForUpdates();

    return () => subscription.tryUnsubscribe();
  }, [store, subscription]);

  return selectedState;
}
```

여기가 바로 가장 흥미로운 부분이다.
기본적으로 react-redux는 useRef를 이용하여 가장 최근의 `latestSelectedState`값을 보관한다.
우선 `selectedState`를 반환한 뒤, 내부적으로 정의한 `checkforUpdate` callback을 `subscription.onStateChange`에 등록시켜 놓는다.

이제 값이 바뀔 때마다 `checkForUpdates`가 실행이 되고, 만약에 바뀐 값이 있을 경우 `forceRender`를 실행하여 새로운 `selectedState`를 반환한다.

이 방법을 통해 Redux와 react-redux는 store의 변화가 있을 때 `useSelector`를 통해, 부분적으로 업데이트를 실행할 수 있게된다.

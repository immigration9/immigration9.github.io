## Redux Saga 도입

### 개요
Redux Saga는 Redux를 사용 할 때 애플리케이션에서 발생하는 Data Fetching과 같은 비동기 동작들을 쉽게 처리하기 위한 라이브러리다.

Redux Saga는 기존의 Redux Thunk가 잘 처리하지 못하는 비동기 상황에서의 state 처리를 간단하게 해결해준다는 점에서 의의가 있다.

기존에 계속해서 도입하지 못했던 가장 큰 이유는 multireducer의 존재가 가장 크기도 했고, 또 나아가서, Redux Saga 자체의 러닝커브가 굉장히 높다.

일단 닭잡는데 칼 만드는 방법을 배울 필요는 없지만, https://davidwalsh.name/es6-generators 최소한 ES6 Generator가 무엇인지 공부해보면 좋을것 같다.

아래 내용은 Redux Saga에 대한 설명이라기보단, 어떻게 implement되었고, 어떻게 사용하면 되는지에 대한 설명에 가깝다. 자세한 내용은 https://mskims.github.io/redux-saga-in-korean/introduction/BeginnerTutorial.html 여기를 참고하는 것이 좋을듯하다.


### 기본 용어 설명

* Effect: Saga Middleware이 처리하도록 하는 instruction을 가지고 있는 JavaScript 객체. redux-saga 라이브러리가 제공하는 factory를 사용하여 Effect를 생성한다. `call(myfunc, 'arg1', 'arg2')`을 사용하면 middleware는 `myfunc('arg1', 'arg2')`를 호출하고, 해당 Effect를 yield한 Generator로 결과를 반환한다.

Root Saga에는 모든 페이지들로부터 나오는 Saga들을 한 곳으로 모아서 병렬적으로 처리해준다.
* all: 병렬적으로 여러개의 Effect를 실행시키고, 전부 완료될 때까지 대기하도록 하는 effect를 생성한다. `Promise.all`과 유사하게 작동한다고 보면 된다.

### 구현부


```js
import { all } from 'redux-saga/effects'
import cubeSaga from 'containers/CubeV2/CubeSagas'

function* rootSaga() {
  yield all([
    cubeSaga()
  ])
}

export default rootSaga;
```

기존의 액션은 동일하게 구성해도 된다. 다만, Action으로 전달되는 모든 항목들은 `payload` 안에 넣어줘야 한다.
```ts
/** 
 * CubeActions 
 */

/** Request Actions */
export function fetchRequest(): SagaRequestAction {
  return { type: REQUESTS.USER_FETCH.REQUEST, payload: { user: "test" } }
}

export function asyncRequest(): SagaRequestAction {
  return { type: REQUESTS.ASYNC_FETCH.REQUEST }
}

/** Reducer */
const initialState: CubeState = {
  user: undefined
}

const REDUCER: ReducerObject = {
  [REQUESTS.USER_FETCH.SUCCESS]: (state, { payload }) => {
    return produce<CubeState>(state, draft => { draft.user = payload.user; })
  },
  [REQUESTS.ASYNC_FETCH.SUCCESS]: (state, { payload }) => {
    return produce<CubeState>(state, draft => { draft.user = payload.data; })
  }
}
```

여기서 Saga는 별도의 Middleware인데, 어떻게 별도의 명시 없이 어떻게 multireducer의 key를 가질 수 있을까?
이를 해결하기 위해 별도의 호환 유틸리티를 추가하였다. 

아래를 보면 알 수 있지만, multireducer는 기존의 `dispatch`를 한 번 더 감싸준 다음, `reducerKey`를 추가하는 방법을 취하고 있었다.
이 방법을 통해, reducers에 접근 할 때, key값을 가지고 접근할 수 있던 것이었다.
```json
{
  type: "NORMAL_REQUEST",
  meta: {
    __multireducerKey: "CubeV2"
  }
}
```

이와 같은 방법으로 중간에 하이재킹을 통해, 다시 한 번 reducerKey를 객체에 추가하는 방식으로 구성하였다.
이제, Saga와 multireducer를 동시에 사용하는 사용자는, 기존의 코드를 사용하면서, Saga가 reducerKey를 따라가도록 할 수 있다.
물론, 꼭 Saga를 사용하지 않는 경우에는 그냥 thunk를 사용해도 된다.
```ts
/** CubeV2 */
const mapDispatchToProps = (dispatch: any, { }: OwnProps): DispatchProps => bindSagaActionCreators(CubeActions, dispatch, REDUCER_KEY);

/** sagaUtils */
import { wrapAction } from "multireducer";
import { bindActionCreators as originalBindActionCreator } from 'redux';
/**
 * @description Action Function에 reducerKey를 넣어서 실질적인 작업을 하는 Saga Action쪽에서 활용할 수 있도록 한다.
 */
export function bindSagaActionCreators(actionCreators: any, dispatch: Function, reducerKey: string) {
  const wrappedDispatch = (action: any) => {
    let wrappedAction;
    if (typeof action === 'function') {
      wrappedAction = (globalDispatch: any, getState: any, extraArgument: any) =>
        action(wrappedDispatch, getState, globalDispatch, reducerKey, extraArgument);
    } else if (typeof action === 'object') {
      wrappedAction = wrapAction(action, reducerKey);
    }
    /**
     * 여기서 Wrap된 Action 내부에 reducerKey를 추가로 넣어준다.
     */
    wrappedAction.reducerKey = reducerKey;
    return dispatch(wrappedAction);
  };
  return originalBindActionCreator(actionCreators, wrappedDispatch);
}
```

이제, 마지막으로 Saga를 구현하는 단계가 남았다.
여기서 메인 포인트는 아래와 같았다.
1. Saga를 제작하는데 지나치게 많은 코드가 들어가선 안된다.
2. 매번 똑같은 코드를 복붙하는것은 하고싶지 않다.
```ts
/**
 * CubeSagas
 */
import { delay, call } from "redux-saga/effects";
import { sagaAction, sagaHandlerFactory, sagaRequestCreator } from "utils";
import { getAllApplication } from "whatap-rest";
import { SagaStepSet } from "common-interfaces/SagaInterface";

const USER_FETCH_REQUESTED = 'USER_FETCH_REQUESTED'
const USER_FETCH_SUCCEEDED = 'USER_FETCH_SUCCEEDED'

const ASYNC_FETCH_REQUESTED = 'ASYNC_FETCH_REQUESTED';
const ASYNC_FETCH_SUCCEEDED = 'ASYNC_FETCH_SUCCEEDED';
const ASYNC_FETCH_FAILED = 'ASYNC_FETCH_FAILED';

export const REQUESTS: SagaStepSet = {
  user_fetch: sagaRequestCreator(USER_FETCH_REQUESTED, USER_FETCH_SUCCEEDED, handleFetchRequest),
  async_fetch: sagaRequestCreator(ASYNC_FETCH_REQUESTED, ASYNC_FETCH_SUCCEEDED, handleAsyncRequest, undefined, ASYNC_FETCH_FAILED)
}

export function* handleFetchRequest(action: any) {
  yield delay(3000);
  yield sagaAction(REQUESTS.user_fetch.success, action);
}

export function* handleAsyncRequest(action: any) {
  const { async_fetch } = REQUESTS;
  const res = yield call(getAllApplication);
  try {
    const data = yield res.json();
    yield sagaAction(async_fetch.success, action, { data });
  } catch (error) {
    if (async_fetch.failure) yield sagaAction(async_fetch.failure, action, { error });
  }
}

export default sagaHandlerFactory(REQUESTS);
```

`sagaRequestCreator`는 말 그대로, Saga Request가 갈 수 있는 항목들을 객체로 만들어주는 것이다.
`sagaHandlerFactory`는 해당 파일 내에 있는 Saga Request들에 모아서 하나로 만드는 작업을 해주는 Function을 만드는 Factory역할을 한다.

```js
export function sagaRequestCreator(request: string, success: string, handler: (action: any) => void, pending?: string, failure?: string): SagaStep {
  return { request, success, handler, pending, failure }
}

export function sagaHandlerFactory(sagaRequests: SagaStepSet) {
  /**
   * Generator function을 만들어 리턴한다.
   */
  let sagaHandler = function* () {
    for (let key in sagaRequests) {
      if (sagaRequests.hasOwnProperty(key)) {
        const requestObj = sagaRequests[key];
        yield takeLatest(requestObj.request, requestObj.handler);
      }
    }
  }
  return sagaHandler;
}
```

Async API call은 `async await`와 유사하게 작동한다. `call`은 Blocking Call로 해당 Function의 종료를 대기한다.
```js
export function* handleAsyncRequest(action: any) {
  const { async_fetch } = REQUESTS;
  const res = yield call(getAllApplication);
  try {
    const data = yield res.json(); /** 여기서는 data를 추출하는 작업을 한 번 해준다. */
    yield sagaAction(async_fetch.success, action, { data });
  } catch (error) {
    if (async_fetch.failure) yield sagaAction(async_fetch.failure, action, { error });
  }
}
```

* Blocking / Non-blocking Call이란?
Blocking call은 Effect를 yield한 후, 실행 결과가 종료될 때까지 기다린다.
반대로, Non-blocking call은 Saga가 yield 후 곧바로 다음 작업에 돌입한다.

```js
/** Redux Saga API: https://redux-saga.js.org/docs/Glossary.html */ 
import {call, cancel, join, take, put} from "redux-saga/effects"

function* saga() {
  yield take(ACTION)              // Blocking: will wait for the action
  yield call(ApiFn, ...args)      // Blocking: will wait for ApiFn (If ApiFn returns a Promise)
  yield call(otherSaga, ...args)  // Blocking: will wait for otherSaga to terminate

  yield put(...)                   // Non-Blocking: will dispatch within internal scheduler

  const task = yield fork(otherSaga, ...args)  // Non-blocking: will not wait for otherSaga
  yield cancel(task)                           // Non-blocking: will resume immediately
  // or
  yield join(task)                              // Blocking: will wait for the task to terminate
}
```
ex) 다양한 종류의 Saga를 한 흐름에 콜 해야할 경우,
서로 직렬적으로 작업이 필요할 경우 => `call`
서로 병렬적으로 작업이 필요한 경우 => `fork & cancel`
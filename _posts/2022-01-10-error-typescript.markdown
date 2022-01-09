---
layout: post
title: "TypeScript에서 catch block error message 사용하기"
date: 2022-01-10 01:00:00 +09:00
categories: "typescript"
published: true
---

본 글은 Kent C. Dodds의 Get a catch block error message with TypeScript 글의 한글 번역본입니다.
[Get a catch block error message with TypeScript](https://kentcdodds.com/blog/get-a-catch-block-error-message-with-typescript)

---

아래 코드를 한 번 보자.

```javascript
const reportError = ({ message }) => {
  // 로깅서비스로 에러 전송
};

try {
  throw new Error("Oh no!");
} catch (error) {
  // 진행은 하겠지만, 리포트는 전송하자.
  reportError({ message: error.message });
}
```

여태까지는 좋아보인다. 하지만 그건 어디까지나 JavaScript를 이용하고 있기 때문이다. 여기에 TypeScript를 더해보자:

```typescript
const reportError = ({ message }: { message: string }) => {
  // 로깅서비스로 에러 전송
};

try {
  throw new Error("Oh no!");
} catch (error) {
  // 진행은 하겠지만, 리포트는 전송하자.
  reportError({ message: error.message });
}
```

`reportError` 호출에서 에러가 나는 것을 발견할 수 있다. `error.message` 부분에서 에러가 난다. TypeScript는 `error`의 타입을 `unknown`을 기본값으로 갖기 때문이다. 그리고 실제로 그게 맞기도 하다! 에러의 세계에서 발생한 에러의 타입에 대해서는 전할 수 있는 확신이 부족하다. 실제로 이는 Promise의 rejection항목 `.catch(error => {})`에 promise generic (`Promise<ResolvedValue, NopeYouCantProvideARejectedValueType>`)을 이용하여 타입을 제공하지 못하는 이유와 같다고 할 수 있다. 실제로 던져진 값은 에러가 아닐 수도 있다. 아래와 같은 어떤 값도 가능하다.

```typescript
throw "What the!?";
throw 7;
throw { wut: "is this" };
throw null;
throw new Promise(() => {});
throw undefined;
```

진짜 아무 타입이나 던질 수 있다. 간단해 보이지 않는가? 그러면 에러에 타입 어노테이션을 추가해서 에러만 던져질 것이라 명시해주자.

```typescript
try {
  throw new Error("Oh no!");
} catch (error: Error) {
  // 진행은 하겠지만, 리포트는 전송하자.
  reportError({ message: error.message });
}
```

잠깐! 이렇게 작성하면 아래와 같은 TypeScript 컴파일 에러가 발생한다:

```
Catch clause variable type annotation must be 'any' or 'unknown' if specified. ts(1196)
```

이런 에러가 발생하는 이유는 우리가 작성한 코드가 분명 에러 이외의 다른 것을 던져줄 수 없도록 보여지지만, JavaScript의 희안한 동작 방식으로 인하여 이상한 방식으로 에러 객체를 몽키패칭한 써드 파티 라이브러리가 완전히 다른 무언가를 던지도록 할 수도 있다:

```typescript
Error = function () {
  throw "Flowers";
} as any;
```

그럼 이 문제를 어떻게 해결할 수 있을까?

```typescript
try {
  throw new Error("Oh no!");
} catch (error) {
  let message = "Unknown Error";
  if (error instanceof Error) message = error.message;
  // 진행은 하겠지만, 리포트는 전송하자.
  reportError({ message });
}
```

이렇게 하니까 이제 더 이상 TypeScript가 에러를 발생시키지도 않고, 우리는 이제 완벽히 기대하지 못했던 일이 발생해도 핸들링 할 수 있게 되었다. 여기서 조금 더 잘 해보자:

```typescript
try {
  throw new Error("Oh no!");
} catch (error) {
  let message;
  if (error instanceof Error) message = error.message;
  else message = String(error);
  // 진행은 하겠지만, 리포트는 전송하자.
  reportError({ message });
}
```

여기서 만약 error가 실제로 `Error` 객체가 아니라면 에러를 stringify 하여 뭔가 유용한게 나오길 기대해보면 된다.

이렇게 만든 코드를 모든 catch 블록에서 사용할 수 있는 유틸 함수로 만들어줄 수도 있다.

```typescript
function getErrorMessage(error: unknown) {
  if (error instanceof Error) return error.message;
  return String(error);
}

const reportError = ({ message }: { message: string }) => {
  // 로깅서비스로 에러 전송
};

try {
  throw new Error("Oh no!");
} catch (error) {
  // 진행은 하겠지만, 리포트는 전송하자.
  reportError({ message: getErrorMessage(error) });
}
```

유용하게 사용해주길 바란다.

Update: Nicolas와 Jesse가 추천해준 방법을 조합하여 아래와 같은 결과를 낼 수도 있다.

```typescript
type ErrorWithMessage = {
  message: string;
};

function isErrorWithMessage(error: unknown): error is ErrorWithMessage {
  return (
    typeof error === "object" &&
    error !== null &&
    "message" in error &&
    typeof (error as Record<string, unknown>).message === "string"
  );
}

function toErrorWithMessage(maybeError: unknown): ErrorWithMessage {
  if (isErrorWithMessage(maybeError)) return maybeError;

  try {
    return new Error(JSON.stringify(maybeError));
  } catch {
    // 순환 참조와 같이 maybeError를 stringify하는 과정에서 발생하는
    // 에러에 대해 fallback을 제공한다
    return new Error(String(maybeError));
  }
}

function getErrorMessage(error: unknown) {
  return toErrorWithMessage(error).message;
}
```

---

### 역자 해설

```typescript
function isErrorWithMessage(error: unknown): error is ErrorWithMessage {
  return (
    typeof error === "object" &&
    error !== null &&
    "message" in error &&
    typeof (error as Record<string, unknown>).message === "string"
  );
}
```

위에 있는 `error is ErrorWithMessage`는 무얼 뜻하는걸까?
위 예제에서 사용된 `isErrorWithMessage` 자체의 기본적인 타입은 `boolean`이라 할 수 있다.
그렇다면 여기서 사용된 `is` 키워드는 Type Predicate, 직역하자면 '타입 단정'이라 할 수 있다.

TypeScript에서 다양한 타입에 대해 공부를 하다 보면 타입을 더 정교하게 접근하는 것에 학습을 하는데,
타입 단정은 여기서 한 발 더 나아가 코드 내에서 특정 타입이 어떻게 바뀌는지를 좀 더 컨트롤 할 수 있게 해준다.

예를들어 위 예제에서 `maybeError`는 `unknown` 타입을 갖는다. 하지만 `isErrorMessage`를 통과하고 나면, type은 `message` 프로퍼티를 갖고 있는 객체 타입이란 것이 확인이 되기 때문에 더 이상 `unknown` 타입이 아니게 된다.

여기서 `error is ErrorWithMessage`를 전해줌으로써, 앞으로 해당 함수가 `true`가 되면, 이후에 사용될 `error` 객체는 `ErrorWithMessage` 타입을 갖게 된다는 점을 의미한다.

여기서 주의해야할 점은, `error is ErrorWithMessage`가 적용되는 지점은, `error` 자체가 해당 타입으로 확인 될 때만 사용할 수 있다는 점이다.

```typescript
function toErrorWithMessage(maybeError: unknown): ErrorWithMessage {
  if (isErrorWithMessage(maybeError)) return maybeError;

  // ...
}
```

예를들어, 위 코드에서 `isErrorWithMessage(maybeError)`가 `true`일 경우, `maybeError`의 타입은 `ErrorWithMessage`가 되지만,
그 이외의 경우에는 모두 `unknown`으로 남게 된다.
